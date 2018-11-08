---
title: Nginx配置忽略找不到favicon.ico文件的错误日志
date: 2018-11-08 13:52:03
tags:
    - nginx
categories: 运维日志
---
![homePage](/upload/homePage/20181108142102.jpg)
<!--more-->

## 情景
今天排查某服务请求问题，发现nginx error日志中重复输出了很多如下信息：

```
[error] 3116#0: *361394530 open() "/nginx-1.11.13/xxx/favicon.ico" failed (2: No such file or directory), client: 100.116.235.xxx, server: www.xxx.com, request: "GET /indexhome/favicon.ico HTTP/1.1", host: "www.xxx.com", referrer: "http://www.xxx.com/indexhome/list"
```

原因很明显是由于nginx找不到favicon.ico文件，但是由于没有特意放置favicon.ico文件，所以导致日志内记录大量这种错误，没有任何作用并且占用磁盘空间，影响其他问题的排查。

## 解决方案
所以我们处理一下，让nginx即使找不到这个文件，也不要打印error，在对应的server{...}内添加如下信息：
```
location ~ /favicon\.ico$ {
    log_not_found off; //默认为开启：启用或禁用404等错误日志
    access_log off; //关闭access_log，即不记录访问日志
}
```