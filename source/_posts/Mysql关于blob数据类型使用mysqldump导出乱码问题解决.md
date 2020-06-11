---
title: Mysql关于blob数据类型使用mysqldump导出乱码问题解决
date: 2020-06-11 09:17:10
tags:
    - Mysql
categories: 运维日志
---
![homePage](/upload/homePage/20200611092200.jpg)
<!--more-->

## 情景
最近查看服务器上的Mysql自动备份，发现关于blob数据类型使用mysqldump导出有乱码的情况，命令如下：
```
mysqldump -h$db_host -u$db_username -p$db_password --events --default-character-set=utf8 --all-databases | gzip > $backup_dir/mysql_dump_$today.sql.gz
```

## 原因分析
看到有乱码，肯定第一时间怀疑是字符集的问题，但是又确认了一下utf8是没有问题的，而且发现乱码是因为包含blob类型字段，所以查了一下，原来少了个参数--hex-blob，看参数名应该是以二进制形式导出blob。

## 解决方案
修改后的命令：
```
mysqldump -h$db_host -u$db_username -p$db_password --events --default-character-set=utf8 --hex-blob --all-databases | gzip > $backup_dir/mysql_dump_$today.sql.gz
```

运行，导出的sql中没有乱码了。