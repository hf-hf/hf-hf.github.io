---
title: Linux too many open files问题调查
date: 2018-06-05 14:38:14
tags:
    - linux
    - too many open files
---

## 问题

今天发现某netty服务总是报too many open files，因为是新上的服务，设备量并不多，不应该啊，T.T

## 原因

调查一下发生原因：
too many open files，字面意思上看就是打开了过多的文件，不过在linux下files并不只是文件的意思，像socket(Tcp、Udp)连接，开启监听的端口这些都属于file，所以有时候也可以叫做句柄(handle)，这个错误也可以描述为句柄数（file-handles）超出系统限制。 

从上述文字描述可以看出，出现这句提示的原因就是程序打开的Socket连接数量超过系统设定值。

既然是超过了系统的设定值，那么当前用户的设定值是多少呢？

用命令查一下：
```
[root@localhost ~]# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 31203
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 31203
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

或者

```
[root@localhost ~]# ulimit -n
1024
```

当前用户open files居然才只有1024，查看一下当前系统已开启的文件数

```
[root@localhost ~]# lsof | wc -l
-bash: lsof: command not found
```

0.0，没有安装lsof,yum安装一下

```
[root@localhost ~]# yum install lsof
Loaded plugins: fastestmirror
base                                                     | 3.6 kB     00:00
epel/x86_64/metalink                                     | 6.7 kB     00:00
epel                                                     | 3.2 kB     00:00
extras                                                   | 3.4 kB     00:00
updates                                                  | 3.4 kB     00:00
(1/3): epel/x86_64/updateinfo                              | 932 kB   00:00
(2/3): updates/7/x86_64/primary_db                         | 2.0 MB   00:01
(3/3): epel/x86_64/primary                                 | 3.5 MB   00:01
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * epel: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
epel                                                                12585/12585
Resolving Dependencies
--> Running transaction check
---> Package lsof.x86_64 0:4.87-5.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package         Arch              Version                Repository       Size
================================================================================
Installing:
 lsof            x86_64            4.87-5.el7             base            331 k

Transaction Summary
================================================================================
Install  1 Package

Total download size: 331 k
Installed size: 927 k
Is this ok [y/d/N]: y
Downloading packages:
lsof-4.87-5.el7.x86_64.rpm                                 | 331 kB   00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : lsof-4.87-5.el7.x86_64                                       1/1
  Verifying  : lsof-4.87-5.el7.x86_64                                       1/1

Installed:
  lsof.x86_64 0:4.87-5.el7

Complete!
```

再查看该进程已打开的文件数

```
[root@localhost ~]# lsof -p pid | wc -l
2858
```

## 解决方案

临时提高一下open files数值

```
ulimit -n 65535
```

再查看发现open files已更新，但是这种一重启就会还原

永久更改open files的方法

```
vim /etc/security/limits.conf  
# insert last line
# * 标识所有用户，也可单独设置某一用户，如test soft nofile 65535
* soft nofile 65535  
* hard nofile 65535  
```

## 深度调查

使用命令打印程序open files详细日志，进行问题排查。

```
[root@localhost /]# lsof -p pid > openfiles.log
[root@localhost /]# tail -f ./openfiles.log

java    5275 root 2864u     IPv6           23540972       0t0       TCP localhost.localdomain:45214->192.168.0.199:19092 (ESTABLISHED)
java    5275 root 2865u     IPv6           23540973       0t0       TCP localhost.localdomain:45216->192.168.0.199:19092 (ESTABLISHED)
java    5275 root 2866u  a_inode                0,9         0      5815 [eventpoll]
java    5275 root 2867u     IPv6           23541135       0t0       TCP localhost.localdomain:45398->192.168.0.199:19092 (ESTABLISHED)
java    5275 root 2868u     IPv6           23541136       0t0       TCP localhost.localdomain:45401->192.168.0.199:19092 (ESTABLISHED)
java    5275 root 2869r     FIFO                0,8       0t0  23541164 pipe
java    5275 root 2870w     FIFO                0,8       0t0  23541164 pipe
java    5275 root 2871u  a_inode                0,9         0      5815 [eventpoll]
java    5275 root 2872u     IPv6           23541165       0t0       TCP localhost.localdomain:45433->192.168.0.199:19092 (ESTABLISHED)
java    5275 root 2873u     IPv6           23541166       0t0       TCP localhost.localdomain:45434->192.168.0.199:19092 (ESTABLISHED)
....
```

发现有很多连接处在ESTABLISHED（通讯中）状态，应该是硬件频繁连接，netty持有的channel并没有被释放。

## 程序解决方案

服务添加idle处理，硬件心跳中断一段时间后自动将channel关闭。

## TCP端口状态说明

### 1、LISTENING状态
FTP服务启动后首先处于侦听（LISTENING）状态。

### 2、ESTABLISHED状态
ESTABLISHED的意思是建立连接。表示两台机器正在通信。

### 3、CLOSE_WAIT
对方主动关闭连接或者网络异常导致连接中断，这时我方的状态会变成CLOSE_WAIT 此时我方要调用close()来使得连接正确关闭。
    
### 4、TIME_WAIT
我方主动调用close()断开连接，收到对方确认后状态变为TIME_WAIT。TCP协议规定TIME_WAIT状态会一直持续2MSL(即两倍的分段最大生存期)，以此来确保旧的连接状态不会对新连接产生影响。处于TIME_WAIT状态的连接占用的资源不会被内核释放，所以作为服务器，在可能的情况下，尽量不要主动断开连接，以减少TIME_WAIT状态造成的资源浪费。目前有一种避免TIME_WAIT资源浪费的方法，就是关闭socket的LINGER选项。但这种做法是TCP协议不推荐使用的，在某些情况下这个操作可能会带来错误。

### 5、SYN_SENT状态
SYN_SENT状态表示请求连接，当你要访问其它的计算机的服务时首先要发个同步信号给该端口，此时状态为SYN_SENT，如果连接成功了就变为 ESTABLISHED，此时SYN_SENT状态非常短暂。但如果发现SYN_SENT非常多且在向不同的机器发出，那你的机器可能中了冲击波或震荡波之类的病毒了。这类病毒为了感染别的计算机，它就要扫描别的计算机，在扫描的过程中对每个要扫描的计算机都要发出了同步请求，这也是出现许多SYN_SENT的原因。
    
根据TCP协议定义的3次握手断开连接规定,发起Socket主动关闭的一方Socket将进入TIME_WAIT状态，TIME_WAIT状态将持续2个MSL(Max Segment Lifetime)，在Windows下默认为4分钟,即240秒，TIME_WAIT状态下的socket不能被回收使用。具体现象是对于一个处理大量短连接的服务器，如果是由服务器主动关闭客户端的连接,将导致服务器端存在大量的处于TIME_WAIT状态的socket，甚至比处于Established状态下的socket多的多，严重影响服务器的处理能力,甚至耗尽可用的socket，停止服务。TIME_WAIT是TCP协议用以保证被重新分配的socket不会受到之前残留的延迟重发报文影响的机制，是必要的逻辑保证。


