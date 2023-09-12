---
title: centos挂载数据盘
date: 2023-04-11 20:45:51
tags: linux
toc: false
published: true
---
### 目的：
做一个通用的java容器，有jdk8和curl，支持监控到服务状态不健康时自动重启，并且把工作目录/app挂载到外部，jar和application-prod.yml放在外部挂载目录里。

### 方法：
#### 1、构建基础镜像
用dockerfile封装一个带有curl和jdk8的自定义镜像。因为默认的镜像没有curl。
新建Dockerfile如下：
```
FROM openjdk:8

# Install curl
#RUN apt-get update && apt-get install -y curl
RUN echo "Asia/Shanghai" > /etc/timezone
RUN apt-get install -y curl
# Set environment variables
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64

```
在dockerfile目录下执行命令创建镜像（这里在wsl命令行下执行可能会遇到dns故障报错，改为在宿主机powershell下执行）：
```
 docker build -t tobyforever/jdk8:latest -f ./Dockerfile .
```
推送到dockerhub或者自定义的私有仓库registry（如nexus）
```
//这块是示意，需要自己根据实际仓库地址调整
docker login
docker push
```
这个自定义镜像如果要手动上传到内网的服务器，要导出tar上传到服务器后导入
```
docker save -o klqfjdk8.tar tobyforever/jdk8:latest

docker load < klqfjdk8.tar
```

#### 2、编写应用的docker-compose.yml

在docker-compose.yml中指定java启动参数、挂载目录路径、healthcheck检查方式。

注意jar所在的工作目录被整个挂载到了外部，方便更新发版时从外部替换更新jar和配置文件而不用重新打包镜像。
```
version: '3.7'
services:
  my-java-service:
    image: tobyforever/jdk8
    volumes:
      - ./app_deploy:/app
    working_dir: /app
    command: java -jar blade-api.jar --spring.profiles.active=prod --server.port=18080
    ports:
      - "18080:18080"
    restart: always
    healthcheck:
      test: ["CMD-SHELL", "curl -sS 'http://localhost:18080' || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 60s
```
注意的点: 
1、因为jar运行在容器中，所以application-prod.yml文件里的url都要用宿主机的实际ip，不要用localhost不然就会指到容器内部了。
2、这里并没有用host网络模式，所以需要指定好暴露的服务端口

#### 3、实现服务自愈
启动服务
```
docker-compose up -d
docker ps
docker inspect 容器id
```
可以发现，经过start_period时间后，healthcheck执行了，服务的status变成了unhealthy，但是！容器没有自动重启。原因是docker本身并不会根据容器unhealthy状态来自动重启，哪怕是配置了restart:always. （restart:always只会在容器运行退出了后重启、而不是根据healthcheck来重启）

要想根据healthcheck自动重启，需要启动一个外部的专用镜像[autoheal](https://hub.docker.com/r/willfarrell/autoheal)来监控并发送重启命令。
```
docker run -d --name autoheal --restart=always -e AUTOHEAL_CONTAINER_LABEL=all -v /var/run/docker.sock:/var/run/docker.sock willfarrell/autoheal
```
启动这个autoheal后，它会通过docker.sock来监控并自动重启unhealthy的容器。

```
docker ps
docker-compose logs -f
```
通过docker ps以及java服务容器的日志可以看到，容器定时收到了healthcheck的ping，并且如果检测到unhealthy就会自动被重启。

#### 其他优化

日期落后8小时