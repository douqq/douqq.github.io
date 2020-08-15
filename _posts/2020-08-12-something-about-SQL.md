---
title: somethint about SQL
author: douqq
date:  2020-08-15 16:00:34 +0800
categories: [Database, SQL]
tags: [sql]
---
# ANSI标准

最早
ANSI X3.135
October 16, 1986
INFORMATION SYSTEMS - DATABASE LANGUAGE - SQL
目前最新的
ISO/IEC 9075-1:2016
Information technology — Database languages — SQL



# 直观理解

​	数据库在早期发展阶段，有各种类型，比如网状数据库。后来关系型数据库占据主流，而对应的，SQL是关系数据库的主要操作语言，其目的是通过类似自然语言的方式直接去问数据库要数据。

​	当然随着大数据技术的兴起，大量的NOSQL数据库也开始得到应用，对于此类数据的获取可能通过其他方式（比如API）。但是总的来说，SQL是数据处理人员使用范围最广，沟通最为基础的技术。很多新技术也会为了获得最大的影响力而提供SQL层面的转义和兼容（如HIVE）。

​	不同的数据库厂家，对于ANSI标准的兼容程度不尽相同，但是90%以上的功能都是相通的。数据库主要由表（table）构成，表由字段（column）构成。在关系数据库的概念里面，表的每一行代表一个实体（entity），行的每一列（column）代表一个属性。列在存储层面支持不同的数据类型，但是大体上分为：字符串，数字，日期（本质上也是数字），近些年随着数据的负责，部分数据库支持一些复杂数据类型比如：set，list，json等。

# SQL的分类

​	总的来说，SQL分为3类，即:DDL,DML,DCL.其中也有人单独把SELECT预计拿出来分类的。但是大体上，业界一般是这么分的。

## DDL

​	我们知道数据放在数据库内，是以表的形式存放的，一个表有其结构schema（即columns的类型和顺序，以及其他物理特性比如存储分区等）。

​	DDL sql一般就是用于定义或者修改schema。它一般可以实现创建对象，修改对象，删除对象等工作。在分工上，这部分权限一般归DBA所有。

## DML

​	DML是整个sql内最丰富的最基础的能力，其核心功能就是获取数据，或者修改数据。我一般喜欢把select语句归类为DML

## DCL

​	作为整个IT系统最核心的数据存储模块，安全和权限必然是重中之重，DCL语句一般用于控制谁有权限做什么事情。



# 参考学习网站：

http://xuesql.cn/

https://www.liaoxuefeng.com/wiki/1177760294764384/1179613436834240

https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Oracle-and-Standard-SQL.html#GUID-330DEBBB-006E-4B35-A516-5C0AEFFE06B9