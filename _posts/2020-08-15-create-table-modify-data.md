---
title: create table and modify data
author: douqq
date:  2020-08-15 12:00:00 +0800
categories: [Database, SQL]
tags: [sql]
---



概念部分介绍完毕，终于要进入实操环节。在此我选择了入门门槛最低的mysql作为接下来的实验环境，需要知道，实际上不同的数据库在SQL的各个细节上会有不同，最好的办法就是直接查看相应数据库的语法手册，再结合自己动手测试。



# 测试环境：

mac，Server version: 10.4.13-MariaDB Homebrew

为了尽可能的抓住重点，在前期我们会绕开字符集。



# database

​	数据库是由数据库、用户、表、视图、宏和索引等对象构成的。可以使用SQL语句来创建这些对象、修改已存在对象的定义或者删除对象，完成这些工作的SQL语句称为DDL (数据定义语言)

​	在数据库内，一般会由几个概念，比如 database，owner，cluster，schema。这些概念在不同的数据库又有不同的所指，反倒是TABLE这种最底层的对象，大家都表示的是同一种意思。所以我们可以由table出发，来定义（或者描述）一些相关概念。

​	可以设想一下，数据库内应该会存在着不同数量的表，这些表所代表的的entity可能来自不同的部门或者业务，那么就会产生一种想法，就是把相同范畴的table组织在一起，让他们有一些共同的所属。这种数据库对象（一般是指表）的集合，一般来说叫 database，在oracle叫schema，在某些数据库根据从属关系叫owner。具体到我们本次的实验环境，在mysql里面，叫database。

​	可以通过下述语句，查看当前数据库的databases；

```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.000 sec)
```



## 切换db（database的简称）

```
MariaDB [(none)]> **use test**;
Database changed
```

一般来说，在数据库内，每个对象都有唯一的id，不过我们一般不会直接使用这个id，我们通过   db_name.tb_name(或者说obj_name)来唯一引用一个表（对象）。因此当然在同一个database内，不能有同名的对象。  

但是每次都在table前加db_name真的太麻烦了，所以一般在做具体开发过程中，我们会先use database，等于切换到一个独立的scope内，这样database就可以省略了。

例-可以在其他db下使用全局唯一名创建对象：

```
MariaDB [test]> create table **test1**.tb_1(id int);
Query OK, 0 rows affected (0.011 sec)
```



## 字符集

默认test库的字符集是latin的，如果想使用test库做测试，可以修改数据库字符集

`alter database test default character set utf8;`

当然也可以创建一个新的database with utf8

`create database test2 default character set=utf8;`



# create table





## syntax

我摘录mysql8.0 reference manual关于create table的部分章节（因为全部参数太多了）

```sql
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name

(create_definition,...)

[table_options]

[partition_options]

[AS] query_expression

--

create_definition: {

col_name column_definition

--

column_definition: {

data_type [NOT NULL | NULL] [DEFAULT {literal | (expr)} ]

[AUTO_INCREMENT] [UNIQUE [KEY]] [[PRIMARY] KEY]

[COMMENT 'string']
```



## 看懂语法图

一般现在用于描述sql或者命令行的 都是这么一套语法规则，主要有以下几个字符

`|`   竖线    分隔括号或大括号内的语法项目。只能选择一个项目。   

`[]`  方括号  可选语法项目。不必键入方括号。   

`{}`  大括号  必选语法项。不要键入大括号。 



对应于上述语法，我们可以看到，create table是关键字，中间的TEMPORARY 可以省略（是临时表的意思），看到create_definition的时候，没有进一步展开，因此在下文有其进一步描述，col_name column_definition，而column_definition 又有其进一步描述。因此，如果从这个语法图中我们抽取一种写法，将会是这个样子：

```sql
CREATE  TABLE  tbl_name
(col_name data_type)
```

这就是最简单的建表语句写法，也即仅包含了 表名，字段名，字段类型 这3个必要因素。

### 如下例：

```sql
CREATE TABLE Product
(product_id CHAR(4) NOT NULL,
product_name VARCHAR(100) NOT NULL,
product_type VARCHAR(32) NOT NULL,
sale_price INTEGER ,
purchase_price INTEGER ,
regist_date DATE ,
PRIMARY KEY (product_id));
```

这里PRIMARY KEY 的含义是在其指定字段上增加一个唯一非空索引（可以回顾范式理论）

