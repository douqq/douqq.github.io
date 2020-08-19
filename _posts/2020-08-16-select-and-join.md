---
title: select and join
author: douqq
date:  2020-08-16 12:00:00 +0800
categories: [Database, SQL]
tags: []
---





​	从表中选取数据时需要使用SELECT 语句，也就是只从表中选出（SELECT）必要数据的意思。通过SELECT 语句查询并选取出必要数据的过程称为匹配查询或查询（query）。SELECT 语句是SQL 语句中使用最多的最基本的SQL 语句。掌握了SELECT 语句，距离掌握SQL 语句就不远了。



# 

以mysql的为例

# 基础select

```sql
SELECT
[DISTINCT]
select_expr [, select_expr] ...
[FROM table_references]
[WHERE where_condition]
[GROUP BY {col_name | expr | position},...]
[HAVING where_condition]
[ORDER BY {col_name | expr | position}
[LIMIT {[offset,] row_count | row_count OFFSET offset}]
```

子句（clause）是SQL 语句的组成要素，是以SELECT 或者FROM 等作为起始的短语。

## 查询指定列

SELECT 子句中列举了希望从表中查询出的列的名称，而FROM 子句则指定了选取出数据的表的名称。

接下来，我们尝试从Product（商品）表中，查询出product_id（商品编号）列、product_name（商品名称）列和purchase_price（进货单价）列。



```sql
MariaDB [test]> SELECT product_id, product_name, purchase_price
    -> FROM Product;
+------------+--------------+----------------+
| product_id | product_name | purchase_price |
+------------+--------------+----------------+
| 0001       | T恤          |             50 |
+------------+--------------+----------------+
1 row in set (0.000 sec)
```

SELECT 语句第一行的SELECT product_id, product_name,purchase_price 就是SELECT 子句。查询出的列的顺序可以任意指定。查询多列时，需要使用逗号进行分隔。 查询结果中列的顺序和SELECT 子句中顺序相同。

## 查询出表中所有的列

想要查询出全部列时，可以使用代表所有列的星号（*）。

如果使用星号的话，就无法设定列的显示顺序了 。这时就会按照CREATE TABLE 语句的定义对列进行排序。

```
MariaDB [test]> select * from Product;
+------------+--------------+--------------+------------+----------------+-------------+
| product_id | product_name | product_type | sale_price | purchase_price | regist_date |
+------------+--------------+--------------+------------+----------------+-------------+
| 0001       | T恤          | 衣服         |        100 |             50 | 2009-09-21  |
+------------+--------------+--------------+------------+----------------+-------------+
1 row in set (0.000 sec)
```

## 为列设定别名

SQL 语句可以使用AS 关键字为列设定别名

别名可以使用中文，使用中文时需要用双引号（"）括起来

```
MariaDB [test]> SELECT
    -> product_id as pid,
    -> product_name as pname,
    -> purchase_price as pprice
    -> FROM Product;
+------+-------+--------+
| pid  | pname | pprice |
+------+-------+--------+
| 0001 | T恤   |     50 |
+------+-------+--------+
1 row in set (0.000 sec)
```

## 为表设定别名

表名也可以设定别名，具体语法为 表或者子查询后面 跟一个自定义的字符串，不过我们一般推荐使用t1,t2这样（t代表table缩写，v可以代表视图缩写）

> 为表名设置别名有特别的好处：在之后学习了join，会知道可能关联的2张表会有相同的字段名，此时如果要区分它们，在不使用表别名的情况下，就必须写全 tablename.columnname，对于某些特别长的表名就会比较费劲。

样例如下：



```sql
MariaDB [test]> select distinct t.product_type from product t;
+--------------+
| product_type |
+--------------+
| 衣服         |
+--------------+
1 row in set (0.000 sec)
```



## 查询literal

SELECT 子句中不仅可以书写列名，还可以书写常数(literal),这些字面值也可以作为一列添加到返回结果内，并且可以有别名，如果更进一步你还可以显式定义它的数据类型（虽然大多数情况下数据库的推断结果都是靠谱的）。

如下例：today 列

```
MariaDB [test]> SELECT
    -> product_id as pid,
    -> product_name as pname,
    -> purchase_price as pprice,
    -> '2020-08-16' as today
    -> FROM Product;
+------+-------+--------+------------+
| pid  | pname | pprice | today      |
+------+-------+--------+------------+
| 0001 | T恤   |     50 | 2020-08-16 |
+------+-------+--------+------------+
1 row in set (0.000 sec)
```



