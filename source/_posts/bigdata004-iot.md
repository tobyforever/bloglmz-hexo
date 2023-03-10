---
title: 大数据第4课 物联网大数据
date: 2023-02-24 20:43:51
tags: 大数据
toc: true
---

# What?
需要对大数据技术选型，解决物联网场景下的数据存储与分析问题
需要实际进行benchmark，了解各个技术性能的配置要求和性能baseline


# Why?
(1) 传统关系型数据库如oracle难以适应物联网时序数据的要求
(2) hadoop的大数据组件在物联网场景下如何应用？
(3) 新型专用时序数据库在实际使用过程中可能有哪些坑？

# How?
## 架构流程图
系统的流程图如下，
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230310114354.png)

emqx是erlang做的物联网网关，能方便接入mqtt协议，具有高性能支持高并发的特点。
[emqx车联网的介绍pdf](https://assets.emqx.com/resources/white-papers/iov-platform-building-from-beginner-to-master.pdf)
