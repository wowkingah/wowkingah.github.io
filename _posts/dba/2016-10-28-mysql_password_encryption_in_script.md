---
layout: post
title: 加密脚本中MySQL用户密码
date: 2016-10-28
tags: MySQL
---
  
  MySQL 5.6.6开始支持mysql_config_editor小工具。该工具将创建一个login-path，用于存储数据库的身份认证信息（仅仅只是存储数据库的用户密码等信息，实际的用户密码还是数据库创建）。
  当脚本等一些地方需连接数据库时，可避免暴露用户明文密码，以增强数据库安全。
  

## 语法
### 创建login-path
***数据库中已有backup用户***
```bash
[root@ZTEST-218 ~]# mysql_config_editor set --login-path=backup_db --user=backup --password --host=localhost --port=3306
Enter password:
```
  

### 查看login-path
```bash
[root@ZTEST-218 ~]# mysql_config_editor print --all
[db_backup]
user = backup
password = *****
host = localhost
port = 3306
```
  

### 查看login-path存储路径及格式
```bash
[root@ZTEST-218 ~]# file ~/.mylogin.cnf
/root/.mylogin.cnf: data
```
  

### 删除login-path
```bash
[root@ZTEST-218 ~]# mysql_config_editor remove --login-path=backup_db
[root@ZTEST-218 ~]# mysql_config_editor print --all
[root@ZTEST-218 ~]# mysqldump --login-path=backup_db test > test.sql
mysqldump: Got error: 1045: Access denied for user 'root'@'localhost' (using password: NO) when trying to connect
```
  


***tips***
mysqldump在脚本中使用login-path备份，将不再提示Warning: Using a password on the command line interface can be insecure.
```bash
[root@ZTEST-218 ~]# mysqldump --login-path=db_backup test > test.sql
[root@ZTEST-218 ~]# ll test.sql
-rw-r--r-- 1 root root 169259 Oct 28 14:11 test.sql
```

  
### reference
http://dev.mysql.com/doc/refman/5.6/en/mysql-config-editor.html