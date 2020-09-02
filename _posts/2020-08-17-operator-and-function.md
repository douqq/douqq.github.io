---
title: operation and function
author: douqq
date:  2020-08-17 12:00:00 +0800
categories: [Database, SQL]
tags: [sql,function]
---



# 运算符概览

运算符就是对其两边的列或者值进行运算（计算或者比较大小等）的符号。 
使用算术运算符可以进行四则运算。 
括号可以提升运算的优先顺序（优先进行运算）。
包含NULL的运算，其结果也是NULL。
比较运算符可以用来判断列或者值是否相等，还可以用来比较大小。
判断是否为NULL，需要使用IS NULL或者IS NOT NULL运算符。

## 运算符优先级

在mysql中，有如下优先级，最上方优先级最高，从上往下逐次降低

 ```
  BINARY, COLLATE
  !
  - (unary minus), ~ (unary bit inversion)
    ^
    *, /, DIV, %, MOD
    -, +
    <<, >>
    &
    |
    = (comparison), <=>, >=, >, <=, <, <>, !=, IS, LIKE, REGEXP, IN, MEMBER OF
    BETWEEN, CASE, WHEN, THEN, ELSE
    NOT
    AND, &&
    XOR
    OR, ||
    = (assignment), :=
 ```

  当然为了保险，还是使用()改变优先级最好



# 算术运算符

四则运算所使用的运算符（+、-、*、/）称为算术运算符。运算符就是使用其两边的值进行四则运算或者字符串拼接、数值大小比较等运算，并返回结果的符号。加法运算符（+）前后如果是数字或者数字类型的列名的话，就会返回加法运算后的结果。

取模，乘方等其他运算符可以使用函数实现，或者进一步阅读参考手册

例：

```
MariaDB [test]> select 1+2*3;
+-------+
| 1+2*3 |
+-------+
|     7 |
+-------+
1 row in set (0.000 sec)
```

除了进行常量计算，你也可以在字段的基础上进行运算，比如让某个字段值*2

```
MariaDB [test]> select t.*,t.num*2 from tb_1 t;
+------+------+-------+---------+
| id   | val  | num   | t.num*2 |
+------+------+-------+---------+
|    1 | a    | 10.20 |   20.40 |
+------+------+-------+---------+
1 row in set (0.000 sec)
```



# 比较运算符

| 运算符 | 含义 |
| ------ | ---- |
|=    |和~相等      |
|<>   |和~不相等     |
|\>=  |大于等于~     |
|\>   |大于~       |
|<=   |小于等于~     |
|<    |小于~       |

> 注：有些数据库 != 也可以表达不等于

这些比较运算符可以对字符、数字和日期等几乎所有数据类型的列和值进行比较

例：

```
MariaDB [test]> select * from tb_1 t where t.num>10;
+------+------+-------+
| id   | val  | num   |
+------+------+-------+
|    1 | a    | 10.20 |
|    2 | b    | 22.22 |
+------+------+-------+
2 rows in set (0.000 sec)
```

## 字符串比较

假定有如下数据：

```
MariaDB [test]> select * from tb_1 t;
+------+------+--------+
| id   | val  | num    |
+------+------+--------+
|    1 | a    |  10.20 |
|    2 | b    |  22.22 |
|    3 | 3    |  33.22 |
|    4 | 222  | 222.22 |
|    5 | 10   |  10.10 |
+------+------+--------+
5 rows in set (0.000 sec)
```

我们尝试使用val字段对于大于'2'的数据进行查询

```
MariaDB [test]> select * from tb_1 t where val>'2';
+------+------+--------+
| id   | val  | num    |
+------+------+--------+
|    1 | a    |  10.20 |
|    2 | b    |  22.22 |
|    3 | 3    |  33.22 |
|    4 | 222  | 222.22 |
+------+------+--------+
4 rows in set (0.000 sec)
```

如果你试图用 数字去理解 '10' 和'2' 的大小，则不会得到争取的结论。