如果你足够细心，你会发现from 子句(clause)也是可以忽略的（在mysql里面，在oracle里面就不行）

那么当你仅想查询某些常量，而不取任何一张表的数据的时候，你可以只写select子句，如下例：

```
MariaDB [test]> select 1 as a ,
    -> 2 as b,
    -> 'xx' as c ;
+---+---+----+
| a | b | c  |
+---+---+----+
| 1 | 2 | xx |
+---+---+----+
1 row in set (0.000 sec)
```

## Distinct

如果查询的结果内有重复的行，而我们只需要去掉重复的行，只保留不一样的行（术语叫做：去重），我们可以使用关键字 distinct 跟在select后面表示去重。

DISTINCT 关键字只能用在第一个列名之前，select之后（看语法图）

> 注：所谓重复是仅指返回的结果而言，比如2个货物，其product_id自然不同，但是其product_type有可能相同，当你仅select product_type字段时，就会可能有重复。  
>
> 注：当你学习了group by之后，你会学会另外一种去重的办法



```
我再插入一行数据

MariaDB [test]> INSERT INTO Product VALUES ('0003', '运动T恤', '衣服',4000, 2800, NULL);
Query OK, 1 row affected (0.001 sec)

MariaDB [test]> select product_type from product;
+--------------+
| product_type |
+--------------+
| 衣服         |
| 衣服         |
+--------------+
2 rows in set (0.000 sec)

MariaDB [test]> select distinct product_type from product;
+--------------+
| product_type |
+--------------+
| 衣服         |
+--------------+
1 row in set (0.000 sec)


```

## where

​	SELECT 语句通过WHERE 子句来指定查询数据的条件。在WHERE子句中可以指定“某一列的值和这个字符串相等”或者“某一列的值大于这个数字”等条件。执行含有这些条件的SELECT 语句，就可以查询出只符合该条件的记录了

```sql
MariaDB [test]> SELECT product_name, product_type
    -> FROM Product
    -> WHERE product_type = '衣服';
+--------------+--------------+
| product_name | product_type |
+--------------+--------------+
| T恤          | 衣服         |
| 运动T恤      | 衣服         |
+--------------+--------------+
2 rows in set (0.000 sec)
```

​	WHERE 子句中的“product_type = '衣服'”就是用来表示查询条件的表达式（条件表达式）。等号是比较两边的内容是否相等的符号，上述条件就是将product_type 列的值和'衣服' 进行比较，判断是否相等。Product 表的所有记录都会被进行比较。

​	SQL 中子句的书写顺序是固定的，不能随意更改。WHERE 子句必须紧跟在FROM 子句之后，书写顺序发生改变的话会造成执行错误

> ​	关于操作符，也即where子句中的各种运算，我们随后会专门介绍

​	where 子句的 条件表达式可以有**多个** 表达式 使用 and or 等逻辑操作符 组成一个完整的逻辑表达式

## order by

​	通常，从表中抽取数据时，如果没有特别指定顺序，最终排列顺序便无从得知。即使是同一条SELECT 语句，每次执行时排列顺序很可能发生改变。但是不进行排序，很可能出现结果混乱的情况。这时，便需要通过在SELECT 语句末尾添加ORDER BY 子句来明确指定排列顺序。

​	不论何种情况，ORDER BY 子句都需要写在SELECT 语句的末尾。这是因为对数据行进行排序的操作必须在结果即将返回时执行。

> ​	注：根据这条规则，你也可以推导出在子查询（subquery）内排序是没有必要的也不允许的

​	ORDER BY syntax permits one or more column names to be specified for sorting, each of which optionally can be followed by ASC or DESC to indicate ascending or descending sort order, respectively.

The default is **ascending** order.

​	`order by col1[asc|desc],coln...`

​	order by 子句后面跟字段的顺序，每个字段可以 设定升序（按照行号方向从小到大）或者降序（从大到小），默认是升序所以可以省略asc。

​	多字段的组合按照字段顺序优先级排序



```sql
SELECT product_name, product_type 
FROM Product WHERE product_type = '衣服'
order by product_name desc ,product_type ;
+--------------+--------------+
| product_name | product_type |
+--------------+--------------+
| 运动T恤      | 衣服         |
| T恤          | 衣服         |
+--------------+--------------+
2 rows in set (0.000 sec)
```



## limit

有时候，你需要控制返回的行数。大多数数据库都有此功能，在mysql里面，是使用limit对于最终返回客户端的结果行数进行限定。如下例，我们只返回1条记录。

