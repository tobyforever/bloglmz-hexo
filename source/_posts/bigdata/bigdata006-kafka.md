---
---
title: 大数据第6课 kafka的使用
date: 2023-02-25 20:43:51
tags: 大数据
toc: false
published: true
---

# What?
掌握kafka的主要概念和使用要点

# Why?
kafka是大数据技术中起到数据hub作用的重要组件。

# How?
## kafka

## 基本概念
kafka可以用来实现event streaming的架构。event streaming是把系统通过事件数据流的方式进行存储、处理、消费，从而实现系统的解耦。数据的来源类型包括传感器物联网设备、数据库、系统日志等。
kafka涉及的概念包括event、producer、consumer，topic，和传统的消息队列订阅发布概念一致。
kafka作为分布式系统，topic是partitioned的，也就是分区，目的是通过增加处理的并行度提高性能。
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230310172829.png)
kafka作为分布式系统，还有replica（副本）的概念，也就是将一份数据复制为多份，从而实现High Availability(高可用)和fault-tolerant（r容错）。生产环境中的replication factor通常会设置为3（一个数据有3份）。

## 启动kafka
参考[官方quick start](https://kafka.apache.org/quickstart)
kafka最新版本号是3.4，其中和scala配合使用的时候需要考虑对应2.12和2.13版本，虽然kafka推荐的是2.13的scala但目前flink等用的还是2.12scala。

以前kafka依赖于zookeeper，最新的kafka可以不用zookeeper了。



参考
[Kafka的压测和配置](https://blog.csdn.net/weixin_44275820/article/details/119869999)