---
---
title: 大数据第7课 hbase的使用
date: 2023-02-25 20:43:51
tags: 大数据
toc: false
published: true
---

# What?
掌握的主要hbase概念和使用要点

# Why?
hbase是列式KV数据库，可用于大数据的KV、时序数据、空间数据。
hbase可以做到百亿级别数据量rowkey查询秒级返回

hbase适合少量数据的实时CRUD（OLTP）， hive适合海量数据的统计分析离线计算OLAP

# How?
## hbase

## 基本概念
hbase的数据模型是tablename表名、列族columnfamily...（不多于4个）， 列column 

hbase底层存储是hadoop上，hadoop一个block=128M

hbase将 元数据保存在hbase自身的一张表meta表里

## 启动hbase

```bash
// 先启动zk-hadoop：
cd ~/bin
./zkhadoop.sh start
//启动hbase
cd /opt/install/hbase-2.2.6/bin
./start-hbase.sh
//进入hbase client命令行
cd /opt/install/hbase-2.2.6/bin
hbase shell

//浏览器访问bigdata01:60010来查看hbase服务状态页面
```
在hbase shell中的常见命令：

(注意命令末尾不要加分号！！直接回车就可以了)

查看表
```
list
```
创建表
```
create 'tablename', 'colfamily1', 'colfamily2'... 
```
put插入数据
(在user表里插入主键rowkey为rowkey001的行，列族cf1里插入列col1，值为value1)
```
put 'user', 'rowkey001', 'cf1:col1', 'value1'
```

查询数据
根据rowkey查指定行的所有cell
```
get 'tablename', 'rowkey001'
```

范围查询指定表
```
scan 'tablename' 千万不要用，数据量很多会爆内存

//指定范围查询你，按rowkey的字典序排序，区间是前闭后开，包含start不包含end
scan 'user', {COLUMNS => 'info', STARTROW=> 'rowkey0001', ENDROW='rowkey0003'}

//按rowkey的前缀进行过滤   （速度怎么样？）
scan 'user', {FILTER=>"PrefixFilter('rk')"}
```

删除表
disable 'user'
drop 'user'

统计行数
count 'user'

查看表结构
describe 'user'

truncate 

## 海量数据装载
bulkload方式，将数据直接转换为HFile格式。

hbase的数据写入流程：
1、客户端写入数据，把数据保存在memoryStore里，128M内存空间。
2、memoryStore数据写满了，进行溢写，将memoryStore的数据写入到StoreFile里
3、多个storeFile会进行合并为一个HFile
最终hbase的数据存储到hdfs上是HFile格式。

# hbase和hive的整合使用：
hbase的数据太多了，查询过滤api调用比较繁琐，在OLAP场景下使用不方便。
使用hive把hbase映射成为一张表，作用：
1、使用hive SQL语句来进行查询分析hbase中的数据。
2、将hive统计分析的结果保存到hbase里面去供给外部使用。 
在hive中创建表：
create table course.hbase_score(id int,cname string, score int)
stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
with serdeproperties("hbase.columns.mapping"="cf:name,cf:score") tblpropertise("hbase.table.name"="hbase_score")
用hive插入数据到hbase：
insert overwrite table course.hbase_score select id, cname, score from course.score;

使用hive来映射hbase当中的表数据，数据是存储在hbase里了

# hbase和ES结合加速查找
hbase的scan要按范围过滤（age>18）会比较慢，所以用es把age=18的rowkey做二级索引）

# 图形化界面工具HbaseGUI
zk.port   2181
zk.quorum    bigdata01
hbase.master    bigdata01:16000
znode.parent    /hbase
mavenhome         D:\lib\apache-maven-3.6.3 （根据实际）
Hbase version  2.2.4

开windows命令行窗口，运行bin下的start.cmd, 在连接配置界面里选mavenhome好后选hbase version，会开始下载驱动jar，这时候看控制台输出找到下载的ouput目录，例如C:\Users\tobyf\.hbase-gui-conf\driver， 然后中断退gui出界面，把driver压缩包里2.2.4版本解压到对应的驱动目录里。然后重新start打开配置界面，再填好maven和hbase version点刷新圆圈按钮，会提示“已加载2.2.4所需的所有jar包”，填写上面其他的配置参数，点connect，就能成功进入hbase管理界面。进入一次后连接配置会自动保存。

# java使用jdbc驱动phoenix查询hbase

java项目的resources下面放配置文件：
core-site.xml
fairscheduler.xml
hbase-site.xml
hdfs-site.xml
hive-site.xml
