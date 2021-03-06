---
layout: post
title: "数据库大小写敏感问题"
date: 2017/1/23 14:08:05 
tags: Databases MySQL Oracle MSSQL
---

### Oracle  
默认大小写不敏感，表名、字段名等不区分大小写。小写字母会自动转换为大写字母，需要用小写字母时需要使用双引号，或借助函数upper()和lower()。

### SQLServer  
默认大小写不敏感。可以通过修改排序规则改变：右击数据库 -> 属性 -> 选项 -> 排序规则：Chinese_PRC_CS_AS（区分大小写）、Chinese_PRC_CI_AS（不区分大小写）。

### MySQL  
#### Windows  
都不区分大小写。

#### Linux  
数据库名、表名、列名、别名大小写规则如下：  
> 1. 数据库名与表名是严格区分大小写；  
> 2. 表的别名是严格区分大小写；  
> 3. 变量名也是严格区分大小写；
> 4. 列名与列的别名在所有的情况下均忽略大小写。  

##### 解决办法：
**关于库名、表名、表别名的大小写敏感问题**  
可通过lower_case_table_names参数来控制数据库名、表名、表别名的大小写敏感。
```bash
mysql> show variables like 'lower%';
+------------------------+-------+
| Variable_name  | Value |
+------------------------+-------+
| lower_case_file_system | OFF   |
| lower_case_table_names | 1 |
+------------------------+-------+
2 rows in set (0.00 sec)
```
tips:  
lower_case_file_system是只读参数，无法被修改，这个参数是用来告诉你在当前的系统平台下，是否对文件名大小写敏感。

notes:  
> 1. 对于windows和Mac OX S 这些文件系统对大小写不敏感的系统该参数不能设置为0，并且在5.6.27以后如果在大小写不敏感的操作系统上设置该参数为0启动实例时会报错，并且退出。  
> 2. linux系统值为0，windows默认设置为1，Mac OXS设置为2， lower_case_table_names=2 只能用于大小写不敏感的系统，在linux系统上设置lower_case_table_names=2，启动后errorlog中会有如下提示：  
 [Warning] lower_case_table_names was set to 2,even though your the file system '/home/test/var/lib/data1/' is casesensitive.  Now settinglower_case_table_names to 0 to avoid future problems.  
> 3. 为了避免不同平台之间迁移数据库时遇到大小写问题，建议所有平台上都设置lower_case_table_names=1。  
> 4. 在unix系统上，如果你准备把lower_case_table_names从0改成1，那么你的先把有大写的表名都改成小写(RENAME TABLE T1 TO t1;)，否则你访问之前有大写字母的表会报如下错误：desc sysNumLimit_0408; ERROR 1146 (42S02): Table 'kopc.sysnumlimit_0408'doesn't exist.  
当然也可以在修改参数之前把库做导出，然后删除所有数据库(drop database)，然后修改参数，最后重新导入。  
  
**关于字段值内容的大小写敏感问题**  
字段值的大小写敏感性由MySQL的校对规则(collate)来控制。提到校对规则，就不得不说字符集。字符集是一套符号和编码，collate指定了字符集中数据的排序和比对规则则，比如定义'A'<'B'这样的关系的规则。每种字符集都会对应一定数据库的collate，一般而言，collate是以其相关的字符集名开始，通常包括一个语言名，并且以_ci（ci是case insensitive的缩写）、_cs（cs是case sensitive的缩写）或_bin（二元）结束 。比如 latin1字符集的collate有latin1_general_ci(这个是utf8字符集默认的校对规则)，latin1_general_cs，latin1_bin表示二进制比较(区分大小写)。可以通过show collation命令来查看字符集支持的校对规则有哪些。  

校对规则通过关键字collate指定，可以在数据库、表、列几个层面指定字符集和校对规，越往后优字符集和校对规则生效的优先级越高，如果没有为列或者表指定字符集和校对规则，那么它会继承上级的字符集和校对规则。比如创建数据库test1，指定字符集为utf8，校对规则为utf8_bin。  

