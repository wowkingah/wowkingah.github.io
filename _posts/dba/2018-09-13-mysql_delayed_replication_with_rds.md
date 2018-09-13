---
layout: post
title: 从RDS搭建本地MySQL延时复制从库
date: 2018-09-13
tags: RDS MySQL Delayed Replication
---

### 简述  
将阿里云`RDS`备份恢复到本地`MySQL`，并建立延时主从同步，比`RDS`同步慢 6 小时。  
复制架构：阿里云`RDS`主 => 本地`MySQL`从。  

### 环境  
- 阿里云  
    + `RDS`：MySQL 5.6  
    + 创建主从同步用户：`GRANT REPLICATION SLAVE ON *.* TO 'repl' IDENTIFIED BY '123456';`  
- 本地  
    + `MySQL`：MySQL 5.6  
    + 安装`Percona XtraBackup`  

### 下载`RDS`备份  
```shell
[root@localhost ~]# wget --no-check-certificate -c 'https://rdsbak-shanghai-v2.oss-cn-shanghai.aliyuncs.com/xxxx' -O /srv/rds_20180913.tar.gz
```
*Tips*  
> -c：启用断点续传模式。  
> -O：将下载的结果保存为指定的文件.   

### 恢复`RDS`备份  
#### 解压  
```shell
# 下载 RDS 备份文件解压工具rds_backup_extract.sh
[root@localhost ~]# wget http://oss.aliyuncs.com/aliyunecs/rds_backup_extract.sh?spm=a2c4g.11186623.2.6.VBSjRR&file=rds_backup_extract.sh
[root@localhost ~]# sh rds_backup_extract.sh -f /srv/rds_20180913.tar.gz -C /data/mysql/
```
*Tips*  
> -f：指定要解压的备份集文件。  
> -C：指定文件要解压到的目录。可选参数，若不指定就解压到当前目录。  

#### 恢复  
```shell
[root@localhost ~]# innobackupex --defaults-file=/data/mysql/backup-my.cnf --apply-log /data/mysql/
```

#### 修改`my.cnf`  
如果使用本地 `my.cnf` ，添加 `/data/mysql/backup-my.cnf` 中以下参数。  
```shell
# RDS
[mysqld]
innodb_checksum_algorithm=innodb
#innodb_log_checksum_algorithm=innodb
innodb_data_file_path=ibdata1:200M:autoextend
innodb_log_files_in_group=2
innodb_log_file_size=1572864000
#innodb_fast_checksum=false
innodb_page_size=16384
#innodb_log_block_size=512
innodb_undo_directory=.
innodb_undo_tablespaces=0
```
*tips*  
注释的参数不能添加，是`RDS`专用参数，不兼容`MySQL 5.6`  

#### 修改文件属主  
```shell
[root@localhost ~]# chown -R mysql:mysql /data/mysql/
```

#### 启动本地`mysql`  
```shell
[root@localhost ~]# service mysqld start
```

### 配置主从  
```shell
# 没密码，可直接登录
[root@localhost ~]# mysql
mysql> reset slave all;
ERROR 1794 (HY000): Slave is not configured or failed to initialize properly. You must at least set --server-id to enable either a master or a slave. Additional error messages can be found in the MySQL error log.
# 因 RDS 原故，需清理以下表
mysql> truncate table mysql.slave_relay_log_info;
mysql> truncate table mysql.slave_master_info;
mysql> truncate table mysql.slave_worker_info;
mysql> delete from mysql.db where user<>'root' and char_length(user)>0;
mysql> delete from mysql.tables_priv where user<>'root' and char_length(user)>0;
mysql> use mysql;
# 导入本地 MySQL 系统表，不然 Slave 会报错
mysql> source /usr/share/mysql/mysql_system_tables.sql;
mysql> flush privileges;
```
需要重启`MySQL`
```shell
[root@localhost ~]# service mysqld restart
[root@localhost ~]# mysql
mysql> reset slave all;
Query OK, 0 rows affected (0.01 sec)

# 从本地恢复目录，找到 gtid_purged 信息
mysql> SET GLOBAL gtid_purged='96e7dfa9-4765-11e7-a9c1-008cfaf59458:1-3290114587, b41624a4-4765-11e7-a9c2-7cd30ac4f848:1-3258227';
ERROR 1840 (HY000): @@GLOBAL.GTID_PURGED can only be set when @@GLOBAL.GTID_EXECUTED is empty.
# 需清理 master 日志
mysql> reset master;
Query OK, 0 rows affected (0.01 sec)
mysql> SET GLOBAL gtid_purged='96e7dfa9-4765-11e7-a9c1-008cfaf59458:1-3290114587, b41624a4-4765-11e7-a9c2-7cd30ac4f848:1-3258227';
Query OK, 0 rows affected (0.00 sec)

mysql> CHANGE MASTER TO 
MASTER_HOST='rm-xxx.mysql.rds.aliyuncs.com',
MASTER_PORT =3306,
MASTER_USER='repl',
MASTER_PASSWORD='123456',
MASTER_DELAY=21600,
MASTER_AUTO_POSITION=1;

mysql> start slave;
mysql> show slave status \G
*************************** 1. row ***************************
               Slave_IO_State: Queueing master event to the relay log
                  Master_Host: rm-xxx.mysql.rds.aliyuncs.com
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.193401
          Read_Master_Log_Pos: 276286855
               Relay_Log_File: relay-bin.000002
                Relay_Log_Pos: 408
        Relay_Master_Log_File: mysql-bin.193401
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 271129684
              Relay_Log_Space: 5157777
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1061153568
                  Master_UUID: xxx-4765-xx-a9c1-xxx
             Master_Info_File: /data/mysql/master.info
                    SQL_Delay: 21600
          SQL_Remaining_Delay: 5265
      Slave_SQL_Running_State: Waiting until MASTER_DELAY seconds after master executed event
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set: 96e7dfa9-4765-11e7-a9c1-008cfaf59458:3292790197-3292793640
            Executed_Gtid_Set: 96e7dfa9-4765-11e7-a9c1-008cfaf59458:1-3292790196,
b41624a4-4765-11e7-a9c2-7cd30ac4f848:1-3258227
                Auto_Position: 1
1 row in set (0.00 sec)
```

