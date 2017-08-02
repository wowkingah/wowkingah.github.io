---
layout: post
title: MySQL INSERT条件判断
date: 2017-08-08
tags: MySQL
---

### 简述  
INSERT条件判断：如果不存在则插入。与REPLACE是：如果存在则替换。  
对于普通的 INSERT 插入，如果想要保证不插入重复记录，可以通过唯一索引实现。  
那有没有不创建唯一索引，仅通过 INSERT 一条语句实现的方案呢？ INSERT INTO IF EXISTS就来了。  

### 语法  
```sql
INSERT INTO t_name(c1, c2, cn) SELECT 'v1', 'v2', 'vn' FROM DUAL WHERE NOT EXISTS(SELECT c1 FROM t_name WHERE c1 = ?);
```
*tips: DUAL 为临时表，无需创建。*  

### 示例  
#### 环境  
```shell
mysql> DROP TABLE IF EXISTS t1;
Query OK, 0 rows affected (0.20 sec)

mysql> CREATE TABLE `t1` (
    ->   `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
    ->   `name` varchar(8) NULL,
    ->   PRIMARY KEY (`id`)
    -> );
Query OK, 0 rows affected (0.78 sec)

mysql> INSERT INTO t1(name) VALUES ('aa'),('bb');
Query OK, 2 rows affected (0.06 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM t1;
+----+------+
| id | name |
+----+------+
|  1 | aa   |
|  2 | bb   |
+----+------+
2 rows in set (0.00 sec)
```
  
#### 测试  
```sql
mysql> INSERT INTO t1(name) VALUES('aa');
Query OK, 1 row affected (0.08 sec)

mysql> SELECT * FROM t1;
+----+------+
| id | name |
+----+------+
|  1 | aa   |
|  2 | bb   |
|  3 | aa   |
+----+------+
3 rows in set (0.00 sec)
```
*tips: 因name列无唯一索引，故aa可以插入。*  

```sql
mysql> INSERT INTO t1(name) SELECT 'bb' FROM DUAL WHERE NOT EXISTS(SELECT name FROM t1 WHERE name='bb');
Query OK, 0 rows affected (0.00 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM t1;
+----+------+
| id | name |
+----+------+
|  1 | aa   |
|  2 | bb   |
|  3 | aa   |
+----+------+
3 rows in set (0.00 sec)
```
*tips: bb未插入。*  


### reference  
https://my.oschina.net/jsan/blog/270161/