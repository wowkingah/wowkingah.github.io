---
layout: post
title: MySQL大表快速导入导出
date: 2016-11-30
tags: MySQL
---
  
MySQL大表快速导出常用两种方式：
> * mysqldump
> * INTO OUTFILE

### mysqldump
```bash
[root@db_node01 ~]# mysqldump --opt -uxxx -pxxx test t1 > /tmp/t1.sql
```
***tips***
> 适合整库或整表导出。  
> –opt：默认包含–add-drop-table –add-locks –create-options –disable-keys –extended-insert –lock-tables –quick –set-charset  
  

### INTO OUTFILE
```bash
mysql>  SELECT xxx FROM t1 WHERE xx INTO OUTFILE '/tmp/t1.sql';
```
***tips***
> 适合按指定条件导出表。 
> 可以通过FIELDS TERMINATED BY ‘字符串’ 来指定字段之间的分隔符，可以为单个或多个字符。默认为’\t’。  
> 可以通过FIELDS ENCLOSED BY ‘字符’ 来指定括住字段的值，只能为单个字符。默认情况下不使用任何符号，即当表中值为 TEST 时，导出后为 字符TEST符。  
  

## 根据以上导出方式，有两种快速导入方式：
> * SOURCE
> * LOAD DATA INFILE
  

### SOURCE
```bash
mysql> use db_name; source /tmp/t1.sql;
```
  

### LOAD DATA INFILE
```bash
mysql> use db_name; LOAD DATA INFILE '/tmp/t1.sql' INTO TABLE t1;
```
***tips***
> 如果SELECT … INTO OUTFILE时有指定FIELDS TERMINATED等参数，这里也需要指定。  
  

对于其它方式导入大表数据，可根据表ENGINE类型加以下参数：
### MyISAM
```bash
mysql> ALTER TABLE t_name DISABLE KEYS;
mysql> LOADING THE DATA ALTER TABLE t_name ENABLE KEYS;
```
***tips***
> 这两个命令用来打开或者关闭Myisam表非唯一索引的更新。对于导入大量数据到一个空MyISAM表，默认就是先导入数据然后才创建索引的，故不用进行设置。  
  

### InnoDB
```bash
mysql> SET UNIQUE_CHECKS=0;
mysql> SET AUTOCOMMIT=0;
```
***tips***
> 一个关闭唯一性校验，一个关闭自动提交。数据导完后，得改回来。  

