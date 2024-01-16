---
title: docker使用培训
date: 2023-12-17 20:45:51
tags: linux
toc: false
published: true
---
### docker是什么：
docker可以理解为把应用程序和操作系统一起打包成一个容器，整个容器在宿主操作系统中运行。因为基于linux内核隔离技术namespace和cgroup，相比传统的虚拟机（如vmware）而言对性能影响较小。

### 使用docker的好处：
- 1、可以快速搭建出各种程序，只要有配置文件，用同样的命令启动即可，如果出问题可以直接删掉重新创建。传统的应用手动安装方法可能会对宿主机环境造成污染。
- 2、app作为docker容器发布，运行环境（容器操作系统）在开发环境和生产环境中都是一样的，就不容易出现开发环境和生产环境操作系统不一致导致的bug。
- 3、 第一点里的“启动即运行，用完直接删”的特点非常适合云环境（不用考虑底层的硬件资源差异，只要有docker环境就能运行），所以在国产信创项目中只要是x86架构CPU能跑docker（UOS服务器版兼容CentOS8）就能运行应用程序，从而避免一些国产操作系统软件或者硬件对运行环境的影响。



### docker基本命令和相关概念：
#### 1、镜像和容器
镜像相当于是程序，容器相当于是程序运行起来的实例。

查看已下载的镜像
```
docker images
```

用指定镜像启动容器
```
//-d 后台运行 -p网络端口映射  --name 容器命名  使用镜像
docker run -d -p 80:80 --name nginx nginx
```

注意事项：
- 1、数据是在容器内部的，这里还没有做容器内部目录和外部存储的绑定，容器一旦销毁数据就丢了。要保留容器数据，需要把数据目录映射到外部目录，或者映射到docker的数据卷（volumn）。直接映射本地目录比较直观方便编辑内容，但有时候启动时可能会有文件权限问题导致容器启动失败； 数据卷方式是比较正规安全的方式，能保证容器正常启动，缺点是文件查看编辑相对繁琐一些。
  https://cloud.tencent.com/developer/article/1540710
- 2、docker安装默认是安装在/var/lib/docker下的，而宿主机服务器的系统盘一般空间有限，生产环境安装docker后要把docker数据目录指向数据盘。


查看运行中的容器
```
docker ps
```

进入运行中的容器的bash命令行
```
docker exec -it 容器id或容器名 bash
```

查看容器的控制台日志
```
docker logs -f 容器id或容器名
```


#### 2、编写应用的docker-compose.yml

如果每次都用docker run来启动应用，如果应用由多个镜像组成多个服务的话需要编写脚本，会比较麻烦。
官方提供了应用编排工具docker-compose，把docker run里需要的参数都以yaml格式携程配置文件docker-compose.yaml以后，可以用一样的命令来启动容器。
```bash
#启动服务
docker-compose up -d 即可启动。-d表示后台运行。
#重启
docker-compose restart
#停止
docker-compose down
#停止容器并且删除数据卷volumn（数据卷里的数据也会被丢弃）。
docker-compose down -v
```
使用docker-compose配置MinIO服务的例子参见 E:\lib\minio-docker

#### 3、使用docker-compose来封装java应用，并支持健康状态监控自动重启

docker-compose可以配置健康检查探针命令，例如定期curl请求指定服务接口根据返回值判断服务是否正常，如果不正常则可以自动重启服务。

配置例子见docker-autoheal.md