数据类型的右侧设置了NOT NULL 约束。NULL 是代表空白（无记录）的关键字A。在NULL 之前加上了表示否定的NOT，就是给该列设置了不能输入空白，也就是必须输入数据的约束（如果什么都不输入就会出错）。

## 命名规则

我们只能使用**半角英文字母、数字、下划线（_）**作为数据库、表和列的名称 。例如，不能将product_id 写成product-id，因为标准SQL 并不允许使用连字符作为列名等名称。$、#、? 这样的符号同样不能作为名称使用。

> 注：这些特殊符号可能会被数据库系统使用

名称必须以半角英文字母开头

在同一个数据库中不能创建两个相同名称的表，在同一个表中也不能创建两个名称相同的列

# describe table

查看表结构。我们知道，创建了数据库对象之后，其实相关信息是放在数据字典内的。

如果想要查看表结构，可以使用desc语句进行查询，如下例：

```
MariaDB [test]> desc Product;
+---------------------+--------------+------+-----+---------+-------+
| Field               | Type         | Null | Key | Default | Extra |
+---------------------+--------------+------+-----+---------+-------+
| product_id          | char(4)      | NO   | PRI | NULL    |       |
| product_name        | varchar(100) | NO   |     | NULL    |       |
| product_type        | varchar(32)  | NO   |     | NULL    |       |
| sale_price          | int(11)      | YES  |     | NULL    |       |
| purchase_price      | int(11)      | YES  |     | NULL    |       |
| regist_date         | date         | YES  |     | NULL    |       |
+---------------------+--------------+------+-----+---------+-------+

```





# Alter table

有时好不容易把表创建出来之后才发现少了几列，其实这时无需把表删除再重新创建，只需使用变更表定义的ALTER TABLE 语句就可以了。

## 部分语法

```sql
ALTER TABLE tbl_name
[alter_option [, alter_option] ...]
[partition_options]

alter_option: {
table_options
| ADD [COLUMN] col_name column_definition
[FIRST | AFTER col_name]
```



## 增加字段

ALTER TABLE Product ADD COLUMN product_name_pinyin VARCHAR(100) after product_name;

在数据库内，字段的顺序也是很重要的，因此指定了 after语句之后，就在其后面增加了一个字段，如：

```sql
MariaDB [test]> desc Product;
+---------------------+--------------+------+-----+---------+-------+
| Field               | Type         | Null | Key | Default | Extra |
+---------------------+--------------+------+-----+---------+-------+
| product_id          | char(4)      | NO   | PRI | NULL    |       |
| product_name        | varchar(100) | NO   |     | NULL    |       |
| product_name_pinyin | varchar(100) | YES  |     | NULL    |       |
| product_type        | varchar(32)  | NO   |     | NULL    |       |
| sale_price          | int(11)      | YES  |     | NULL    |       |
| purchase_price      | int(11)      | YES  |     | NULL    |       |
| regist_date         | date         | YES  |     | NULL    |       |
+---------------------+--------------+------+-----+---------+-------+
7 rows in set (0.002 sec)
```

## 删除字段

```sql


MariaDB [test]> ALTER TABLE Product DROP COLUMN product_name_pinyin;
Query OK, 0 rows affected (0.013 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [test]> desc Product;
+----------------+--------------+------+-----+---------+-------+
| Field          | Type         | Null | Key | Default | Extra |
+----------------+--------------+------+-----+---------+-------+
| product_id     | char(4)      | NO   | PRI | NULL    |       |
| product_name   | varchar(100) | NO   |     | NULL    |       |
| product_type   | varchar(32)  | NO   |     | NULL    |       |
| sale_price     | int(11)      | YES  |     | NULL    |       |
| purchase_price | int(11)      | YES  |     | NULL    |       |
| regist_date    | date         | YES  |     | NULL    |       |
+----------------+--------------+------+-----+---------+-------+
6 rows in set (0.002 sec)
```

表定义变更之后无法恢复（包括被drop column内的数据）。

在执行ALTER TABLE语句之前请务必仔细确认。

# drop table 



如果想要删除Product 表

`DROP TABLE Product;`

删除的表是无法恢复的B。即使是被误删的表，也无法恢复，只能重新创建，然后重新插入数据。



# insert

为了方便之后的测试，我们需要了解一下如何插入数据

## 语法

一般来说，插入数据有2种语法，一种是基于 常量（data literal）的插入，一种是基于其他表的数据插入。