实际上字符串的比较是使用其字符集的字节码进行比较的，通常情况下，就是字符串按字符顺序比较ascii码

## 比较与null

```
  MariaDB [test]> select * from tb_1 t where val = null;
 Empty set (0.000 sec)
```

如之前所说，null的运算只能使用 is null或者 is not null来进行判断。

任何和null的运算都是null或false，在此处，val = null 这个表达式的运算结果为false

## between

Betweeen  **min** and **max** 表达式一般用于一段范围的限定判断，实际上可以通过 >= and <= 来等价替换。但是从sql 可读性上来说，使用 between会更好看

If expr is greater than or **equal** to min and expr is less than or **equal** to max, BETWEEN returns 1, otherwise it returns 0.

between对于左右判断都是包含的（也就是数学上的闭合）





# 逻辑运算符

通过使用逻辑运算符，可以将多个查询条件进行组合。

|Name| Description     |
| ------ | ---- |
|AND, && |Logical AND  |
|NOT, ! | Negates value |
|OR, \|\| | Logical OR    |
|XOR |Logical XOR      |

## NOT

想要指定“不是~”这样的否定条件时，需要使用 NOT 运算符,NOT 不能单独使用，必须和其他查询条件组合起来使用。

如下例：

```
MariaDB [test]> select * from tb_1 where num>20;
+------+------+--------+
| id   | val  | num    |
+------+------+--------+
|    2 | b    |  22.22 |
|    3 | 3    |  33.22 |
|    4 | 222  | 222.22 |
+------+------+--------+
3 rows in set (0.000 sec)

MariaDB [test]> select * from tb_1 where not num>20;
+------+------+-------+
| id   | val  | num   |
+------+------+-------+
|    1 | a    | 10.20 |
|    5 | 10   | 10.10 |
+------+------+-------+
2 rows in set (0.000 sec)
```

## AND运算符和OR运算符

在实际使用当中，往往都是同时指定多个查询条件对数据进行查询的。比如同时满足 条件1和条件2的情况。

在WHERE 子句中使用AND 运算符或者OR 运算符，可以对多个查询条件进行组合。AND 运算符在其两侧的查询条件**都**成立时整个查询条件才成立，其意思相当于“并且”。OR 运算符在其两侧的查询条件**有一个成立**时整个查询条件都成立，

```
MariaDB [test]> select * from tb_1 where num>20 and val='b';
+------+------+-------+
| id   | val  | num   |
+------+------+-------+
|    2 | b    | 22.22 |
+------+------+-------+
1 row in set (0.000 sec)
```

## 括号

括号用于改变运算符优先级

AND 运算符优先于OR 运算符 。如果你按照自然语序去写一些逻辑表达式时，可能会得到不正确的结果。另外在组合非常复杂的逻辑表达式时，就更离不开括号了。

```
MariaDB [test]> select * from tb_1 where (num>20) and (val='b' or val='3');
+------+------+-------+
| id   | val  | num   |
+------+------+-------+
|    2 | b    | 22.22 |
|    3 | 3    | 33.22 |
+------+------+-------+
2 rows in set (0.000 sec)
```



# 集合操作符

## In, not in

当我们想表达 某个数据 是否再一个数据集合之中时，可以使用in 操作符，反之可以使用not in。其实in 可以转义成为所有元素的 or = 操作。

```
MariaDB [test]> select 3 in (2,3,4,5);
+----------------+
| 3 in (2,3,4,5) |
+----------------+
|              1 |
+----------------+
1 row in set (0.001 sec)
```

但需要注意的是，在使用IN 和NOT IN 时是无法选取出NULL 数据的

## IN和子查询

IN（NOT IN ）可以使用子查询作为其参数

你可以理解，子查询的结果其实就是一个集合，in 使用一个集合作为其参数，无所谓这个集合是写死的常量，还是动态结果的子查询，如下例：

