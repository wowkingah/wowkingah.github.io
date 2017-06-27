---
layout: post
title: MySQL清理binary日志
date: 2017-06-27
tags: MySQL binary
---

### 简述  
当MySQL开启binary log（二进制日志）后，会产生大量日志文件，时间久了会占用不少磁盘空间。  

### 解决方案  
> 1. 自动清理：expire_logs_days  
> 2. 手动清理：PURGE  
> 3. 重置日志：RESET  

#### 自动清理  
自动清理可通过expire_logs_days参数来实现。  
* MySQL中动态修改  
```shell
mysql> SHOW GLOBAL VARIABLES LIKE 'expire_logs_days';  
mysql> SET GLOBAL expire_logs_days =15;  
```

* 写入my.cnf配置文件  
`expire_logs_days =15`  

#### 手动清理  
**语法**  
```shell
PURGE {MASTER | BINARY} LOGS TO 'binary_log_name'   
PURGE {MASTER | BINARY} LOGS BEFORE 'date'   
```
*tips: 删除二进制日志索引（index）中指定日志或日期之前的二进制日志。*  

**实例**  
```shell
mysql> SHOW MASTER LOGS;  
# 15天前  
mysql> PURGE MASTER LOGS BEFORE DATE_SUB(CURRENT_DATE, INTERVAL 15 DAY);  
#删除日志文件到mysql-bin.000152  
mysql> PURGE MASTER LOGS TO 'mysql-bin.000152';   
```
  
#### 重置日志  
高危命令，会清理所有要清理的日志，一般不常用。  
```shell
mysql> RESET MASTER;  
```
  

