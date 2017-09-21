---
layout: post
title: MySQL server has gone away
date: 2017-09-21
tags: MySQL
---

### 简述  
比较常见的一个报错日志：  
`ERROR 2006 (HY000): MySQL server has gone away`  

### 原因  
#### 原因一：MySQL 服务宕了  
##### 排查方法：  
**1.1 查看MySQL运行时长**  
```shell
mysql> show global status like 'uptime';  
+---------------+-------+  
| Variable_name | Value |  
+---------------+-------+  
| Uptime        | 138   |  
+---------------+-------+  
```  

**1.2 查看MySQL日志**  
```shell
[root@ZTEST-218 ~]# tail /data/logs/mysql/mysqld.log 
2017-09-21 11:12:47 7f13935fe700 InnoDB: Loading buffer pool(s) from .//ib_buffer_pool
2017-09-21 11:12:48 9744 [Note] Server hostname (bind-address): '*'; port: 63306
2017-09-21 11:12:48 9744 [Note] IPv6 is available.
2017-09-21 11:12:48 9744 [Note]   - '::' resolves to '::';
2017-09-21 11:12:48 9744 [Note] Server socket created on IP: '::'.
2017-09-21 11:12:48 9744 [Warning] 'proxies_priv' entry '@ root@ztest-217' ignored in --skip-name-resolve mode.
2017-09-21 11:12:48 9744 [Note] Event Scheduler: Loaded 0 events
2017-09-21 11:12:48 9744 [Note] /usr/sbin/mysqld: ready for connections.
Version: '5.6.34-log'  socket: '/var/lib/mysql/mysql.sock'  port: 63306  MySQL Community Server (GPL)
2017-09-21 11:12:50 7f13935fe700 InnoDB: Buffer pool(s) load completed at 170921 11:12:50
```  
如果Uptime的值比较大，说明MySQL运行时间长，最近一段时间内没有重启服务之类的。  
如果MySQL错误日志中没有相关日志记录，也说明MySQL最近未重启。  

#### 原因二：连接超时  
##### 排查方法：  
如果程序使用的是长连接，则这种情况的可能性会比较大。  
即某个长连接很久没有活动，达到了MySQL Server端的time out，被Server强行关闭。此后再通过这个Client发起查询的时候，就会报错`MySQL server has gone away`。  
```shell
mysql> show global variables like 'wait_timeout';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 3600  |
+---------------+-------+
1 row in set (0.00 sec)

mysql> select now();
+---------------------+
| now()               |
+---------------------+
| 2017-09-21 11:27:31 |
+---------------------+
1 row in set (0.00 sec)

mysql> SET SESSION wait_timeout=5;		#将wait_timeout修改为5s
Query OK, 0 rows affected (0.00 sec)

mysql> select now();		#等待一段时间后，再次执行SQL
ERROR 2006 (HY000): MySQL server has gone away
No connection. Trying to reconnect...
Connection id:    7
Current database: *** NONE ***

+---------------------+
| now()               |
+---------------------+
| 2017-09-21 11:27:53 |
+---------------------+
1 row in set (0.00 sec)
```  
当连接sleep时间超过wait_timeout时间后，再次发起活动查询时，就会报这个错。  

#### 原因三：进程在Server端被主动`kill`  
##### 排查方法：  
这种原因和第2种原因比较相似，只是发起者是DBA，或者其他job，发现有长时间的慢查询执行kill xxx导致。  
```shell
mysql> show global status like 'com_kill';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Com_kill      | 1     |
+---------------+-------+
1 row in set (0.00 sec)
```  

#### 原因三：Your SQL statement was too large  
##### 排查方法：  
当查询的结果集超过`max_allowed_packet`也会出现这样的报错，定位方法是打出相关报错的语句。 用 `select * into outfile` 的方式导出到文件，查看文件大小是否超过 `max_allowed_packet`，如果超过则需要调整参数，或者优化语句。  
```shell
mysql> show global variables like 'max_allowed_packet';
+--------------------+----------+
| Variable_name      | Value    |
+--------------------+----------+
| max_allowed_packet | 16777216 |
+--------------------+----------+
1 row in set (0.00 sec)  

mysql> set global max_allowed_packet=1024*1024*32;		#修改参数
Query OK, 0 rows affected (0.00 sec)

mysql> show global variables like 'max_allowed_packet';
+--------------------+----------+
| Variable_name      | Value    |
+--------------------+----------+
| max_allowed_packet | 33554432 |
+--------------------+----------+
1 row in set (0.00 sec)
```  

### reference  
http://cenalulu.github.io/mysql/mysql-has-gone-away/