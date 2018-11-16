---
title: Linux Crontab定时任务生成文件为空，手动执行脚本正常问题
date: 2018-11-16 11:50:30
tags:
    - linux
categories: 运维日志
---

![homePage](/upload/homePage/20181116140902.jpg)
<!--more-->
## 情景
双十一新购入了一台云服务器，将原先服务器上的Mysql迁移了过来，在配置定时备份脚本时，发现手动执行脚本正常，但是通过Crontab定时运行，能够生成文件但是内容为空。

Shell备份脚本内容如下：

```
echo "mysql backup start..."$start_date
# config information 定义数据库连接 
db_host=localhost  
db_port=3306
db_username=XXXXX  
db_password=XXXXX

backup_dir=/usr/bak/backups/mysql-backup/
today=`date "+%Y%m%d%H"`

#cd $backup_dir

# -d 参数判断 $folder 是否存在
#if [ ! -d '$today']; then
#  mkdir $today
#fi

mysqldump -h$db_host -u$db_username -p$db_password --events --all-databases | gzip > $backup_dir/mysql_dump_$today.sql.gz
find $backup_dir -mtime +7 -name 'mysql_dump_*[1-9].sql.gz' -exec rm -rf {} \;
find $backup_dir -mtime +92 -name 'mysql_dump_*.sql.gz' -exec rm -rf {} \;
 
echo "mysql backup end..."
```

## 原因
调查后发现原因为在Crontab任务中没有环境变量，执行命令时需要通过绝对路径。

修改Shell执行mysqldump备份这一行，修改内容如下。

```
mysqldump_dir=/usr/local/mysql/bin/
$mysqldump_dir/mysqldump  -h$db_host -u$db_username -p$db_password --events --all-databases | gzip > $backup_dir/mysql_dump_$today.sql.gz
```
