---
layout: post
title: MySQL中VARCHAR最大长度
date: 2017-11-22
tags: MySQL
---

### 简述  
MySQL中VARCHAR长度取值范围为0~65535，是不是意味着真的能存这么多字符串？  

### 测试  
1. t_varchar_1，超过最大行限制。失败：  
```sql
mysql> CREATE TABLE t_varchar_1 (
    ->   `id` int(10) unsigned,
    ->   `name` varchar(65535)
    -> ) ENGINE=InnoDB DEFAULT CHARSET=latin1;
ERROR 1118 (42000): Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. This includes storage overhead, check the manual. You have to change some columns to TEXT or BLOBs
```

2. t_varchar_2，varchar需要2字节存储长度。失败：  
```sql
mysql> CREATE TABLE t_varchar_2 (
    ->   `name` varchar(65535)
    -> ) ENGINE=InnoDB DEFAULT CHARSET=latin1;
ERROR 1118 (42000): Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. This includes storage overhead, check the manual. You have to change some columns to TEXT or BLOBs
```

###  官方文档  
#### The CHAR and VARCHAR Types  
`Values in VARCHAR columns are variable-length strings. The length can be specified as a value from 0 to 65,535. The effective maximum length of a VARCHAR is subject to the maximum row size (65,535 bytes, which is shared among all columns) and the character set used. See Section C.10.4, “Limits on Table Column Count and Row Size”.`  
大概意思是 `VARCHAR`长度取值为0~65535，实际有效最大长度取决于最大行大小（在一个表的所有列中共享65535字节）和使用的字符集。  

#### Limits on Table Column Count and Row Size  
`The internal representation of a MySQL table has a maximum row size limit of 65,535 bytes, even if the storage engine is capable of supporting larger rows. BLOB and TEXT columns only contribute 9 to 12 bytes toward the row size limit because their contents are stored separately from the rest of the row.`
大概意思是 MySQL内部限制表的最大行大小为65535字节，`BLOB`和`TEXT`列仅在限制字节中占9到12个字节。  

###  验证  
3. t_varchar_3，字符集为 latin1。成功：  
```sql
mysql> CREATE TABLE t_varchar_3 (
    ->   `name` varchar(65532)
    -> ) ENGINE=InnoDB DEFAULT CHARSET=latin1;
Query OK, 0 rows affected (0.19 sec)
```

4. t_varchar_4，指定utf8字符集，因utf占3个字节，理论上varchar最长为(65535-2)/3=21844，先试个21845字节。失败：  
```sql
mysql> CREATE TABLE t_varchar_4 (
    ->   `name` varchar(21845)
    -> ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
ERROR 1118 (42000): Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. This includes storage overhead, check the manual. You have to change some columns to TEXT or BLOBs
```

5. t_varchar_5，字符集为 InnoDB。成功：  
```sql
mysql> CREATE TABLE t_varchar_5 (
    ->   `name` varchar(21844)
    -> ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
Query OK, 0 rows affected (0.32 sec)
```

6. t_varchar_6，一个表的所有列中共享65535字节。失败：  
```sql
mysql> CREATE TABLE t_varchar_6 (
    ->   `id` int(11),
    ->   `name` varchar(21844)
    -> ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
ERROR 1118 (42000): Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. This includes storage overhead, check the manual. You have to change some columns to TEXT or BLOBs
```

7. t_varchar_7，成功：
```sql
mysql> CREATE TABLE t_varchar_7 (
    ->   `id` int(11),
    ->   `name` varchar(21842)
    -> ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
Query OK, 0 rows affected (0.23 sec)
```

###  结论  
可见，`VARCHAR`的实际长度取决于表的字符集，以及表中其它字段占用的长度有关。不过，`BLOB`和`TEXT`列仅在共享长度中占9到12个字节，这里没贴测试结果。  

### reference
https://dev.mysql.com/doc/refman/5.6/en/column-count-limit.html