---
title: Spring Boot临时文件jar_cache.tmp堆积，填满磁盘空间不释放问题
date: 2018-09-07 11:41:50
tags:
    - Spring Boot
---
![homePage](/upload/homePage/20180907140001.jpg)
<!--more-->

## 情景
前段时间发现的Spring Boot在/tmp目录下有生成很多的jar_cache.tmp临时文件，使用lsof查询发现更多的临时文件已经被删除，但是仍然占据磁盘空间不释放，导致服务器磁盘被填满。

```
[root@iZm5e38a6dvhdwwr77l6hdZ ~]# df -m
Filesystem     1M-blocks  Used Available Use% Mounted on
/dev/vda1          40188 29211      8913  77% /
devtmpfs            3903     3      3900   1% /dev
tmpfs               3912     0      3912   0% /dev/shm
tmpfs               3912     1      3911   1% /run
tmpfs               3912     0      3912   0% /sys/fs/cgroup
tmpfs                783     0       783   0% /run/user/0
[root@iZm5e38a6dvhdwwr77l6hdZ ~]# lsof |grep deleted
java       9736  9828    root 2258r      REG   253,1   4321205    170  9998 /alarmTmp/jar_cache6752106489938374687.tmp (deleted)
java       9736  9828    root 2259r      REG   253,1   4321205    170  9999 /alarmTmp/jar_cache1207708198438224698.tmp (deleted)
java       9736  9828    root 2260r      REG   253,1   4321205    171  0000 /alarmTmp/jar_cache1489322590599279349.tmp (deleted)
java       9736  9828    root 2261r      REG   253,1   4321205    171  0001 /alarmTmp/jar_cache5743614387643801711.tmp (deleted)
java       9736  9828    root 2262r      REG   253,1   4321205    171  0002 /alarmTmp/jar_cache8711237805467029629.tmp (deleted)
java       9736  9828    root 2263r      REG   253,1   4321205    171  0033 /alarmTmp/jar_cache7144840203531988105.tmp (deleted)
...
```

当时使用了临时的方案，使用-Djava.io.tmpdir=/alarmTmp将生成的临时文件单独存放到了一个目录，并定时重启释放这部分磁盘空间。

## 原因调查
今天抽出空来继续调查这个问题，最后在Github上定位到了具体的原因。原来是Spring boot v1.4.5 ~ v2.1.0.M2之间的版本，由于ServletContext获取静态资源的行为发生了改变，导致在Spring Boot临时目录生成的很多jar_cache.tmp文件，它们是被打开的并且已删除的文件，只能通过lsof才能查看，这些文件直到应用程序退出才会被释放，所以它们会在程序运行期间一直占据空间并填满磁盘。

[Retrieving static resource via ServletContext root creates (many) jar_cache tmp files #9866](https://github.com/spring-projects/spring-boot/issues/9866)
   
## 解决方案
1、升级/降级Spring Boot版本，使用低于v1.4.4或者高于v2.1.0.M2的版本。
2、使用spring.resources.static-locations，定制静态资源位置，并将classpath:/META-INF/resources/放置到首位。

解决后再查看磁盘，已恢复到正常状态。
```
[root@iZm5e38a6dvhdwwr77l6hdZ ~]# df -m
Filesystem     1M-blocks  Used Available Use% Mounted on
/dev/vda1          40188  8286     29838  22% /
devtmpfs            3903     8      3896   1% /dev
tmpfs               3912     0      3912   0% /dev/shm
tmpfs               3912     1      3911   1% /run
tmpfs               3912     0      3912   0% /sys/fs/cgroup
tmpfs                783     0       783   0% /run/user/0
```



