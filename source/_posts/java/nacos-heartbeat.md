---
title: nacos服务注册和心跳机制探究
date: 2023-06-14 20:46:51
tags: [java]
toc: false
published: false
---

参考[nacos源码分析系列文章](https://juejin.cn/post/7091058942125539364)对nacos的服务注册过程进行抓包分析。

坑1：注意windows下不好抓docker容器的包，所以nacos要用本机原生方式启动，不要用docker方式启动。
坑2：注意nacos里微服务注册到的ip网段是否正常，服务启动日志里也可以看到：
```
2023-06-14 15:21:47.725  INFO 45100 --- [           main] c.a.c.n.registry.NacosServiceRegistry    : nacos registry, DEFAULT_GROUP blade-admin 192.168.5.212:7002 register finished
```
，否则要用参考https://blog.csdn.net/Harry_Botter/article/details/125269877来手动调整服务的IP设置。注意“配置中心的的propertySource 优先级>启动命令行VM-Option的PropertySource>本地配置文件的PropertySource”

wireshark开启抓包，启动bladex的各个微服务后，过滤