```
MariaDB [test]> select id from tb_1;
+------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
|    4 |
|    5 |
|    6 |
|    7 |
|    8 |
|    9 |
+------+
9 rows in set (0.000 sec)

MariaDB [test]> select 4 in (select id from tb_1);
+----------------------------+
| 4 in (select id from tb_1) |
+----------------------------+
|                          1 |
+----------------------------+
1 row in set (0.003 sec)
```





# 函数概览

所谓函数，就是输入某一值得到相应输出结果的功能，输入值称为参数（parameter），输出值称为返回值。

函数大致可以分为以下几种。

● 算术函数（用来进行数值计算的函数）

● 字符串函数（用来进行字符串操作的函数）

● 日期函数（用来进行日期操作的函数）

● 转换函数（用来转换数据类型和值的函数）

● 聚合函数（用来进行数据聚合的函数）

函数特别多，本文只介绍一些基本常用的

所有的函数，能基于常量计算的，也都可以基于字段（以字段为参数）计算，为了演示方便，下文函数部分都直接使用 select function的方式以常量进行演示

# 算术函数

## ROUND

ROUND(X), ROUND(X,D) D defaults to 0 if not specified

round就是四舍五入，d参数可以决定保留的小数位，d如果是负数，则从小数点左边算起

下例：

```
MariaDB [test]> select round(12.113), round(12.456,2),round(357.56,-2),round(347.56,-1);
+---------------+-----------------+------------------+------------------+
| round(12.113) | round(12.456,2) | round(357.56,-2) | round(347.56,-1) |
+---------------+-----------------+------------------+------------------+
|            12 |           12.46 |              400 |              350 |
+---------------+-----------------+------------------+------------------+
```



## ABS

取绝对值，同样和null运算为null

```
MariaDB [test]> select abs(1),abs(-1),abs(null);
+--------+---------+-----------+
| abs(1) | abs(-1) | abs(null) |
+--------+---------+-----------+
|      1 |       1 |      NULL |
+--------+---------+-----------+
1 row in set (0.000 sec)
```

## MOD

MOD(被除数，除数)

MOD 是计算除法余数（求余）的函数，是modulo 的缩写。例如，7 / 3 的余数是1，因此MOD（7, 3）的结果也是1.

MariaDB [test]> select mod(7,3);
+----------+
| mod(7,3) |
+----------+
|        1 |
+----------+
1 row in set (0.000 sec)

# 字符串函数

## 字符串拼接

在实际业务中，我们经常会碰到abc + de = abcde 这样希望将字符串进行拼接的情况。在SQL 中，可以通过CONCAT函数来实现

```
MariaDB [test]> select concat('ab','cd');
+-------------------+
| concat('ab','cd') |
+-------------------+
| abcd              |
+-------------------+
1 row in set (0.000 sec)
```

进行字符串拼接时，如果其中包含NULL，那么得到的结果也是NULL。

```
MariaDB [test]> select concat('ab','cd',null);
+------------------------+
| concat('ab','cd',null) |
+------------------------+
| NULL                   |
+------------------------+
1 row in set (0.000 sec)
```

有些时候并不是你主动写的null，而是某个字段其中包含了你意想不到的null导致的错误。

注：在大多数数据库，可以使用 || (双竖线)操作符来进行字符串拼接

## length

想要知道字符串中包含多少个字**符**时，可以使用LENGTH（长度）函数

在大多数数据库，length都是返回字符的，但是在mysql中，length函数返回的是字符串的字节数（这就是字符串对应的字符集实际占有的字节长度）

```
MariaDB [test]> select length('abcd');
+----------------+
| length('abcd') |
+----------------+
|              4 |
+----------------+
1 row in set (0.001 sec)

MariaDB [test]> select length('中国');
+------------------+
| length('中国')   |
+------------------+
|                6 |
+------------------+
1 row in set (0.000 sec)
```

如果要返回字符串内字符的长度，在mysql内，使用CHAR_LENGTH()函数

