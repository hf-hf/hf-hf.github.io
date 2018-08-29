---
title: 批量kill mysql中锁表的sql
date: 2018-08-29 10:39:45
tags:
    - mysql
---

## 情景
公司开发新平台项目，迁移数据库后，发现某位同学的sql有问题（迁移后的数据库中缺少一个索引），导致mysql中出现很多运行时间长且未结束的mysql，连带其他查询也很缓慢。

## 解决方案
正常开发中出现锁表的情况，一般自己show processlist后，再自行kill对应sql即可，但是现在这种sql比较多，一条条kill太耗时了（主要还是自己比较懒）。

```
show processlist;
kill xxx;
```

## 批量kill方案
1) 方案1：通过SQL语句临时文件

```
# 构造出批量kill的sql文件tmp
mysql>select concat('KILL ',id,';') from information_schema.processlist where user='root' into outfile '/tmp/a.txt';
Query OK, 2 rows affected (0.00 sec)
# 执行该文件tmp 
mysql>source /tmp/a.txt;
Query OK, 0 rows affected (0.00 sec)
```

2) 方案2：通过Shell脚本实现

```
#杀掉锁定的MySQL连接
for id in `mysqladmin processlist|grep -i locked|awk '{print $1}'`
do
   mysqladmin kill ${id}
done
```

3) 方案3：杀掉当前所有的/指定用户所有的MySQL连接

```
# 杀掉当前所有的MySQL连接
mysqladmin -uroot -p processlist|awk -F "|" '{print $2}'|xargs -n 1 mysqladmin -uroot -p kill
# 杀掉指定用户运行的连接，这里为ddtest
mysqladmin -uroot -p processlist|awk -F "|" '{if($3 == "ddtest")print $2}'|xargs -n 1 mysqladmin -uroot -p kill
```



