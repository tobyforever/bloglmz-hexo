---
title: 大数据第1课 hadoop
date: 2023-02-20 20:43:51
toc: false
tags: 大数据
---

# What?
hadoop是分布式文件系统，文件uri格式为“hdfs://path-to-file”

# Why?
使用廉价的x86机器搭建的集群代替高端存储，hdfs分布式文件系统具有可扩展、多副本高可用的特点。

# How?

## 实验环境
vmware里启动bigdata01、bigdata02、bigdata03虚拟机，对应的IP分别为192.168.52.100  192.168.52.110  192.168.52.120。在本地宿主机上配置好hosts文件里域名指向（bigdata01 bigdata02 bigdata03）

## 启动hadoop

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

## 基本操作命令
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

## idea的bigdata-tool大数据窗口连接hadoop
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230307110700.png)
这里8020端口是在HADOOP_HOME/etc(路径/opt/install/hadoop-3.3.4/etc/hadoop
)下的core-sites.xml里定义的
```xml
<property>
        <name>fs.defaultFS</name>
        <value>hdfs://bigdata01:8020</value>
</property>
```


## yarn资源调度
yarn 是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台，而 MapReduce/Spark 等运算程序则相当于运行于操作系统之上的应用程序。
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230731114741.png)
YARN 主要由 ResourceManager、NodeManager、ApplicationMaster 和 Container 等组件构成。
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230731114908.png)

RM
只是jobtracker中“资源管理”的角色，只负责运行中应用程序资源的分配，而不管监控应用程序和状态跟踪。

本质：是一个独立的守护进程，运行在专有的机器上，机器的配置要足够好

RM的功能
RM处理客户端请求，接收jobsubmitterf提交的作业，按照作业的上下文（ Context）信息，以及从 Nodemanager(NM）收集来的状态信息，启动调度过程，分配一个 Container作为 App Master；
RM拥有为系统中所有应用资源分配的決定权，是中心服务，做的事情就是调度，启动每一个job所属的 Application、另外监控 Applicationg的存在情况am
与运行在每个节点上的NM进程交互，通过心跳通信，达到监控NM的目的
RM有一个可插拔的调度器组件 Scheduler
不负责应用程序的监控和状态跟踪（AM)
不保证应用程序失败或者硬件失败的情况下对Task的重启（AM）


AM
只是jobtracker中“任务调度”的角色 。只有在任务的整个生命周期内，AM才是启动状态，当任务执行完毕，AM消失，不需要监控。

本质：是一个特殊的container，管理其他的container

AM的作用
根据提交的任务和RM的反馈，去寻找干活的人(NodeManager)。

AM的启动流程
Client向RM发出请求；
RM返回一个 ApplicationID作为回应；
Clientl向RM回应 Application Submission Context(ASC)。ASC包括 Applicationid、user、 queue，以及其他一些启动AM相关的信息，除此之外，还有一个 Container Launch Context(CLC)，CLC包含了资源请求数（内存与CPU), job files，安全 token，以及其他些用以在一个node上启动AM的信息。任务一旦提交以后， clients可以请求RM去杀死应用或查询应用的运行状态；
当RM接受到ASC后，它会调度一个合适的Node Manager，来启动一个container（这个 containe经常被称作为 container0）并运行Application Master实例。如果没有合适的 container，AM就不能启动。当有合适的 container时，RM发请求到合适的NM上，来启动AM。这时候，AM的PRC与监控的URL就已经建立了；
当AM启动起来后，RM回应给AM集群的最小与最大资源等信息。这时AM必须決定如何使用当前可用的资源。YARN不像那些请求固定资源的 scheduler，它能够根据集群的当前状态动态调整；
AM根据从RM那里得知的可使用的资源，它会请求一些一定数目的 container；
RM将会根据调度策略，尽可能的满足AM申请的 container。AM获取到container资源执行分布式计算任务，并监控和管理作业状态。当任务完成之后，AM会通知RM释放 container资源。
NM
相当于是tasktracker，这里NM和RM通过心跳进行通讯。

NM的作用
是 slave进程，类似 Tasktracker的角色，是每个机器框架代理；
处理来自RM的任务请求；
接收并处理来自 Applicationmaster的 Container启动、停止等各种请求；
负责启动应用程序的 container（执行应用程序的容器），并监控他们的资源使用情況（CPU、内存、磁盘和网络），并报告给RM；
总的来说，在单节点上进行资源管理和任务管理；


配置文件：maped-site.xml (mapred=mapreduce)和yarn-site.xml
Yarn集群必须在ResourceManager所在节点启动
```
start-yarn.sh
```
Yarn集群的可视化是ResourceManager所在节点机器的IP地址加端口号：8088

#### Yarn工作机制
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230731115129.png)

MR 程序提交到客户端所在的节点。
YarnRunner 向ResourceManager 申请一个 Application。
RM 将该应用程序的资源路径返回给 YarnRunner。
该程序将运行所需资源提交到HDFS 上。
程序资源提交完毕后，申请运行 mrAppMaster。
RM 将用户的请求初始化成一个 Task。
其中一个NodeManager 领取到Task 任务。
该NodeManager 创建容器Container，并产生MRAppmaster。
Container 从HDFS 上拷贝资源到本地。
MRAppmaster 向RM 申请运行MapTask 资源。
RM 将运行 MapTask 任务分配给另外两个 NodeManager，另两个NodeManager 分别领取任务并创建容器。
MR 向两个接收到任务的NodeManager 发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask 对数据分区排序。
MrAppMaster 等待所有MapTask 运行完毕后，向RM 申请容器，运行ReduceTask。
ReduceTask 向 MapTask 获取相应分区的数据。
程序运行完毕后，MR 会向RM 申请注销自己。

#### Yarn作业提交过程
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230731115302.png)

