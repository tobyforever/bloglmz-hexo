---
title: centos挂载数据盘
date: 2023-04-11 20:45:51
tags: linux
toc: false
published: true
---

1、找磁盘路径：fdisk -l
2、格式化磁盘： mkfs -t ext4 /dev/vdb
3、挂载：
mkdir /data
mount /dev/vdb /data
4、执行以下命令，查看挂载结果
df -TH
5、开机自动挂载，修改 /etc/fstab 文件。
vi /etc/fstab
/dev/vdb /data   ext4 defaults     0   0
7.保存退出， 执行以下命令，检查 /etc/fstab 文件是否写入成功。
mount -a