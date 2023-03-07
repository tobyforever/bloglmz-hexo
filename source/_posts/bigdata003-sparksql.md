---
title: 大数据第3课 SparkSQL
date: 2023-02-22 20:43:51
tags: 大数据
toc: true
---

# What?
spark是统一的内存计算引擎。SparkSQL是通过SQL界面写spark任务程序。SparkStreaming是通过微批次的方式，实现实时流数据处理的效果。

>流：在流的观点下，数据被看成是无界的，无限延伸，无穷无尽；流处理是搭建一个“管道”，数据随时进来随时处理，而这个“管道”程序是一直保持运行状态的。
>而传统观点下数据是有界的，类似一个数组，可以遍历。批处理是把数据一批一批地处理，数据处理完一批程序就结束。


# Why?
(1)Spark把中间数据放在内存中，迭代运算效率高。mapreduce中的计算结果保存在磁盘上，而spark支持DAG图的分布式并行计算的编程框架，减少了迭代过程中数据的落地，提高了处理效率。
spark可以支持读写hive，从而避免hive执行mapreduce慢的问题。

(2)spark容错性高。引进了RDD,如果数据集一部分丢失，则可以重建。另外，在RDD计算时可以通过checkpoint来实现容错。

(3)spark更加通用。不像hadoop只提供map和reduce两种操作。spark提供的数据集操作类型有很多种，大致分为转换操作和行动操作。转换操作包括map,filter,flatmap,sample,groupbykey,reducebykey,union,join,cogroup,mapvalues,sort和partionby等多种操作类型，行动操作包括collect,reduce,lookup和save等操作类型。另外，各个处理节点之间的通信模型不再像Hadoop只有shuffle一种模式，用户可以命名，物化，控制中间结果的存储，分区等。

