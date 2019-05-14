---
title: Docker挂载宿主目录，容器内对其操作提示Permission denied
date: 2019-05-14 16:09:14
tags:
    - docker
---
![homePage](/upload/homePage/20190514163007.jpg)
<!--more-->

## 情景
Docker挂载宿主主机目录，docker run后容器启动正常，但是挂载目录中不存在容器内生成的文件，``journalctl -ex``查看容器内输出，发现报错提示'Permission denied'。

## 原因
因SELinux状态启用，导致容器内对指定目录、文件无操作权限。

> ### 什么是SELinux？
> SELinux 全称 Security Enhanced Linux (安全强化 Linux),是美国国家安全局2000年以 GNU GPL 发布，是 MAC (Mandatory Access Control，强制访问控制系统)的一个实现，目的在于明确的指明某个进程可以访问哪些资源(文件、网络端口等)。
> 强制访问控制系统的用途在于增强系统抵御 0-Day 攻击(利用尚未公开的漏洞实现的攻击行为)的能力。所以它不是网络防火墙或 ACL 的替代品，在用途上也不重复。在目前的大多数发行版中，已经默认在内核集成了SELinux。

## 解决方案
该问题可通过以下两种方式解决：
1、关闭SELinux
```
# 临时关闭
setenforce 0
# 永久关闭 
# 修改/etc/sysconfig/selinux文件，将SELINUX的值设置为disabled。
```

2、以特权方式启动容器，指定--privileged参数
```
docker run -it --privileged=true -v /config:/app/ centos /bin/bash
```