```
MariaDB [test]> select char_length('你好 中国');
+------------------------------+
| char_length('你好 中国')     |
+------------------------------+
|                            5 |
+------------------------------+
1 row in set (0.000 sec)
```

## 大小写转换

LOWER 函数只能针对英文字母使用，它会将参数中的字符串全都转换为小写

```
MariaDB [test]> select lower('ABcdE');
+----------------+
| lower('ABcdE') |
+----------------+
| abcde          |
+----------------+
1 row in set (0.000 sec)
```

UPPER 函数则转换为大写

```
MariaDB [test]> select upper('ABcdE');
+----------------+
| upper('ABcdE') |
+----------------+
| ABCDE          |
+----------------+
1 row in set (0.000 sec)
```

## 字符串替换

`REPLACE(对象字符串，替换前的字符串，替换后的字符串)`

使用REPLACE 函数，可以将字符串的一部分替换为其他的字符串

```
MariaDB [test]> select replace('abcdeabdec','a','x');
+-------------------------------+
| replace('abcdeabdec','a','x') |
+-------------------------------+
| xbcdexbdec                    |
+-------------------------------+
```

## 字符串的截取

`SUBSTRING（对象字符串 FROM 截取的起始位置 FOR 截取的字符数）`

使用SUBSTRING 函数可以截取出字符串中的一部分字符串。截取的起始位置从字符串最左侧开始计算

```
MariaDB [test]> select substring('123456' from 2 for 3);
+----------------------------------+
| substring('123456' from 2 for 3) |
+----------------------------------+
| 234                              |
+----------------------------------+
1 row in set (0.000 sec)
```



## like

`expr LIKE pat [ESCAPE 'escape_char']`

如果expr或者pat是null，那么匹配结果也是null

可以使用如下2个特殊符号当做通配符

• % 匹配任意字符数字符号，即使是0个

• _ 仅匹配1个任意字符.

如下例

MariaDB [test]> SELECT 'David!' LIKE 'David

```
+------------------------+
| 'David!' LIKE 'David_' |
+------------------------+
|                      1 |
+------------------------+
1 row in set (0.000 sec)

MariaDB [test]> SELECT 'David!' LIKE '%D%v%';
+-----------------------+
| 'David!' LIKE '%D%v%' |
+-----------------------+
|                     1 |
+-----------------------+
1 row in set (0.000 sec)
```

如果要匹配这2个字符，%,_, 请使用\转义，如

```
MariaDB [test]> SELECT 'David_' LIKE 'David\_';
+-------------------------+
| 'David_' LIKE 'David\_' |
+-------------------------+
|                       1 |
+-------------------------+
1 row in set (0.000 sec)
```

# 日期函数

## CURRENT_DATE

CURRENT_DATE 函数能够返回SQL 执行的日期，也就是该函数执行时的日期。由于没有参数，因此无需使用括号。

```
MariaDB [test]> select current_date;
+--------------+
| current_date |
+--------------+
| 2020-08-21   |
+--------------+
1 row in set (0.001 sec)
```

## CURRENT_TIME

CURRENT_TIME 函数能够取得SQL 执行的时间，也就是该函数执行时的时间

```
MariaDB [test]> select current_time;
+--------------+
| current_time |
+--------------+
| 18:18:41     |
+--------------+
```

## CURRENT_TIMESTAMP

CURRENT_TIMESTAMP 函数具有CURRENT_DATE + CURRENT_TIME 的功能。使用该函数可以同时得到当前的日期和时间，当然也可以从结果中截取日期或者时间

```
MariaDB [test]> select current_timestamp;
+---------------------+
| current_timestamp   |
+---------------------+
| 2020-08-21 18:19:31 |
+---------------------+
1 row in set (0.000 sec)
```

## EXTRACT

`EXTRACT(日期元素 FROM 日期)`

使用EXTRACT 函数可以截取出日期数据中的一部分，例如“年”“月”，或者“小时”“秒”等。该函数的返回值并不是日期类型而是**数值**类型。