```sql
INSERT [INTO] tbl_name
[(col_name [, col_name] ...)]
{ {VALUES | VALUE} (value_list) [, (value_list)] ...

INSERT [INTO] tbl_name
[(col_name [, col_name] ...)]
{SELECT ... | TABLE table_name}
```

当values的列表顺序和column的顺序一致是，可以省略 column list

当只想往部分字段里面插入数据的时候，则不能省略

## auto commit

之前在数据库概念的章节，我们讲到过事务。在mysql里面，可以使用

start transaction; commit;来开启和关闭事务。但是现在数据库一般都设置开启了自动提交，因此我们此处只关注insert 即可。

## 样例

```sql
MariaDB [test]> INSERT INTO Product VALUES ('0001', 'T恤', '衣服',100, 50, '2009-09-21');
Query OK, 1 row affected (0.001 sec)

MariaDB [test]> select * from Product;
+------------+--------------+--------------+------------+----------------+-------------+

| product_id | product_name | product_type | sale_price | purchase_price | regist_date |
| ---------- | ------------ | ------------ | ---------- | -------------- | ----------- |
|            |              |              |            |                |             |

+------------+--------------+--------------+------------+----------------+-------------+

| 0001 | T恤  | 衣服 | 100  | 50   | 2009-09-21 |
| ---- | ---- | ---- | ---- | ---- | ---------- |
|      |      |      |      |      |            |

+------------+--------------+--------------+------------+----------------+-------------+
1 row in set (0.000 sec)


```

# update

使用INSERT 语句向表中插入数据之后，有时却想要再更改数据，这时并不需要把数据删除之后再重新插入，使用UPDATE 语句就可以改变表中的数据了。

## 基本语法

```sql
UPDATE [LOW_PRIORITY] [IGNORE] table_reference
SET assignment_list
[WHERE where_condition]
```

## 案例

将更新对象的列和更新后的值都记述在SET 子句中。

```
MariaDB [test]> select * from product;
+------------+--------------+--------------+------------+----------------+-------------+
| product_id | product_name | product_type | sale_price | purchase_price | regist_date |
+------------+--------------+--------------+------------+----------------+-------------+
| 0001       | T恤          | 衣服         |        100 |             50 | 2009-09-21  |
| 0003       | 运动T恤      | 衣服         |       4000 |           2800 | NULL        |
+------------+--------------+--------------+------------+----------------+-------------+
2 rows in set (0.000 sec)

MariaDB [test]> update product t set t.sale_price=500;
Query OK, 2 rows affected (0.001 sec)
Rows matched: 2  Changed: 2  Warnings: 0

MariaDB [test]> select * from product;
+------------+--------------+--------------+------------+----------------+-------------+
| product_id | product_name | product_type | sale_price | purchase_price | regist_date |
+------------+--------------+--------------+------------+----------------+-------------+
| 0001       | T恤          | 衣服         |        500 |             50 | 2009-09-21  |
| 0003       | 运动T恤      | 衣服         |        500 |           2800 | NULL        |
+------------+--------------+--------------+------------+----------------+-------------+
2 rows in set (0.000 sec)
```

通常情形下，不会对全表进行更新的，一般都是选择某些符合条件的记录进行更新，此时需要使用where条件。

> 注：大多数新人常犯的错误是不写where条件或者写了不严格的where子句导致全表数据被污染

如下例：

```
MariaDB [test]> update product t set t.sale_price=5000 where product_id='0001';
Query OK, 1 row affected (0.001 sec)
Rows matched: 1  Changed: 1  Warnings: 0

MariaDB [test]> select * from product;
+------------+--------------+--------------+------------+----------------+-------------+
| product_id | product_name | product_type | sale_price | purchase_price | regist_date |
+------------+--------------+--------------+------------+----------------+-------------+
| 0001       | T恤          | 衣服         |       5000 |             50 | 2009-09-21  |
| 0003       | 运动T恤      | 衣服         |        500 |           2800 | NULL        |
+------------+--------------+--------------+------------+----------------+-------------+
2 rows in set (0.000 sec)
```



使用UPDATE 也可以将列更新为NULL（该更新俗称为NULL 清空）。此时只需要将赋值表达式右边的值直接写为NULL 即可。

