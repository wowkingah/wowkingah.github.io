---
layout: post
title: Navicat查询MySQL数据库BLOB字段问题
date: 2017-12-29
tags: MySQL SQL
---

### 简述  
BLOB是啥？  
MySQL对BLOB的定义：`A BLOB is a binary large object that can hold a variable amount of data. `  
百度百科对BLOB的定义：`BLOB(binary large object)，二进制大对象，是一个可以存储二进制文件的容器。`  

### 模拟环境  
#### 建表与初始数据  
```bash
mysql> CREATE TABLE `t_blob` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `info` BLOB,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB;
mysql> INSERT INTO t_blob(info) VALUES('www.google.com'),('www.baidu.com');
Query OK, 2 rows affected (0.03 sec)
Records: 2  Duplicates: 0  Warnings: 0
```

### 问题  
通过`Navicat`查询`MySQL`数据库中`BLOB`字段，返回结果异常，显示为`(BLOB) 14 bytes`，如下图：  
![](/images/posts/dba/select_blob_of_mysql_for_navicat/select_from_navicat_1.jpg)  
Are you kidding me???  

### 排查  
`Navicat`查询结果拷到文本编辑器，显示正常：  
![](/images/posts/dba/select_blob_of_mysql_for_navicat/copy_to_text.jpg)  
`Navicat`以备注的方式查看，显示正常：  
![](/images/posts/dba/select_blob_of_mysql_for_navicat/select_from_navicat_2.jpg)  
`MySQL`客户端查询，返回结果也是正常：  
![](/images/posts/dba/select_blob_of_mysql_for_navicat/select_from_mysql_client.jpg)  
不过，就是想通过`Navicat`去查，怎么办？  

### 解决方案：通过函数转换  
#### 方法一：CAST  
![](/images/posts/dba/select_blob_of_mysql_for_navicat/select_from_navicat_3.jpg)  

#### 方法二：CONCAT  
![](/images/posts/dba/select_blob_of_mysql_for_navicat/select_from_navicat_4.jpg)  