```
MariaDB [test]> SELECT CURRENT_TIMESTAMP,
    -> EXTRACT(YEAR FROM CURRENT_TIMESTAMP) AS year,
    -> EXTRACT(MONTH FROM CURRENT_TIMESTAMP) AS month,
    -> EXTRACT(DAY FROM CURRENT_TIMESTAMP) AS day,
    -> EXTRACT(HOUR FROM CURRENT_TIMESTAMP) AS hour,
    -> EXTRACT(MINUTE FROM CURRENT_TIMESTAMP) AS minute,
    -> EXTRACT(SECOND FROM CURRENT_TIMESTAMP) AS second;
+---------------------+------+-------+------+------+--------+--------+
| CURRENT_TIMESTAMP   | year | month | day  | hour | minute | second |
+---------------------+------+-------+------+------+--------+--------+
| 2020-08-21 18:20:53 | 2020 |     8 |   21 |   18 |     20 |     53 |
+---------------------+------+-------+------+------+--------+--------+
1 row in set (0.000 sec)
```

# 转换函数

## cast

`CAST（转换前的值 AS 想要转换的数据类型）`

之所以需要进行类型转换，是因为可能会插入与表中数据类型不匹配的数据，或者在进行运算时由于数据类型不一致发生了错误，又或者是进行自动类型转换会造成处理速度低下。这些时候都需要事前进行数据类型转换。

```
MariaDB [test]> select cast('123' as int),cast('2020-01-01' as date);
+--------------------+----------------------------+
| cast('123' as int) | cast('2020-01-01' as date) |
+--------------------+----------------------------+
|                123 | 2020-01-01                 |
+--------------------+----------------------------+
1 row in set (0.000 sec)
```

## COALESCE

COALESCE 是SQL 特有的函数。该函数会返回可变参数A 中左侧开始第1 个不是NULL 的值。该函数最主要的作用就是处理各个字段中可能出现的null值。

```
MariaDB [test]> select coalesce(null,'-99');
+----------------------+
| coalesce(null,'-99') |
+----------------------+
| -99                  |
+----------------------+
1 row in set (0.001 sec)
```



## case 表达式

CASE表达式分为简单CASE表达式和搜索CASE表达式两种。搜索CASE表达式包含简单CASE表达式的全部功能。

```sql
搜索CASE表达式

CASE WHEN <求值表达式> THEN <表达式>

WHEN <求值表达式> THEN <表达式>

WHEN <求值表达式> THEN <表达式>

.. .

ELSE <表达式>

END
```

​	WHEN 子句中的“< 求值表达式>”就是类似“列 = 值”这样，返回值为真值（TRUE/FALSE/UNKNOWN）的表达式。我们也可以将其看作使用=、!= 或者LIKE、BETWEEN 等谓词编写出来的表达式。

​	CASE 表达式会从对最初的WHEN 子句中的“< 求值表达式>”进行求值开始执行（从上往下的顺序）如果结果为真（TRUE），那么就返回THEN 子句中的表达式，CASE 表达式的执行到此为止。如果结果不为真，那么就跳转到下一条WHEN 子句的求值之中。如果直到最后的WHEN 子句为止返回结果都不为真，那么就会返回ELSE中的表达式，执行终止。

​	CASE 表达式最后的“END”是不能省略的，请大家特别注意不要遗漏

```
简单CASE表达式

CASE <表达式>

WHEN <表达式> THEN <表达式>

WHEN <表达式> THEN <表达式>

WHEN <表达式> THEN <表达式>

.. .

ELSE <表达式>

END
```

如下例：

```
MariaDB [test]> select case
    -> when 1=2 then 0
    -> when 'a'='b' then 0
    -> else 1
    -> end;
+------------------------------------------------------+
| case
when 1=2 then 0
when 'a'='b' then 0
else 1
end |
+------------------------------------------------------+
|                                                    1 |
+------------------------------------------------------+
1 row in set (0.000 sec)
```

