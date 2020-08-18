---
title: create table and insert data
author: douqq
date:  2020-08-15 12:00:00 +0800
categories: [Database, SQL]
tags: []
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

-  |   竖线     分隔括号或大括号内的语法项目。只能选择一个项目。   
-  []  方括号  可选语法项目。不必键入方括号。   
-  {}  大括号  必选语法项。不要键入大括号。 



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





