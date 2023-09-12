---
title: 达梦数据库笔记
date: 2023-05-13 20:43:51
tags: 数据库
toc: false
published: true
---

# 无法调整屏幕分辨率
麒麟系统虚拟机默认分辨率800x600太小导致选择大分辨率后点不到保存按钮。
解决办法：在命令行执行xrandr -s 9 然后再调整，建议选到1920X1200。 
分辨率太小会导致管理工具新建用户界面没滚动条选不到表空间。




# 以图形界面安装数据库软件时报错

Exception in thread "main" org.eclipse.swt.SWTError: No more handles [gtk_init_check() failed]
    at org.eclipse.swt.SWT.error(SWT.java:4109)

问题原因
当前操作系统的登录用户应该为非dmdba用户，如果在当前会话中启用图形界面需要将图形界面权限放开。

解决方法
切换到root用户后在命令行中输入xhost +，可以使得dmdba可以调用图形界面进行安装。

注意xhost和+中间有个空格。

总结：
登陆系统后需要
xhost +
echo $DISPLAY
su - dmdba
export DISPLAY=:0.0
否则非root用户会没有GUI图形界面权限

# 字符集设置
簇大小、页大小、字符集、大小写敏感、VARCHAR 类型以字符为单位等一旦指定，数据库
创建完成将无法更改。
页是达梦数据库的最小存储单元，簇是由连续的页组成，簇是达梦数据库的最小分配单元。
达梦中 varchar 类型长度默认不能大于页大小的一半。
数据库页大小 varchar 实际最大长度（字节）
4K 约 1900
8K 约 3900
16K 约 8000
32K 约 8188
VARCHAR 类型以字符为单位（默认以字节为单位）
以字节为单位：
Varchar(10), gb18030 字符集，一个中文占用两个字节，Varchar(10)只能保存五个中文。
Varchar(10), utf-8 字符集，一个中文占用三个字节，Varchar(10)只能保存三个中文。
以字符为单位：
Varchar(10), gb18030 时，相当于 Varchar(20),可以保存 10 个中文；
Varchar(10), utf-8 时，相当于 Varchar(40),可以保存 13 个中文；

# 密码
要求9位以上，测试用所以设置了1234567890

# 关闭防火墙
装好数据库后，记得关闭防火墙
systemctl status firewalld
systemctl stop firewalld
systemctl disable firewalld

# 默认装完没有$DM_HOME
教程里$DM_HOME安装完成后没有自动生效，需要重启或者source profile，或者用/dm8代替。考试不要重启，建议直接用dm8代替。

# 图形化管理工具启动
用dmdba账户，运行 /dm8/tool/manager


# 打开manager报错.fileTableLock“权限不够”
如果报错“权限不够”，一般是因为之前用 root 用户打开过 DM 管理工具，导致了 tool 目录
下 workspace 中的文件夹权限发生变化，导致 dmdba 用户没有权限。
解决方法：
使用 root 执行如下命令更改 DM 安装 tool 目录下所属用户为 dmdba（DM 安装用户）：
chown -R dmdba:dinstall /dm8/tool

# disql命令行工具连接数据库
注意要区分是tool下的还是bin下的，
正常的是bin下的。tool下的默认是没有连接的状态
```
./disql sysdba/1234567890:5238
```
注意上面5238是配置数据库服务的时候指定的端口。达梦可以有多个数据库实例，名字不同，连接时可以端口号区分。

# 编辑数据库服务的别名
网络配置助手实际写的是 dm_svc.conf 文件，也可以直接手工编辑该文件。
[dmdba@KylinDCA03 tool]$ cat /etc/dm_svc.conf
TIME_ZONE=(480)
LANGUAGE=(cn)
DM=(127.0.0.1:5236)
DMTEST=(127.0.0.1:5238)

这一步需要手动做，安装完成后没有自动。

# 新建和修改表空间

在图形管理工具里修改和新建表空间
表空间的文件路径参考/dm8/data/DMTEST/MAIN.DBF
其中DMTEST是实例名，MAIN是表空间名

建好表空间后，后面新建用户时就可以选择创建好的表空间作为表空间和索引表空间。

# 导入数据
直接用sql查询界面执行语句
start sql文件路径.sql

# 冷备还原
表空间的还原与恢复只有两个步骤：restore +recover
其中restore是从物理备份中恢复，recover是从归档中还原数据到最新。
表空间恢复要求需要同一个库（db_magic相同）


库级还原恢复三部曲：
restore、recover、更新数据库魔数（update db_magic）。
DM 数据库还原和恢复支持跨库跨操作系统。目标库和源库可以不是同一个库.

# 参考
[安装系统](https://www.bilibili.com/video/BV1XG4y1i7w1/?spm_id_from=333.788.recommend_more_video.2&vd_source=bb33ada5ced7817c4c4746122e65a6aa)


[管理实例](https://www.bilibili.com/video/BV1yW4y1278k/?spm_id_from=333.788.recommend_more_video.2&vd_source=bb33ada5ced7817c4c4746122e65a6aa)


[管理表空间](https://www.bilibili.com/video/BV1Ce4y1Q7Rh/?spm_id_from=333.788.recommend_more_video.-1&vd_source=bb33ada5ced7817c4c4746122e65a6aa)


[管理用户和权限](https://www.bilibili.com/video/BV1BG4y1i7Zo/?spm_id_from=333.788.recommend_more_video.0&vd_source=bb33ada5ced7817c4c4746122e65a6aa)


[导入数据](https://www.bilibili.com/video/BV1WS4y1t7om/?spm_id_from=333.788.recommend_more_video.6&vd_source=bb33ada5ced7817c4c4746122e65a6aa)

[开归档](https://www.bilibili.com/video/BV1dS4y147BQ/?spm_id_from=333.788.recommend_more_video.8&vd_source=bb33ada5ced7817c4c4746122e65a6aa)

在图形化管理工具里把数据库状态改为mount，然后mkdir创建归档存放目录：
su dmdba
cd /dm8
mkdir arch1
然后选择归档目录指向/dm8/arch1，确定保存后把数据库状态改为open即可。最后在数据库的系统概览页面可以看到归档开启状态。

[物理备份和逻辑备份](https://www.bilibili.com/video/BV1N94y1D7ut/?spm_id_from=333.788.recommend_more_video.0&vd_source=bb33ada5ced7817c4c4746122e65a6aa)

先用root在/dm8/tool下dmservice.sh打开数据库系统服务管理界面，把数据库实例服务停掉。
再/dm8/tool/console打开，在console里可以备份。备份完成后重新启动数据库实例服务。


注意console做数据恢复的时候要用dmdba操作，否则数据文件读写会报权限错误。

用dexp命令行逻辑备份：
dmdba用户在bin下 ./dexp SYSDBA/1234567890@localhost:5236 file=dexp01.dmp log=dexp01.log directory=/dm/backup/dexp/ full=y

dexp 逻辑导出、dimp 逻辑导入四个级别：
全库（full=y）
按用户（owner=XXX）
按模式（schemas=XXX）
按表（tables=XX）



