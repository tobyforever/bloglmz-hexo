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
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230310160721.png)

emqx是erlang做的物联网网关，能方便接入mqtt协议，具有高性能支持高并发的特点。
[emqx车联网的介绍pdf](https://assets.emqx.com/resources/white-papers/iov-platform-building-from-beginner-to-master.pdf)

emqx的商业版提供了数据持久化和到webhook、kafka、mysql等多种sink的数据传输（data bridge）功能，开源版中databridge只保留了webhook。可以通过自己写webhook来通过http接口接收emqx的消息再转发到kafka。

## emqx快速入门
使用docker-compose启动emqx服务[官网](https://www.emqx.io/docs/zh/v5.0/deploy/install-docker.html#%E9%80%9A%E8%BF%87-docker-compose-%E6%9E%84%E5%BB%BA-emqx-%E9%9B%86%E7%BE%A4)
docker-compose.yml如下：
```yaml
version: '3'

services:
  emqx1:
    image: emqx:5.0.19
    container_name: emqx1
    environment:
    - "EMQX_NODE_NAME=emqx@node1.emqx.com"
    - "EMQX_CLUSTER__DISCOVERY_STRATEGY=static"
    - "EMQX_CLUSTER__STATIC__SEEDS=[emqx@node1.emqx.com,emqx@node2.emqx.com]"
    healthcheck:
      test: ["CMD", "/opt/emqx/bin/emqx_ctl", "status"]
      interval: 5s
      timeout: 25s
      retries: 5
    networks:
      emqx-bridge:
        aliases:
        - node1.emqx.com
    ports:
      - 1883:1883
      - 8083:8083
      - 8084:8084
      - 8883:8883
      - 18083:18083 
    # volumes:
    #   - $PWD/emqx1_data:/opt/emqx/data

  emqx2:
    image: emqx:5.0.19
    container_name: emqx2
    environment:
    - "EMQX_NODE_NAME=emqx@node2.emqx.com"
    - "EMQX_CLUSTER__DISCOVERY_STRATEGY=static"
    - "EMQX_CLUSTER__STATIC__SEEDS=[emqx@node1.emqx.com,emqx@node2.emqx.com]"
    healthcheck:
      test: ["CMD", "/opt/emqx/bin/emqx_ctl", "status"]
      interval: 5s
      timeout: 25s
      retries: 5
    networks:
      emqx-bridge:
        aliases:
        - node2.emqx.com
    # volumes:
    #   - $PWD/emqx2_data:/opt/emqx/data

networks:
  emqx-bridge:
    driver: bridge
```
使用doker-compose up -d启动，会启动2个节点的emqx集群。
启动后在控制台可以看端口：
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230310161501.png)
控制台默认地址是localhost:18083用户名admin密码public,第一次进去需要修改密码。
 
下载MQTTX客户端进行mqtt测试，连接刚才docker启动的emqx。注意默认系统没有开启认证所以不需要用户名密码就能连接了。
新建一个监听的topic test123/#
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230310161738.png)
新建一个监听的topic,名字是testtopic123/#  ，  然后在消息发送窗口发送消息，发送的topic填写testtopic123/1  (topic格式是模糊匹配的)，点击发送，就可以看到监听收到了发出的消息。
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230310161926.png)

## emqx使用webhook做消息转发
在控制台里的rules页面里新建一个转发规则rule，把testtopic123/#这个topic的消息转发到action，新建一个webhook作为action，webhook实际上就是一个接受json的post http接口地址。
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230310162415.png)

在webhook设置里把主题topic、客户端clientid、消息payload都放在body里传过去：
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230310162644.png)
在MQTTX工具里点击发送，就能在webhook http接口里看到收到了emqx转发过来的消息。
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230310162944.png)

## webhook将消息写入kafka
如果消息量不大，webhook http接口可以直接用springboot写。消息并发量大的情况下，建议用vertx之类的reactive框架编写，防止大量并发进程占用系统资源。

