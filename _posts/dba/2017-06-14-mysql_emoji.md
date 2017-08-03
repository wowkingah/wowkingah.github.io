---
layout: post
title: MySQL支持Emoji表情
date: 2017-06-14
tags: MySQL
---

### 简述  
MySQL 5.5.3开始通过修改字符集来支持Emoji表情。  

### 查看当前字符集   
```shell
mysql> SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';
+--------------------------+-----------------+
| Variable_name            | Value           |
+--------------------------+-----------------+
| character_set_client     | utf8            |
| character_set_connection | utf8            |
| character_set_database   | utf8            |
| character_set_filesystem | binary          |
| character_set_results    | utf8            |
| character_set_server     | utf8            |
| character_set_system     | utf8            |
| collation_connection     | utf8_general_ci |
| collation_database       | utf8_general_ci |
| collation_server         | utf8_general_ci |
+--------------------------+-----------------+
10 rows in set (0.01 sec)
```

### 相关配置
#### 修改字符集  
```shell
[root@test ~]# cat /etc/my.cnf
[client]
default_character_set=utf8mb4

[mysql]
default_character_set=utf8mb4

[mysqld]
##Character settings
init_connect='SET NAMES utf8mb4'
character_set_server=utf8mb4
collation_server=utf8mb4_general_ci
skip_character_set_client_handshake
```

#### 查看字符集   
```shell
mysql> SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';
+--------------------------+--------------------+
| Variable_name            | Value              |
+--------------------------+--------------------+
| character_set_client     | utf8mb4            |
| character_set_connection | utf8mb4            |
| character_set_database   | utf8mb4            |
| character_set_filesystem | binary             |
| character_set_results    | utf8mb4            |
| character_set_server     | utf8mb4            |
| character_set_system     | utf8               |
| collation_connection     | utf8mb4_general_ci |
| collation_database       | utf8mb4_general_ci |
| collation_server         | utf8mb4_general_ci |
+--------------------------+--------------------+
10 rows in set (0.00 sec)
```

**已有库表字段修改**  
对于已有实例、表、字段如果需支持Emoji表情，可按需修改字符集。如果只是某个字段需要，只需修改对应字段的字符集即可。  

**修改数据库字符集**  
```shell
ALTER DATABASE database_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
```

**修改表的字符集**  
```shell
ALTER TABLE table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

**修改字段的字符集**  
```shell
ALTER TABLE table_name CHANGE column_name column_name VARCHAR(191) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### 注意事项  
如果通过以上配置还不支持Emoji表情，请Connector/J版本或查看sql_mode参数。  

#### Connector  
connector版本大于5.1.13的不能加characterEncoding=utf8参数（未测试）。  

#### sql_mode  
```shell
mysql> show variables like 'sql_mode';
+---------------+----------------------------------------------------------------+
| Variable_name | Value                                                          |
+---------------+----------------------------------------------------------------+
| sql_mode      | STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION |
+---------------+----------------------------------------------------------------+
1 row in set (0.00 sec)
```
若返回结果中带有STRICT_TRANS_TABLES，去掉即可。  
**动态配置**  
```shell
mysql> set global sql_mode = 'NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
```

**永久配置**  
```shell
[root@test ~]# cat /etc/my.cnf
[mysqld]
sql_mode="NO_ENGINE_SUBSTITUTION,NO_AUTO_CREATE_USER"
```