```
MariaDB [test]> select product_name from Product
    -> limit 1;
+--------------+
| product_name |
+--------------+
| T恤          |
+--------------+
1 row in set (0.000 sec)
```



# join部分

```sql
table_reference: {
table_factor
| joined_table
}

table_factor: {
tbl_name
[[AS] alias]
| table_subquery [AS] alias [(col_list)]
| ( table_references )
}

joined_table: {
table_reference {[INNER | CROSS] JOIN } table_factor [join_specification]
| table_reference {LEFT|RIGHT} [OUTER] JOIN table_reference join_specification
| table_reference NATURAL [INNER | {LEFT|RIGHT} [OUTER]] JOIN table_factor
}

join_specification: {
ON search_condition
}

```







# union部分

```sql
SELECT ...
UNION [ALL | DISTINCT] SELECT ...
[UNION [ALL | DISTINCT] SELECT ...]
```



# 子查询（subquery）



我们先来回顾一下视图的概念，视图并不是用来保存数据的，而是通过保存读取数据的SELECT 语句的方法来为用户提供便利。相比之下，子查询就是将用来定义视图的SELECT语句直接用于FROM子句当中。用官方语言讲，就是

A subquery is a SELECT statement within another statement


```sql
MariaDB [test]> SELECT product_type, cnt_product
    -> FROM ( SELECT product_type, COUNT(*) AS cnt_product
    -> FROM Product
    -> GROUP BY product_type ) AS ProductSum;
+--------------+-------------+
| product_type | cnt_product |
+--------------+-------------+
| 衣服         |           2 |
+--------------+-------------+
1 row in set (0.000 sec)
```

如上所示，子查询就是将用来定义视图的SELECT 语句直接用于FROM 子句当中。虽然“AS ProductSum”就是**子查询的名称**，但由于该名称是一次性的，因此不会像视图那样保存在存储介质（硬盘）之中，而是在SELECT 语句执行之后就消失了。

实际上，该SELECT 语句包含嵌套的结构，**首先**会执行FROM 子句中的SELECT 语句，然后才会执行外层的SELECT 语句

## 嵌套示例

由于子查询的层数原则上没有限制，因此可以像“子查询的FROM 子句中还可以继续使用子查询，该子查询的FROM 子句中还可以再使用子查询……”这样无限嵌套下去

```
MariaDB [test]> select product_type from (
    -> SELECT product_type, cnt_product
    -> FROM ( SELECT product_type, COUNT(*) AS cnt_product
    -> FROM Product
    -> GROUP BY product_type ) AS ProductSum ) t;
+--------------+
| product_type |
+--------------+
| 衣服         |
+--------------+
1 row in set (0.000 sec)
```

因为存在着各种嵌套，所以其实对于子查询的命名（别名）也显得特别重要，请按照一定的层次命名以免分不清楚层级。

## 标量子查询

​	标量就是单一的意思，在数据库之外的领域也经常使用。子查询基本上都会返回多行结果（虽然偶尔也会只返回1 行数据）。而标量子查询则有一个特殊的限制，那就是必须而且只能返回1 行1列的结果，也就是返回表中某一行的某一列的值或者某种聚合

​	由于返回的是单一的值，因此标量子查询的返回值可以用在= 或者<> 这样需要单一值的比较运算符之中。这也正是标量子查询的优势所在

如下例：需要返回售价大于平均值的商品。

```
MariaDB [test]> select * from product where sale_price>(select avg(sale_price) from product);
+------------+--------------+--------------+------------+----------------+-------------+
| product_id | product_name | product_type | sale_price | purchase_price | regist_date |
+------------+--------------+--------------+------------+----------------+-------------+
| 0001       | T恤          | 衣服         |       5000 |             60 | NULL        |
+------------+--------------+--------------+------------+----------------+-------------+
1 row in set (0.003 sec)
```

> 注：这种业务逻辑在学会join之后也可以用其他方式来实现

### 标量子查询的书写位置

标量子查询的书写位置并不仅仅局限于WHERE 子句中，通常任何可以使用单一值的位置都可以使用。也就是说，能够使用常数或者列名的地方，无论是SELECT 子句、GROUP BY 子句、HAVING 子句，还是ORDER BY 子句，几乎所有的地方都可以使用。

例：在select 子句代入标量子查询

```
MariaDB [test]> select product_id,
    -> product_name,sale_price,
    -> (select avg(sale_price) from product) as avgprice
    -> from product;
+------------+--------------+------------+-----------+
| product_id | product_name | sale_price | avgprice  |
+------------+--------------+------------+-----------+
| 0001       | T恤          |       5000 | 2750.0000 |
| 0003       | 运动T恤      |        500 | 2750.0000 |
+------------+--------------+------------+-----------+
2 rows in set (0.000 sec)
```



