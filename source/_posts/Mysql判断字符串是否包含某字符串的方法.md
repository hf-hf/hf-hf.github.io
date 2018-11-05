---
title: Mysql判断字符串是否包含某字符串的方法
date: 2018-11-05 13:58:03
tags:
    - Mysql
---
![homePage](/upload/homePage/20181105161103.jpg)
<!--more-->

## 情景
Mysql下需要根据数据库某字段是否包含某字符串/字符来进行筛选，将包含该字符串/字符的数据展示出来。

## 解决方案
经调查，使用Mysql的LOCATE函数可满足该需求。

函数说明如下：
LOCATE(substr,str) 
POSITION(substr IN str) 
返回子串 substr 在字符串 str 中第一次出现的位置。如果子串 substr 在 str 中不存在，返回值为 0： 

```
mysql> SELECT LOCATE('bar', ‘foobarbar'); 
-> 4 
mysql> SELECT LOCATE('xbar', ‘foobar'); 
-> 0 
```

## 扩展使用
该函数也可用于排序中，例如：将资源中包含*的url展示的最前面。

```
mysql> SELECT * FROM acc_resource ORDER BY LOCATE('*',url) DESC
```

PS：仅限在关联表较少、表数据有限的情况下，否则可能会导致查询过于缓慢。

