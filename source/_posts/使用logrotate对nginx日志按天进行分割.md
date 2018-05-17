---
title: 使用logrotate对nginx日志按天进行分割
date: 2018-05-17 11:04:43
tags:
    - logrotate
    - nginx
---

##  情景：

今天运维大哥来找，某某服务器磁盘报警啦。

``` bash
# df -m
``` 

看了一下/dev/vda1果然快满了，磁盘使用率97%。。。

没办法先紧急清理一下log吧

查一下那个目录占用空间比较大

``` bash
# du -h --max-depth=1
``` 

噼里啪啦一阵之后发现nginx下的access.log居然有13G，这还能忍？赶紧压缩移到别的地方~

## 原因：

nginx会按照nginx.conf的配置生成access.log和error.log，随着访问量的增长，日志文件会越来越大，既会影响访问的速度(写入日志时间延长)，
也会增加查找日志的难度，nginx没有这种按天或更细粒度生成日志的机制。

但是线上的服务器应该都配置了logrotate,为什么会一直累加该文件，赶紧看了下果然这台服务器上没有配置T.T。

只好自己配一下了，省的以后想查日志还得自己切分。

## logrotate配置

logrotate程序是一个日志文件管理工具。用来把旧的日志文件删除，并创建新的日志文件，我们把它叫做“转储”。我们可以根据日志文件的大小，也可以根据其天数来转储，这个过程一般通过 cron 程序来执行。
logrotate程序还可以用于压缩日志文件，以及发送日志到指定的E-mail 。

## 配置选项说明
``` bash
compress：通过gzip 压缩转储旧的日志
nocompress：不需要压缩时，用这个参数 
copytruncate：用于还在打开中的日志文件，把当前日志备份并截断 
nocopytruncate：备份日志文件但是不截断 
create mode owner group：使用指定的文件模式创建新的日志文件 
nocreate：不建立新的日志文件 
delaycompress：和 compress 一起使用时，转储的日志文件到下一次转储时才压缩 
nodelaycompress：覆盖 delaycompress 选项，转储同时压缩
errors address：专储时的错误信息发送到指定的Email 地址 
ifempty：即使是空文件也转储，这个是 logrotate 的缺省选项
notifempty：如果是空文件的话，不转储 
mail address：把转储的日志文件发送到指定的E-mail 地址 
nomail：转储时不发送日志文件 
olddir directory：转储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统 
noolddir：转储后的日志文件和当前日志文件放在同一个目录下 
prerotate/endscript：在转储以前需要执行的命令可以放入这个对，这两个关键字必须单独成行 
postrotate/endscript：在转储以后需要执行的命令可以放入这个对，这两个关键字必须单独成行 
sharedscripts：所有的日志文件都轮转完毕后统一执行一次脚本 
daily：指定转储周期为每天 
weekly：指定转储周期为每周 
monthly：指定转储周期为每月 
rotate count：指定日志文件删除之前转储的次数，0 指没有备份，5 指保留5 个备份 
size size：当日志文件到达指定的大小时才转储，Size 可以指定 bytes (缺省)以及KB (sizek)或者MB
``` 

## 命令参数说明
``` bash
# logrotate –help
Usage: logrotate [OPTION...] <configfile>
  -d, --debug               调试模式，输出调试结果，并不执行。隐式-v参数
  -f, --force               强制模式，对所有相关文件进行rotate
  -m, --mail=command        发送邮件 (instead of `/bin/mail')
  -s, --state=statefile     状态文件，对于运行在不同用户情况下有用
  -v, --verbose             显示debug信息
```

## logrotate配置文件默认位置：
``` bash
/etc/logrotate.conf 通用配置文件，可以定义全局默认使用的选项。 
/etc/logrotate.d/nginx 自定义服务配置文件

/usr/local/nginx/logs/*.log
/usr/local/nginx/logs/XXX/*.log  #指定日志文件位置，可用正则匹配
```

/etc/logrotate.d/nginx
``` bash
nginx
{
    daily           #调用频率，有：daily，weekly，monthly可选
    rotate 30       #一次将存储30个归档日志。对于第31个归档，时间最久的归档将被删除。
    compress        #通过gzip 压缩转储旧的日志 
    sharedscripts   #所有的日志文件都轮转完毕后统一执行一次脚本
    postrotate      #执行命令的开始标志
        if [ -f /usr/local/nginx/logs/nginx.pid ]; then
            kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`
            #不是中止Nginx的进程，而是传递给它信号重新生成日志，如果nginx没启动不做操作
        fi
    endscript #执行命令的结束标志
}
```

## 测试配置手动执行logrotate
``` bash
# 强制执行（-f = force），冗长的-v(-v =verbose），注意调试信息默认携带-v；
# logrotate -vf /etc/logrotate.d/nginx
```

## 输出如下，查看日志文件夹生成已压缩的日志：
![logrotate_1.png](/upload/logrotate/logrotate_1.png)
![logrotate_2.png](/upload/logrotate/logrotate_2.png)

## Cron定时执行logrotate

配置文件测试完毕，logrotate实际使用一般都是通过cron来定时执行；

``` bash
# cat /etc/cron.daily/logrotate

#!/bin/sh

/usr/sbin/logrotate /etc/logrotate.conf >/dev/null 2>&1
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit 0
```

默认的logrotate已经放在/etc/cron.daily/logrotate下，让corn每天执行一次logrotate程序。



