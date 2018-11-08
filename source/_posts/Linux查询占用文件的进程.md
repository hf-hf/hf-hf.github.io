---
title: Linux查询占用文件的进程
date: 2018-09-06 15:52:41
tags:
    - linux
categories: 运维日志
---
![homePage](/upload/homePage/20180906170335.jpg)
<!--more-->
## 情景
今天查看某服务器/tmp目录下生成了很多spring.log文件，占用了很多磁盘空间，清理前需要调查下是那些程序生成并写入的这些文件，见下图。
![Linux查询占用文件的进程_1](/upload/Linux查询占用文件的进程/Linux查询占用文件的进程_1.png)

## 调查方案
查看那个进程在使用该文件
```
lsof /tmp/spring.log
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF    NODE NAME
java    4606 root    6w   REG  253,1   238739 2490940 /tmp/spring.log
```

## 临时解决方案
启动时增加参数-Djava.io.tmpdir=自定义目录，暂时将生成的临时文件单独放到一个目录，并配置脚本将程序每天进行重启。

## lsof语法说明
```
lsof -i 用以显示符合条件的进程情况
# 语法: lsof -i[46] [protocol][@hostname|hostaddr][:service|port]
```

## 扩展延伸
列出所有打开的文件
```
lsof
```

递归查看某个目录的文件信息
```
lsof +D /tmp
perfdata_root/4127
java       4606 root  mem    REG              253,1     32768   2490959 /tmp/hsperfdata_root/4606
java       4606 root    6w   REG              253,1    245683   2490940 /tmp/spring.log
java      22895 root  mem    REG              253,1     32768   2491670 /tmp/hsperfdata_root/22895
java      26036 root  mem    REG              253,1     32768   2491448 /tmp/hsperfdata_root/26036
AliYunDun 28456 root    9u  unix 0xffff880390fb6400       0t0 542580169 /tmp/Aegis-<Guid(5A2C30A2-A87D-490A-9281-6765EDAD7CBA)>
java      31611 root  mem    REG              253,1     32768   2491465 /tmp/hsperfdata_root/31611
java      32717 root  mem    REG              253,1     32768   2490938 /tmp/hsperfdata_root/32717
java      32717 root    6w   REG              253,1 123951874   2490381 /tmp/spring.log.1
```

在之前查询的基础上过滤结果集
```
lsof +D /tmp | grep 'spring.log'
java       4606 root    6w   REG              253,1    249939   2490940 /tmp/spring.log
java      32717 root    6w   REG              253,1 129286500   2490381 /tmp/spring.log.1
```

列出某个用户打开的文件信息
```
lsof -u username
# PS：-u 选项，u其实是user的缩写
```

列出某个用户以及某个程序所打开的文件信息
```
lsof -u test -c java
```

列出除了某个用户外的被打开的文件信息
```
lsof -u ^root
PS：^这个符号在用户名之前，将会把是root用户打开的进程不显示
```

列出某个程序所打开的文件信息
```
lsof -c java
# PS：-c 选项将会列出所有以java开头的程序，其查询结果与lsof | grep java相同
```

列出多个程序多打开的文件信息
```
lsof -c java -c apache
```

通过某个进程号显示该进程打开的文件
```
lsof -p 进程号
# PS：追加-r5可每5s输出内容，lsof -p 进程号 -r5 
```

列出多个进程号对应的文件信息
```
lsof -p 5217,5218,5219
```

列出除了某个进程号，其他进程所打开的文件信息
```
lsof -p ^5217
```

列出所有的网络连接
```
lsof -i
```

列出所有tcp 网络连接信息
```
lsof  -i tcp
```

列出所有udp网络连接信息
```
lsof  -i udp
```

列出谁在使用某个端口
```
lsof -i :3306
```

列出谁在使用某个特定的udp端口
```
lsof -i udp:55
```

特定的tcp端口
```
lsof -i tcp:80
```

列出某个用户的所有活跃的网络端口
```
lsof  -a -u root -i
```

列出所有网络文件系统
```
lsof -N
```

域名socket文件
```
lsof -u
```

某个用户组所打开的文件信息
```
lsof -g 5172
```

根据文件描述列出对应的文件信息
```
lsof -d description(like 2)
```

根据文件描述范围列出文件信息
```
lsof -d 2-3
```
