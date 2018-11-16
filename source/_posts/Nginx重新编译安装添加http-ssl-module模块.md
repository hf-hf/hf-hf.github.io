---
title: Nginx编译安装添加http_ssl_module模块
date: 2018-11-16 14:12:20
tags:
    - nginx
---
![homePage](/upload/homePage/20181116144902.jpg)
<!--more-->

## 情景
今天给服务器nginx开启ssl时，发现配置文件添加ssl后提示如下信息：

```
nginx: [emerg] unknown directive "ssl" in /usr/local/nginx/conf/localhost.conf:17
```

不能识别ssl指令，看来是没有安装http_ssl_module模块。

## 解决方案
添加http_ssl_module模块，重新编译nginx。

找到原来安装nginx时的源码包，如果已经删除了使用nginx -V查看版本后再重新下载。

进入源码包目录，输入如下指令重新编译：
```
[root@instance-zgdnfxxv soft]# ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
make
```

之后将运行中的nginx关闭，替换原来的可执行文件。
<font color=red>PS：记得先备份一下之前的</font>
```
[root@instance-zgdnfxxv soft]# /usr/local/nginx/sbin/nginx stop
[root@instance-zgdnfxxv soft]# cp objs/nginx /usr/local/nginx/sbin/
cp: overwrite `/usr/local/nginx/sbin/nginx'? yes
```

再查看nginx的模块，是否已经把http_ssl_module模块编译进去了。
```
[root@instance-zgdnfxxv soft]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.9.9
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-23) (GCC)
built with OpenSSL 1.0.1e-fips 11 Feb 2013
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
```

可以看到http_ssl_module模块已经加上了，重新启动nginx即可。