## 关联子查询

​	大多数情况下，子查询的返回结果不是一条的，当主表（即非子查询的表）想和子查询进行某些关联（join）的时候，当然可以把子查询当做一张表一样进行关联，也可以像上文的子查询一样在where部分进行限定，所不同的是，要在子查询内部和主表进行关联，以达到使主表的每一条记录在比对时，只和一个确定的值进行比对，这就又回到了标量比较的范畴。

可以参考下例：（虽然这个sql没什么具体的业务含义，只是为了说明可以这样做）

```
MariaDB [test]> select
    ->  product_id,
    ->  product_name,
    ->  sale_price
    ->  from product p1
    ->  where sale_price=
    ->    (select sale_price from product p2
    ->     where p1.product_id=p2.product_id) ;
+------------+--------------+------------+
| product_id | product_name | sale_price |
+------------+--------------+------------+
| 0001       | T恤          |       5000 |
| 0003       | 运动T恤      |        500 |
+------------+--------------+------------+
2 rows in set (0.000 sec)
```

结合条件一定要写在子查询中

关联名称就是像P1、P2 这样作为表别名的名称，作用域（scope）就是生存范围（有效范围）。也就是说，关联名称存在一个有效范围的限制。具体来讲，子查询内部设定的关联名称，只能在该子查询内部使用。换句话说，就是“内部可以看到外部，而外部看不到内部”。



# 聚合函数

通过SQL 对数据进行某种操作或计算时需要使用函数。例如，计算表中全部数据的行数时，可以使用COUNT 函数。该函数就是使用COUNT（计数）来命名的。除此之外，SQL 中还有很多其他用于汇总的函数，请大家先记住以下5 个常用的函数。

COUNT： 计算表中的记录数（行数）

SUM： 计算表中数值列中数据的合计值

AVG： 计算表中数值列中数据的平均值

MAX： 求出表中任意列中数据的最大值

MIN： 求出表中任意列中数据的最小值

如上所示，用于汇总的函数称为聚合函数或者聚集函数，本书中统称为聚合函数。所谓聚合，就是将多行汇总为一行。实际上，所有的聚合函数都是这样，输入多行输出一行。

## count

使用COUNT 函数时，输入表的列，就能够输出数据行数。

COUNT (*) 中的星号，代表全部列的意思。当然如果仅统计行数，用count(1)也是可以的

如下例：

```
MariaDB [test]> select * from tb_1;
+------+------+--------+
| id   | val  | num    |
+------+------+--------+
|    1 | a    |  10.20 |
|    2 | b    |  22.22 |
|    3 | 3    |  33.22 |
|    4 | 222  | 222.22 |
|    5 | 10   |  10.10 |
|    6 | 10   |  10.20 |
|    7 | 10   |   NULL |
+------+------+--------+
7 rows in set (0.000 sec)

MariaDB [test]> select count(*) from tb_1;
+----------+
| count(*) |
+----------+
|        7 |
+----------+
1 row in set (0.000 sec)

MariaDB [test]> select count(1) from tb_1;
+----------+
| count(1) |
+----------+
|        7 |
+----------+
1 row in set (0.000 sec)
```

### count null

在有些情况下，数据内可能含有null，将列名作为参数时会得到NULL 之外的数据行数。如下例

```
MariaDB [test]> select count(num) from tb_1;
+------------+
| count(num) |
+------------+
|          6 |
+------------+
```

因为num字段某一行数据为null

### count distinct

在某些情况下，我们需要统计 某个字段中不重复的计数，则可以仿照select 部分的distinct 在count函数参数列名前加distinct，如下例

```
MariaDB [test]> select count(distinct num) from tb_1;
+---------------------+
| count(distinct num) |
+---------------------+
|                   5 |
+---------------------+
1 row in set (0.000 sec)
```

## Sum,avg,max,min

这些最常用的聚合函数，和count的使用方法非常类似，除了其不能使用*作为参数。

### sum

```
MariaDB [test]> select sum(num) from tb_1;
+----------+
| sum(num) |
+----------+
|   308.16 |
+----------+
1 row in set (0.000 sec)
```

如果参考之前的案例，你会发现其实num字段时有一条记录为null的，我之前说过，任何和null的运算结果都为null，为什么sum能得到正确的结果。

这就算聚合函数的一个特性（其实之前count的时候已经能猜出），就是聚合函数会自动忽略其参数内的null值