**相关测试如下：**  
```bash
#1.创建字符集为utf8，校对规则为utf8_general_ci的数据库（MySQL5.6默认）
mysql> CREATE DATABASE test1;
Query OK, 1 row affected (0.00 sec)

mysql> use test1;
Database changed

#2.1创建test_case表。因为没有为表和列指定字符集、校对规则，它们会继承所在数据库的字符集和校对规则
mysql> CREATE TABLE test_case(name varchar(20));
Query OK, 0 rows affected (0.77 sec)

#2.2为列指定了字符集和校对规则(列优先级高于表，高于数据库)
mysql> CREATE TABLE test_case1(name varchar(20) character set utf8 collate utf8_bin);
Query OK, 0 rows affected (0.80 sec)

mysql> SHOW CREATE TABLE test_case \G
*************************** 1. row ***************************
   Table: test_case
Create Table: CREATE TABLE `test_case` (
  `name` varchar(20) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)

mysql> SHOW CREATE TABLE test_case1 \G
*************************** 1. row ***************************
   Table: test_case1
Create Table: CREATE TABLE `test_case1` (
  `name` varchar(20) CHARACTER SET utf8 COLLATE utf8_bin DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)

#3.插入测试数据
mysql> INSERT INTO test_case VALUES('abc');
Query OK, 1 row affected (0.13 sec)

mysql> INSERT INTO test_case VALUES('ABC');
Query OK, 1 row affected (0.13 sec)

mysql> INSERT INTO test_case1 VALUES('abc');
Query OK, 1 row affected (0.27 sec)

mysql> INSERT INTO test_case1 VALUES('Abc');
Query OK, 1 row affected (0.11 sec)

mysql> SELECT * FROM test_case;
+------+
| name |
+------+
| abc  |
| ABC  |
+------+
2 rows in set (0.00 sec)

mysql> SELECT * FROM test_case1;
+------+
| name |
+------+
| abc  |
| Abc  |
+------+
2 rows in set (0.00 sec)

#4.查询测试
mysql> SELECT * FROM test_case WHERE name LIKE 'ab%';
+------+
| name |
+------+
| abc  |
| ABC  |
+------+
2 rows in set (0.00 sec)

#因为test_case使用的utf8_general_ci校对规则，所以大小写不敏感；test_case1的name列使用的是utf8_bin，所以大小写敏感。
mysql> SELECT * FROM test_case1 WHERE name LIKE 'ab%';
+------+
| name |
+------+
| abc  |
+------+
1 row in set (0.00 sec)

#5.添加主键测试
mysql> ALTER TABLE test_case ADD PRIMARY KEY (name);
ERROR 1062 (23000): Duplicate entry 'ABC' for key 'PRIMARY'
mysql> ALTER TABLE test_case1 ADD PRIMARY KEY (name);
Query OK, 2 rows affected (2.06 sec)
Records: 2  Duplicates: 0  Warnings: 0
#test_case因为大小写不敏感，所以认为表中的abc和ABC两条记录是一样的，故添加主键时报错。
```

NOTE:
1. 通过上面的测试，可以看到如果想某个字段值区分大小写，可以在建表时为该字段指定区别大小写的collate(_ci或者_bin)，对于已存在的表可以通过如下语句修改列的collate： 
```SQL 
ALTER TABLE test_case MODIFY COLUMN NAME VARCHAR(20) CHARACTER SET UTF8 COLLATE UTF8_BIN;  
```

2. 如果不想修改字段的collate，可以在查询的时候指定collate或者通过binary进行字段转换：  
```sql
SELECT * FROM test_case WHERE name LIKE 'ab%' COLLATE UTF8_BIN;  
SELECT * FROM test_case WHERE BINARY name LIKE 'ab%';  
SELECT * FROM test_case WHERE name = BINARY 'abc';  
```

**测试验证**  
```bash
mysql> SELECT * FROM test_case WHERE name LIKE 'ab%';
+------+
| name |
+------+
| abc  |
| ABC  |
+------+
2 rows in set (0.00 sec)
#没有指定collate，默认使用name列的collate(utf8_general_ci)，所以查询不区分大小写

mysql> SELECT * FROM test_case WHERE name LIKE 'ab%' COLLATE UTF8_BIN;
+------+
| name |
+------+
| abc  |
+------+
1 row in set (0.00 sec)
#查询时指定collate为utf8_bin，所以查询区分大小写

mysql> SELECT * FROM test_case WHERE BINARY name LIKE 'ab%';
+------+
| name |
+------+
| abc  |
+------+
1 row in set (0.00 sec)
#通过使用binary关键字将字符串转换成二进制进行比较，这样也能够区分大小写。(实验发现对name列使用binary依然可以使用name列上的索引)

mysql> SELECT * FROM test_case1 WHERE name LIKE 'ab%';
+------+
| name |
+------+
| abc  |
+------+
1 row in set (0.00 sec)
#没有指定collate，查询默认使用name列的collate(utf8_bin)，所以查询区分大小写

mysql> SELECT * FROM test_case1 WHERE name LIKE 'ab%' COLLATE UTF8_GENERAL_CI;
+------+
| name |
+------+
| Abc  |
| abc  |
+------+
2 rows in set (0.00 sec)
#查询指定collate为utf8_general_ci，所以查询区分大小写

mysql> SELECT * FROM test_case WHERE name LIKE 'ab%';
+------+
| name |
+------+
| abc  |
| ABC  |
+------+
2 rows in set (0.00 sec)
#没有指定区别大小写的collate也没有指定binary，所以查询是不区分大小写的

mysql> SELECT * FROM test_case WHERE name LIKE BINARY 'ab%';
+------+
| name |
+------+
| abc  |
+------+
1 row in set (0.00 sec)
#使用binary 将'abc'字符串转换成二进制进行比较  
```
  
  
  

#### reference:  
http://www.voidcn.com/blog/shaochenshuo/article/p-6229106.html  
https://bbs.aliyun.com/read/243931.html
