---
title: operation and function
author: douqq
date:  2020-08-17 12:00:00 +0800
categories: [Database, SQL]
tags: []
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

In, not in





# 字符串函数



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