### 计算平均值

```
MariaDB [test]> select avg(num) from tb_1;
+-----------+
| avg(num)  |
+-----------+
| 51.360000 |
+-----------+
1 row in set (0.000 sec)
```

注意分母是行数，也就是之前count(num)出现的6，而不是以count(*)作为分母

### max，min

​	想要计算出多条记录中的最大值或最小值，可以分别使用MAX 和MIN函数，它们是英语maximam（最大值）和minimum（最小值）的缩写，很容易记住。

​	MAX/MIN 函数和SUM/AVG 函数有一点不同，那就是SUM/AVG 函数只能对数值类型的列使用，而MAX/MIN 函数原则上可以适用于任何数据类型的列（本质上就是比较嘛）

​	也就是说，只要是能够排序的数据，就肯定有最大值和最小值，也就能够使用这两个函数。对日期来说，平均值和合计值并没有什么实际意义，因此不能使用SUM/AVG 函数。这点对于字符串类型的数据也适用，字符串类型的数据能够使用MAX/MIN 函数，但不能使用SUM/AVG 函数。

### distinct

去重在这里也和count一样

```
MariaDB [test]> select sum(distinct num) from tb_1;
+-------------------+
| sum(distinct num) |
+-------------------+
|            297.96 |
+-------------------+
1 row in set (0.000 sec)
```

# 分组(group by)

使用GROUP BY子句可以像切蛋糕那样将表分割。group by子句定义的 分组条件中，取值相同的数据被分为一组。

## 语法

`[GROUP BY {col_name | expr | position},...]`

如之前select部分的group by子句所说，group by 后面跟一些字段，表达式列表。

在GROUP BY 子句中指定的列称为聚合键或者分组列，可以通过逗号分隔指定多列。

## 注意点

使用聚合函数和GROUP BY子句时需要注意以下4点。

1. 只能写在SELECT子句之中，一般写在from子句之后（如无where子句），或者写在where子句之后
2. GROUP BY子句中不能使用SELECT子句中列的别名
3. GROUP BY子句的聚合结果是无序的
4. WHERE子句中不能使用聚合函数
5. 如果有group by子句，那么select部分就只能有group by的字段以及聚合函数。否则会出现因为数据粒度不一致导致的报错

> 注：通常我们把使用group by的行为叫做改变数据粒度。可以直观的理解，数据被group by之后，能表现的粒度会变得越粗（比如如果统计全国人口，其粒度肯定比按省份统计各省人口要粗）



## 如下例：

```
MariaDB [test]> select * from tb_1 order by 2;
+------+------+--------+
| id   | val  | num    |
+------+------+--------+
|    5 | 10   |  10.10 |
|    6 | 10   |  10.20 |
|    7 | 10   |   NULL |
|    4 | 222  | 222.22 |
|    3 | 3    |  33.22 |
|    1 | a    |  10.20 |
|    8 | a    |  33.00 |
|    2 | b    |  22.22 |
+------+------+--------+
8 rows in set (0.000 sec)
```

数据中，我们选取val字段作为分组条件。

```
MariaDB [test]> select val,count(*) from tb_1 group by val;
+------+----------+
| val  | count(*) |
+------+----------+
| 10   |        3 |
| 222  |        1 |
| 3    |        1 |
| a    |        2 |
| b    |        1 |
+------+----------+
5 rows in set (0.000 sec)
```

可以看到，val字段相同的数据，被合并成了一行记录。

## null的情况

之前我们再各种情形下都需要考虑null的情况，那么在group by时，如果group by的字段中有null值出现，会是什么情况呢？

当聚合键中包含NULL 时，也会将NULL 作为一组特定的数据

```
MariaDB [test]> select val,count(*) from tb_1 group by val;
+------+----------+
| val  | count(*) |
+------+----------+
| NULL |        1 |
| 10   |        3 |
| 222  |        1 |
| 3    |        1 |
| a    |        2 |
| b    |        1 |
+------+----------+
6 rows in set (0.000 sec)
```

## 数据过滤

与where一起group by

```
MariaDB [test]> select val,count(*) from tb_1
    -> where num>20
    -> group by val ;
+------+----------+
| val  | count(*) |
+------+----------+
| 222  |        1 |
| 3    |        1 |
| a    |        1 |
| b    |        1 |
+------+----------+
4 rows in set (0.000 sec)
```

此处where的过滤条件是先于group by的。

有同学可能会问了，那么如果我想对 聚合后的数据进行过滤怎么办呢？

这个时候我们可以使用：子查询，或者having子句。