```
MariaDB [test]> update product t set t.regist_date=null where product_id='0001';
Query OK, 1 row affected (0.001 sec)
Rows matched: 1  Changed: 1  Warnings: 0

MariaDB [test]> select * from product;
+------------+--------------+--------------+------------+----------------+-------------+
| product_id | product_name | product_type | sale_price | purchase_price | regist_date |
+------------+--------------+--------------+------------+----------------+-------------+
| 0001       | T恤          | 衣服         |       5000 |             50 | NULL        |
| 0003       | 运动T恤      | 衣服         |        500 |           2800 | NULL        |
+------------+--------------+--------------+------------+----------------+-------------+
2 rows in set (0.000 sec)
```



## 多列更新

UPDATE 语句的SET 子句支持同时将多个列作为更新对象。

```
MariaDB [test]> update product t set t.regist_date=null,purchase_price=60 where product_id='0001';
Query OK, 1 row affected (0.001 sec)
Rows matched: 1  Changed: 1  Warnings: 0

MariaDB [test]> select * from product;
+------------+--------------+--------------+------------+----------------+-------------+
| product_id | product_name | product_type | sale_price | purchase_price | regist_date |
+------------+--------------+--------------+------------+----------------+-------------+
| 0001       | T恤          | 衣服         |       5000 |             60 | NULL        |
| 0003       | 运动T恤      | 衣服         |        500 |           2800 | NULL        |
+------------+--------------+--------------+------------+----------------+-------------+
```





# delete

不同于drop table，DELETE 语句会留下表（容器），而删除表中的全部或部分数据

## 基本语法

```sql
DELETE   FROM tbl_name [[AS] tbl_alias]
[WHERE where_condition]
```

如果不加where 语句，则是删除表中所有数据。

## 案例

```
MariaDB [test1]> select * from tb_1;
+------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 rows in set (0.000 sec)

MariaDB [test1]> delete from tb_1 where id=1;
Query OK, 1 row affected (0.001 sec)

MariaDB [test1]> select * from tb_1;
+------+
| id   |
+------+
|    2 |
|    3 |
+------+
2 rows in set (0.000 sec)

MariaDB [test1]> delete from tb_1 ;
Query OK, 2 rows affected (0.001 sec)

MariaDB [test1]> select * from tb_1;
Empty set (0.000 sec)
```





# CTAS

​	我们在介绍create table的语法的时候，看到过后面有个as 子句，这就是 create table as select 的分支，这个分支又常被缩写为 CTAS

​	在数据库实践中，有大量的场景需要临时建个表，用完就删的那种，此时可以不用先写好表结构，create table，然后再insert，直接使用 CTAS即可。

​	

​	如下例：

```
MariaDB [test]> select * from Product;
+------------+--------------+--------------+------------+----------------+-------------+

| product_id | product_name | product_type | sale_price | purchase_price | regist_date |
| ---------- | ------------ | ------------ | ---------- | -------------- | ----------- |
|            |              |              |            |                |             |

+------------+--------------+--------------+------------+----------------+-------------+

| 0001 | T恤  | 衣服 | 100  | 50   | 2009-09-21 |
| ---- | ---- | ---- | ---- | ---- | ---------- |
|      |      |      |      |      |            |

+------------+--------------+--------------+------------+----------------+-------------+
1 row in set (0.001 sec)

MariaDB [test]> create table Product_1 as select * from Product;
Query OK, 1 row affected (0.015 sec)
Records: 1  Duplicates: 0  Warnings: 0

MariaDB [test]> select * from Product_1;
+------------+--------------+--------------+------------+----------------+-------------+
| product_id | product_name | product_type | sale_price | purchase_price | regist_date |
+------------+--------------+--------------+------------+----------------+-------------+
| 0001       | T恤          | 衣服         |        100 |             50 | 2009-09-21  |
+------------+--------------+--------------+------------+----------------+-------------+
1 row in set (0.001 sec)
```

CTAS 会自动推断你select部分的 字段的数据类型，并且按照select的字段顺序建立表，并且把select 语句应该返回的结果插入到你建立的表内。

## 仅创建表结构

​	在有些场景下，你只需要依据子查询创建一个表结构，而不需要其数据。聪明如你，我想应该可以想到如下办法：

​	`create table xxx as select * from xxxx where 1=2`

​	其原理不过是CTAS的一个特例，即人为的导致select 结果为空而已。

如下例：

