---
title: 大数据第2课 hive
date: 2023-02-21 20:43:51
tags: 大数据
toc: true
---

# What?
hive是在hadoop上通过sql执行mapreduce。

# Why?
mapreduce程序编写繁琐，目前基本不会直接使用。hive提供了sql界面，将sql语句转化为mapreduce执行。由于MR的执行速度较慢，hive只适合用来做离线数仓。

# How?
参考[hive官方手册GettingStarted](https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/GettingStarted.html)

## 启动hive服务
启动hadoop集群，进入bigdata03虚拟机，启动hiveserver2服务
```bash
cd /opt/install/apache-hive-3.1.3/bin
hive --service hiveserver2
```
另外开启一个窗口，启动beeline：
```bash
cd /opt/install/apache-hive-3.1.3/bin
# 进入beeline cli
beeline
```
在beeline cli下，先connect到hiveserver2服务，然后使用SQL建表和查询
```bash
# 注意这里是在beeline cli提示符下
!connect jdbc:hive2://bigdata03:10000 
输入bigdata03的用户名密码hadoop 123456后就连接成功
```
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230307143638.png)

```sql
-- 查看数据库和数据表
show databases;
show tables;
```
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230307144032.png)

## 建表
create table完成后，[用LOAD TABLE将txt数据加载入表](https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/LanguageManual_DML.html)，然后就可以用sql查询了。


```sql
DROP TABLE IF EXISTS `student`;
CREATE TABLE `student` (
`s_id` string,
`s_name` string,
`s_birth` string,
`s_sex` string
)
row format delimited
fields terminated by ','
lines terminated by '\n';
 
load data local inpath '/opt/install/data/student.csv' into table student;

DROP TABLE IF EXISTS `teacher`;
CREATE TABLE `teacher`(
`t_id` string,
`t_name` string
)
row format delimited
fields terminated by ','
lines terminated by '\n';
 
load data local inpath '/opt/install/data/teacher.csv' into table teacher;
```
## 查询
见[作业](https://github.com/tobyforever/geektime-bigdata-homework)

## 踩坑
中文乱码，注意create table的时候字段用string，而且load table加载数据的时候，输入的文本文件要统一按utf-8编码