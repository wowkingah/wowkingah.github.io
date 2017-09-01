---
layout: post
title: Oracle 超过最大进程数
date: 2017-09-01
tags: Oracle
---

### 简述  
Oracle超过最大进程数。  

### 报错  
#### 客户端连接报错  
ORA-12519: TNS:no appropriate service handler found  

#### 命令行连接报错  
```shell
SQL> conn /as sysdba;  
ERROR:  
ORA-00020: maximum number of processes (150) exceeded  
```

### 解决方案  
> 1.重启或停止问题客户端（如问题Tomcat/Nginx），释放几个空闲连接；
> 2.查看当前进程数；
```shell
SQL> select count(1) from v$process;  

  COUNT(1)  
----------  
       147  
```
> 3.查看最大进程数；
```shell
SQL> select value from v$parameter where name = 'processes';  

VALUE  
--------------------------------------------------------------------------------  
150  
```
> 4.修改最大进程数；
```shell
SQL> alter system set processes = 512 scope = spfile;  
```
> 5.重启数据库。  
```shell
SQL> conn /as sysdba;  
SQL> shutdown immediate;  
SQL> startup;  
```  
