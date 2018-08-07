---
title: Linux使用timedatectl操作时间时区
date: 2018-08-07 18:03:33
tags:
    - linux
    - timedatectl
---

## 情景
今天搭建新的测试环境，发现系统时间存在问题，使用date指令查询日期时间正确，但是程序log中打印的时间与系统时间相差8个小时。

## 原因
使用timedatectl查询系统时区，发现CST使用的是美国中部时间Central Standard Time (USA) UT-6:00，与中国标准时间相差8个小时。

## CST概念
CST（时区缩写），可视为美国、澳大利亚、古巴或中国的标准时间。

CST可以为如下4个不同的时区的缩写：
美国中部时间：Central Standard Time (USA) UT-6:00
澳大利亚中部时间：Central Standard Time (Australia) UT+9:30
中国标准时间：China Standard Time UT+8:00
古巴标准时间：Cuba Standard Time UT-4:00

## 解决方案
修改设置系统时区为上海，并将硬件时钟与本地时钟一致。
```
# 设置系统时区为上海
timedatectl set-timezone Asia/Shanghai 
# 将硬件时钟调整为与本地时钟一致
timedatectl set-local-rtc 1
```

修改后重启程序，log上时间已经与本地时间一致。

## timedatectl其他命令
```
# 查看系统时区信息
[root@localhost ~]# timedatectl
      Local time: Tue 2018-08-07 18:13:37 CST
  Universal time: Tue 2018-08-07 10:13:37 UTC
        RTC time: Tue 2018-08-07 18:13:37
       Time zone: Asia/Shanghai (CST, +0800)
     NTP enabled: n/a
NTP synchronized: no
 RTC in local TZ: yes
      DST active: n/a

# 列出所有时区
[root@localhost ~]# timedatectl list-timezones
Africa/Abidjan
Africa/Accra
Africa/Addis_Ababa
Africa/Algiers
Africa/Asmara
...
```




