---
title: oracle笔记
date: 2023-04-11 20:43:51
tags: 数据库
toc: true
published: true
---

# windows解决开了archivelog导致oracle无法启动

fink-cdc需要开启archivelog，但是oracle的archivelog空间如果满了会导致oracle服务无法启动，表现为连接不上数据库。
需要清理archivelog空间
PowerShell:
```powershell
sqlplus.exe / as sysdba
shutdown abort
startup mount
```

另外开PowerShell窗口：
```powershell
rman.exe target  /
RMAN> crosscheck archivelog all
RMAN>delete noprompt archivelog until time "sysdate -3";
```

# 自动清理archivelog空间
[自动清理Oracle的Archivelog](https://blog.csdn.net/jump_gu/article/details/116159584)
[windows上设置自动删除oracle归档日志](https://blog.csdn.net/lixiaohuiok111/article/details/78119683?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-4-78119683-blog-116159584.235^v28^pc_relevant_t0_download&spm=1001.2101.3001.4242.3&utm_relevant_index=7)


# oracle创建用户和表空间

例子：创建表空间
```sql
create tablespace DEMOSPACE 
datafile 'E:/oracle_tablespaces/DEMOSPACE_TBSPACE.dbf' 
size 1500M 
autoextend on next 5M maxsize 3000M;
```
删除表空间
```sql
drop tablespace DEMOSPACE including contents and datafiles
```

完整例子
```sql
--表空间
CREATE TABLESPACE sdt
DATAFILE 'F:\tablespace\demo' size 800M
         EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO; 
--索引表空间
CREATE TABLESPACE sdt_Index
DATAFILE 'F:\tablespace\demo' size 512M         
         EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;     
 
--2.建用户
create user demo identified by demo 
default tablespace sdt;
 
--3.赋权
grant connect,resource to demo;
grant create any sequence to demo;
grant create any table to demo;
grant delete any table to demo;
grant insert any table to demo;
grant select any table to demo;
grant unlimited tablespace to demo;
grant execute any procedure to demo;
grant update any table to demo;
grant create any view to demo;
```

# 导入导出数据
```sql
--导入导出命令   
ip导出方式： exp demo/demo@127.0.0.1:1521/orcl file=f:/f.dmp full=y
exp demo/demo@orcl file=f:/f.dmp full=y
imp demo/demo@orcl file=f:/f.dmp full=y ignore=y
```