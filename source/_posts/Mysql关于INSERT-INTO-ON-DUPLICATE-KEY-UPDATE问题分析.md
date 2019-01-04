---
title: Mysql关于INSERT INTO...ON DUPLICATE KEY UPDATE问题分析
date: 2019-01-04 09:41:39
tags:
    - Mysql
categories: 运维日志
---
![homePage](/upload/homePage/20190104095502.jpg)
<!--more-->

## 情景
元旦假期期间，公司Mysql某张大增量表，该表创建了3个字段的UNIQUE索引，并且为自增长主键，主键增长到int上限2147483647时，无法再插入新数据行，而在使用该表的业务代码中，通过`INSERT INTO...ON DUPLICATE KEY UPDATE`实现记录不存在时，插入该记录，存在则更新，但在主键到达上限这一情景下，程序没有报错，系统会反复更新最后一条数据（id为2147483647）。

> ### mysql中int、bigint、smallint 和 tinyint的区别详细介绍
> 1 bytes = 8 bit ,一个字节最多可以代表的数据长度是2的8次方 11111111 在计算机中也就是-128到127
> 
> BIT[M]
> 位字段类型，M表示每个值的位数，范围从1到64，如果M被忽略，默认为1
> 
> TINYINT[(M)] [UNSIGNED] [ZEROFILL]  M默认为4
> 很小的整数。带符号的范围是-128到127。无符号的范围是0到255。
> 
> BOOL，BOOLEAN
> 是TINYINT(1)的同义词。zero值被视为假。非zero值视为真。
> 
> SMALLINT[(M)] [UNSIGNED] [ZEROFILL] M默认为6
> 小的整数。带符号的范围是-32768到32767。无符号的范围是0到65535。
> 
> MEDIUMINT[(M)] [UNSIGNED] [ZEROFILL] M默认为9
> 中等大小的整数。带符号的范围是-8388608到8388607。无符号的范围是0到16777215。
> 
> INT[(M)] [UNSIGNED] [ZEROFILL]   M默认为11
> 普通大小的整数。带符号的范围是-2147483648到2147483647。无符号的范围是0到4294967295。
> 
> BIGINT[(M)] [UNSIGNED] [ZEROFILL] M默认为20
> 大整数。带符号的范围是-9223372036854775808到9223372036854775807。无符号的范围是0到18446744073709551615。

## 问题分析
首先看一下官网上关于该语句的描述，在[13.2.6.2 INSERT ... ON DUPLICATE KEY UPDATE Syntax](https://dev.mysql.com/doc/refman/5.7/en/insert-on-duplicate.html)第一段有如下解释：

If you specify an ON DUPLICATE KEY UPDATE clause and a row to be inserted would cause a duplicate value in a UNIQUE index or PRIMARY KEY, an UPDATE of the old row occurs. For example, if column a is declared as UNIQUE and contains the value 1, the following two statements have similar effect:
```
INSERT INTO t1 (a,b,c) VALUES (1,2,3)
  ON DUPLICATE KEY UPDATE c=c+1;

UPDATE t1 SET c=c+1 WHERE a=1;
```
(The effects are not identical for an InnoDB table where a is an auto-increment column. With an auto-increment column, an INSERT statement increases the auto-increment value but UPDATE does not.)

其大致意义如下：<br/>
如果指定`ON DUPLICATE KEY UPDATE`子句并且要插入的行将导致<font color=red>UNIQUE索引/主键</font>中出现重复值，则会根据UNIQUE索引/主键进行旧行的UPDATE。例如，如果将column a声明为UNIQUE并包含该值1，则以下两个语句具有类似的效果：

（对于InnoDB表，如果a是自动递增列，则效果不相同。对于自动增量列，INSERT语句会增加自动增量值，但UPDATE不会。）

代入实际发生的问题中，在InnoDb表，自增长主键增长到上限时，此时再执行insert插入，会提示Duplicate entry '2147483647' for key 'PRIMARY'，其满足`INSERT INTO...ON DUPLICATE KEY UPDATE`的判定条件，会被判定为数据表中已存在该主键的数据行，因此该语句等同于`UPDATE...WHERE ID = 2147483647`，即根据主键ID=2147483647更新该数据行，若`INSERT INTO...ON DUPLICATE KEY UPDATE`多次执行，ID=2147483647的数据也会被多次更新。

这种情况下之前已创建的UNIQUE索引(3个业务字段的唯一索引)并不会被触发，因为该数据行在执行时，首先就会被主键冲突所拦截，其结果就是情景中所描述的，主键到达上限后，最后一条数据会被反复更新。

## 总结
通过创建UNIQUE索引，配合`INSERT INTO...ON DUPLICATE KEY UPDATE`可以快速实现业务上的相同即更新，否则便插入，但是在某些情况下，如主键增长到达上限时，可能会导致一些预料之外的情况发生，不仅会隐藏新数据行无法插入的问题（系统不会抛出错误，而是根据主键冲突去做UPDATE，并且成功了0.0），而且会导致最后一行数据被更新为异常的数据。