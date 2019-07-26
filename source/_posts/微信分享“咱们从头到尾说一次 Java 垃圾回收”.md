---
title: 微信分享“咱们从头到尾说一次 Java 垃圾回收”
date: 2019-07-26 09:46:39
tags:
    - jvm
categories: 微信分享
---
![homePage](/upload/homePage/20190726104410.jpg)
<!--more-->

> 名词解释：
> GC：垃圾收集器
> Minor GC：新生代GC，指发生在新生代的垃圾收集动作，所有的Minor GC都会触发全世界的暂停（stop-the-world），停止应用程序的线程，不过这个过程非常短暂。
> Major GC/Full GC：老年代GC，指发生在老年代的GC。
> JVM：Java Virtual Machine（Java虚拟机）的缩写。

[咱们从头到尾说一次 Java 垃圾回收](https://mp.weixin.qq.com/s?__biz=MjM5NDkxMTgyNw==&mid=2653061721&idx=1&sn=529836b657dfe9a4772edf161d24f29a&chksm=bd56a1658a212873691e11ce7b3e5f47dac13d5908685b64238c2f631f646abc5eb95817ef0a&mpshare=1&scene=23&srcid=0726RiU1Sqcn0f8DbZEzTNjm&sharer_sharetime=1564092234256&sharer_shareid=f4ea5dcd1ac851d940183676dc6dac96#rd)