```
MariaDB [test]> create table Product_2 as select * from Product where 1=2;
Query OK, 0 rows affected (0.011 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [test]> select * from Product_2;
Empty set (0.000 sec)

MariaDB [test]> desc Product_2;
+----------------+--------------+------+-----+---------+-------+
| Field          | Type         | Null | Key | Default | Extra |
+----------------+--------------+------+-----+---------+-------+
| product_id     | char(4)      | NO   |     | NULL    |       |
| product_name   | varchar(100) | NO   |     | NULL    |       |
| product_type   | varchar(32)  | NO   |     | NULL    |       |
| sale_price     | int(11)      | YES  |     | NULL    |       |
| purchase_price | int(11)      | YES  |     | NULL    |       |
| regist_date    | date         | YES  |     | NULL    |       |
+----------------+--------------+------+-----+---------+-------+
6 rows in set (0.002 sec)
```



在不少数据库其实还提供了另外一种仅copy空表的语法，即 create table like

以mysql为例：

```
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name

{ LIKE old_tbl_name | (LIKE old_tbl_name) }
```

Use CREATE TABLE ... LIKE to create an **empty** table based on the definition of another table,including any column attributes and indexes defined in the original table:

一般情况下，create table like 在copy空表方面会更强大一些，比如会copy存储方面的一些参数，以及index，constraint。不过按照我的建议，如果你想在超出 字段，数据类型等方面进行自定义的话，最好还是自己手动去写各种参数，以确保不会被简单的like导致各种疏忽。

如下例：

```
MariaDB [test]> create table Product_3 like Product;
Query OK, 0 rows affected (0.013 sec)

MariaDB [test]> desc Product_3;
+----------------+--------------+------+-----+---------+-------+
| Field          | Type         | Null | Key | Default | Extra |
+----------------+--------------+------+-----+---------+-------+
| product_id     | char(4)      | NO   | PRI | NULL    |       |
| product_name   | varchar(100) | NO   |     | NULL    |       |
| product_type   | varchar(32)  | NO   |     | NULL    |       |
| sale_price     | int(11)      | YES  |     | NULL    |       |
| purchase_price | int(11)      | YES  |     | NULL    |       |
| regist_date    | date         | YES  |     | NULL    |       |
+----------------+--------------+------+-----+---------+-------+
6 rows in set (0.002 sec)

MariaDB [test]> select * from Product_3;
Empty set (0.000 sec)
```



# 视图(view)

​	视图究竟是什么呢？如果用一句话概述的话，就是“从SQL 的角度来看视图就是一张表”。实际上，在SQL 语句中并不需要区分哪些是表，哪些是视图，只需要知道在更新时它们之间存在一些不同就可以了。

​	那么视图和表到底有什么不同呢？区别只有一个，那就是“是否保存了实际的数据”。

​	通常，我们在创建表时，会通过INSERT 语句将数据保存到数据库之中，而数据库中的数据实际上会被保存到计算机的存储设备（通常是硬盘）中。因此，我们通过SELECT 语句查询数据时，实际上就是从存储设备（硬盘）中读取数据，进行各种计算之后，再将结果返回给用户这样一个过程。

​	但是使用视图时并不会将数据保存到存储设备之中，而且也不会将数据保存到其他任何地方。实际上视图保存的是SELECT 语句（图5-1）。我们从视图中读取数据时，视图会在内部执行该SELECT 语句并创建出一张临时表。

​	根据我的理解，我认为视图最大的用途就是 存储业务逻辑。往往被重复查询的sql会固化为视图，从而实现一种封装和抽象，这样别人可以直接使用这个视图而无非每次都重复实现此业务逻辑。



## 语法

```sql
CREATE
[OR REPLACE]
VIEW view_name [(column_list)]
AS select_statement
```

语法非常简单，我们看一下例子：

```sql
MariaDB [test]> create view v_product as
    -> select product_name,product_type
    -> from product;
Query OK, 0 rows affected (0.010 sec)

MariaDB [test]> desc v_product;
+--------------+--------------+------+-----+---------+-------+
| Field        | Type         | Null | Key | Default | Extra |
+--------------+--------------+------+-----+---------+-------+
| product_name | varchar(100) | NO   |     | NULL    |       |
| product_type | varchar(32)  | NO   |     | NULL    |       |
+--------------+--------------+------+-----+---------+-------+
2 rows in set (0.003 sec)

MariaDB [test]> select * from v_product;
+--------------+--------------+
| product_name | product_type |
+--------------+--------------+
| T恤          | 衣服         |
| 运动T恤      | 衣服         |
+--------------+--------------+
2 rows in set (0.000 sec)
```

我们可以通过创建v_product视图，来使其他用户看不到我们的内部数据product_id字段。