# How?
## spark的数据模型
截止到目前（202303）， spark最新的版本号是3.3.2
spark的数据模型从旧到新（或者从低级到高级）分别为RDD、DataSet(spark1.6开始)、[DataFrame](https://spark.apache.org/docs/latest/sql-programming-guide.html#dataframes)。


spark的dataset/dataframe是有列名的列，类似于关系数据库的table，或者大数据分布式版本的python pandas的dataframe。ds/df在java/scala语言中支持强类型，并且提供了常见的函数式算子如flatMap等用于对数据进行函数式变换(transform)操作。通过使用dataset/dataframe，可以强类型地访问数据表的单元，以及进行transform。在scala、java中，dataframe实际上是Dataset[Row]。


为什么要通过RDD和函数式进行数据操作？因为大数据场景下移动数据的成本很高，需要移动计算程序并且把计算程序并行化，而函数式的immutable性质正好满足了这类场景。

[RDD的操作](https://spark.apache.org/docs/latest/rdd-programming-guide.html#rdd-operations)分为两类，transform和action。像flatMap、filter这种算子就属于transform，也就是从一个RDD转换为另一个RDD，程序调用的时候并不会立即执行（tranforms are lazy）；像first、take、count、collect这种就属于action，会结束数据处理程序并返回计算结果。

## SparkSQL的处理流程
### 获取sparkcontext
由于程序并非单机运行，实际执行环境可能是在多台机器上，因此spark把执行环境抽象为了sparkcontext，需要先获取sparkcontext或者sparksession对象sc，然后再调用sc.sql()方法进行sql查询。

>在zeppelin notebook环境中编写脚本时，sparkcontext已经被注入为sc对象，可以直接引用。

### 从数据源读取数据
常见的数据源包括：

#### hive表
spark可以通过sql直接对hive进行操作 [官网文档](https://spark.apache.org/docs/latest/sql-data-sources-hive-tables.html)
```scala
import java.io.File

import org.apache.spark.sql.{Row, SaveMode, SparkSession}

case class Record(key: Int, value: String)

// warehouseLocation points to the default location for managed databases and tables
val warehouseLocation = new File("spark-warehouse").getAbsolutePath

val spark = SparkSession
  .builder()
  .appName("Spark Hive Example")
  .config("spark.sql.warehouse.dir", warehouseLocation)
  .enableHiveSupport()
  .getOrCreate()

import spark.implicits._
import spark.sql

sql("CREATE TABLE IF NOT EXISTS src (key INT, value STRING) USING hive")
sql("LOAD DATA LOCAL INPATH 'examples/src/main/resources/kv1.txt' INTO TABLE src")

// Queries are expressed in HiveQL
sql("SELECT * FROM src").show()
// +---+-------+
// |key|  value|
// +---+-------+
// |238|val_238|
// | 86| val_86|
// |311|val_311|
// ...

// Aggregation queries are also supported.
sql("SELECT COUNT(*) FROM src").show()
// +--------+
// |count(1)|
// +--------+
// |    500 |
// +--------+

// The results of SQL queries are themselves DataFrames and support all normal functions.
val sqlDF = sql("SELECT key, value FROM src WHERE key < 10 ORDER BY key")

// The items in DataFrames are of type Row, which allows you to access each column by ordinal.
val stringsDS = sqlDF.map {
  case Row(key: Int, value: String) => s"Key: $key, Value: $value"
}
stringsDS.show()
// +--------------------+
// |               value|
// +--------------------+
// |Key: 0, Value: val_0|
// |Key: 0, Value: val_0|
// |Key: 0, Value: val_0|
// ...

// You can also use DataFrames to create temporary views within a SparkSession.
val recordsDF = spark.createDataFrame((1 to 100).map(i => Record(i, s"val_$i")))
recordsDF.createOrReplaceTempView("records")

// Queries can then join DataFrame data with data stored in Hive.
sql("SELECT * FROM records r JOIN src s ON r.key = s.key").show()
// +---+------+---+------+
// |key| value|key| value|
// +---+------+---+------+
// |  2| val_2|  2| val_2|
// |  4| val_4|  4| val_4|
// |  5| val_5|  5| val_5|
// ...

// Create a Hive managed Parquet table, with HQL syntax instead of the Spark SQL native syntax
// `USING hive`
sql("CREATE TABLE hive_records(key int, value string) STORED AS PARQUET")
// Save DataFrame to the Hive managed table
val df = spark.table("src")
df.write.mode(SaveMode.Overwrite).saveAsTable("hive_records")
// After insertion, the Hive managed table has data now
sql("SELECT * FROM hive_records").show()
// +---+-------+
// |key|  value|
// +---+-------+
// |238|val_238|
// | 86| val_86|
// |311|val_311|
// ...

// Prepare a Parquet data directory
val dataDir = "/tmp/parquet_data"
spark.range(10).write.parquet(dataDir)
// Create a Hive external Parquet table
sql(s"CREATE EXTERNAL TABLE hive_bigints(id bigint) STORED AS PARQUET LOCATION '$dataDir'")
// The Hive external table should already have data
sql("SELECT * FROM hive_bigints").show()
// +---+
// | id|
// +---+
// |  0|
// |  1|
// |  2|
// ... Order may vary, as spark processes the partitions in parallel.

// Turn on flag for Hive Dynamic Partitioning
spark.sqlContext.setConf("hive.exec.dynamic.partition", "true")
spark.sqlContext.setConf("hive.exec.dynamic.partition.mode", "nonstrict")
// Create a Hive partitioned table using DataFrame API
df.write.partitionBy("key").format("hive").saveAsTable("hive_part_tbl")
// Partitioned column `key` will be moved to the end of the schema.
sql("SELECT * FROM hive_part_tbl").show()
// +-------+---+
// |  value|key|
// +-------+---+
// |val_238|238|
// | val_86| 86|
// |val_311|311|
// ...

spark.stop()
```

#### txt
参见[作业](https://github.com/tobyforever/geektime-bigdata-homework/blob/main/sparkwithscala/src/main/scala/Main.scala),
注意txt不如csv方便，csv文件里可以设置表头行直接解析成dataframe，而txt因为没有列名信息需要手动做schema mapping。
#### csv
```scala
val peopleDFCsv = spark.read.format("csv")
  .option("sep", ";")
  .option("inferSchema", "true")
  .option("header", "true")
  .load("examples/src/main/resources/people.csv")
```

#### json
和csv类似，属于带自带schema的数据。

#### parquet
Apache Parquet是Cloudera和twitter 2013年推出的Hadoop生态圈的一种新型列式存储二进制格式，针对大数据、写一次读多次（WORM）场景做了优化，在数据压缩性和查询性能（谓词下推、投影下推）方面有优势。自带元数据 [参考](https://www.cnblogs.com/helongBlog/p/13750315.html)
```scala
usersDF.write.format("parquet")
  .option("parquet.bloom.filter.enabled#favorite_color", "true")
  .option("parquet.bloom.filter.expected.ndv#favorite_color", "1000000")
  .option("parquet.enable.dictionary", "true")
  .option("parquet.page.write-checksum.enabled", "false")
  .save("users_with_options.parquet")
```
直接在sql语句中连接数据文件
```sql
val sqlDF = spark.sql("SELECT * FROM parquet.`examples/src/main/resources/users.parquet`")
```
#### avro
avro是hadoop在2009年推出的行式存储格式，文件里schema用json而数据用二进制。

#### 各个数据文件格式的比较：
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230307173046.png)

### 对数据进行查询
查询可以通过2种方式：
1、dataframe api
```scala
  ans = peopleDF.filter("sex=\"男\"").count()
    System.out.println("一共有多个男生参加考试？" + ans)
```
2、sql查询
```scala
var ans3 = peopleDF.sqlContext.sql("select avg(score) from people where name in ( select name from  people group by name having sum(score)>150 and age >=19) and name not in (select distinct name from people where subject=\"math\" and score < 70)");
    System.out.println("总成绩大于150分，且数学大于等于70，且年龄大于等于19岁的学生的平均成绩是多少？" + ans3.first().get(0));
```

