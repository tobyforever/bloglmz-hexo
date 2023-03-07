---
title: 大数据第1课 hadoop
date: 2023-02-20 20:43:51
tags:
---

#What?
hadoop是分布式文件系统，文件uri格式为“hdfs://path-to-file”

#Why?
使用廉价的x86机器搭建的集群代替高端存储，hdfs分布式文件系统具有可扩展、多副本高可用的特点。

#How?

#### 实验环境
vmware里启动bigdata01、bigdata02、bigdata03虚拟机，对应的IP分别为192.168.52.100  192.168.52.110  192.168.52.120。在本地宿主机上配置好hosts文件里域名指向（bigdata01 bigdata02 bigdata03）

#### 启动hadoop

```bash
# 密码123456
ssh hadoop@bigdata01  
cd ~/bin
ls
```
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230307104610.png)

```bash
# 启动zookeeper和hadoop集群
./zkhadoop.sh start 
# 查看启动结果  
jps 

```
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230307105523.png)

#### 基本操作命令
```bash
# 显示hadoop版本号
hdfs version
# 显示根目录下的文件
hdfs dfs -ls /
# 下载hsfs上的文件
hdfs dfs -get /word.txt
# 删除文件
hdfs dfs -rm /word.txt
# 上传
hdfs dfs -put word.txt /
# 创建文件夹
hdfs dfs -mkdir -p /geektime
```

#### idea的bigdata-tool大数据窗口连接hadoop
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230307110700.png)
这里8020端口是在HADOOP_HOME/etc(路径/opt/install/hadoop-3.3.4/etc/hadoop
)下的core-sites.xml里定义的
```xml
<property>
        <name>fs.defaultFS</name>
        <value>hdfs://bigdata01:8020</value>
</property>
```