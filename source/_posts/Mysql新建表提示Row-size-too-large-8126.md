---
title: Mysql新建表提示Row size too large (> 8126)
date: 2018-06-27 16:59:32
tags:
    -- Mysql
    -- too large
categories: 运维日志
---

## 情景
Mysql新增数据表，字段类型为varchar长度较长，且该类型字段较多有200多个，进行create时报错Row size too large (> 8126)。

## 解决方案
在新建数据表create语句前追加：set innodb_strict_mode=0;再新建即可。
