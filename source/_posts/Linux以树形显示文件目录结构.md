---
title: Linux以树形显示文件目录结构
date: 2018-11-07 11:21:50
tags:
    - linux
categories: 运维日志
---
![homePage](/upload/homePage/20181107114302.jpg)
<!--more-->

## 情景
编写部署文档需要添加发布目录层级结构。

## 解决方案
使用linux tree命令，它可以以树形展示文件目录结构。

## tree安装
使用yum -y install tree安装。

```
[root@dd iot]# yum -y install tree
Loaded plugins: fastestmirror
base                                                     | 3.6 kB     00:00
epel/x86_64/metalink                                     |  10 kB     00:00
epel                                                     | 3.2 kB     00:00
extras                                                   | 3.4 kB     00:00
updates                                                  | 3.4 kB     00:00
(1/5): epel/x86_64/group_gz                                |  88 kB   00:00
(2/5): epel/x86_64/updateinfo                              | 934 kB   00:00
(3/5): epel/x86_64/primary                                 | 3.6 MB   00:00
(4/5): extras/7/x86_64/primary_db                          | 204 kB   00:00
(5/5): updates/7/x86_64/primary_db                         | 6.0 MB   00:00
Determining fastest mirrors
 * base: mirrors.tuna.tsinghua.edu.cn
 * epel: mirrors.tuna.tsinghua.edu.cn
 * extras: mirror.bit.edu.cn
 * updates: mirror.bit.edu.cn
epel                                                                12743/12743
Resolving Dependencies
--> Running transaction check
---> Package tree.x86_64 0:1.6.0-10.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package        Arch             Version                   Repository      Size
================================================================================
Installing:
 tree           x86_64           1.6.0-10.el7              base            46 k

Transaction Summary
================================================================================
Install  1 Package

Total download size: 46 k
Installed size: 87 k
Downloading packages:
tree-1.6.0-10.el7.x86_64.rpm                               |  46 kB   00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
  Installing : tree-1.6.0-10.el7.x86_64                                     1/1
  Verifying  : tree-1.6.0-10.el7.x86_64                                     1/1

Installed:
  tree.x86_64 0:1.6.0-10.el7

Complete!
```

## tree使用
输出当前目录结构，深度为3。
```
[root@dd iot]# tree -L 3
.
├── application-prod.properties
├── application.properties
├── config
│   ├── log4j
│   ├── north
│   │   ├── deviceAnswer-1.0.0.jar
│   │   ├── device-answer.xml
│   │   ├── deviceParse-1.0.0.jar
│   │   └── device-parse.xml
│   └── properties
│       └── plugins.properties
├── Finished
├── framework-1.0.0.jar
├── logback-spring.xml
├── Package
├── restart-jenkins.sh
├── restart.sh
├── Running
└── test

4 directories, 15 files
```

将目录结构输出到文件中。
```
[root@dd iot]# tree -L 3 >>  test
[root@dd iot]# cat test
//...省略，输出结果与上方内容相同
```