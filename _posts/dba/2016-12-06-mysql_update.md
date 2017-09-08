---
layout: post
title: MySQL版本升级
date: 2016-12-06 
tags: MySQL
---

### 小版本升级
从MySQL 5.6.1升级到MySQL5.6.x，升级未跨越5.6.x称为小版本升级。

### YUM升级

> 1. 不可少的备份； 
> 2. 查看YUM可升级的目标版本：
> ```bash 
    [root@ZTEST-206 ~]# yum list mysql-community-server
    Installed Packages
    mysql-community-server.x86_64 5.6.28-2.el6 @mysql56-community
    Available Packages
    mysql-community-server.x86_64 5.6.34-2.el6 mysql56-community
    ```

> 3. 升级： 
> ```bash 
    [root@ZTEST-206 ~]# yum update mysql-community-server
    ```

> 4. 执行mysql_upgrade检查：
> ```bash
    [root@ZTEST-206 ~]# mysql_upgrade -uroot -p123456
    ```

> 5. 查看升级后版本：  
> ```bash 
    mysql> select @@version;  
    +------------+  
    | @@version |  
    +------------+  
    | 5.6.34-log |   
    +------------+  
    1 row in set (0.00 sec)    
    ```

### tips:  
如果数据库是主从环境，遵循先升级从库，再升级主备原则；  
一般YUM更新后会重启MySQL。

### reference:  
http://dev.mysql.com/doc/refman/5.6/en/updating-yum-repo.html
