---
title: Vue 2.0 Emitted value instead of an instance of Error
date: 2018-11-05 13:33:34
tags:
    - vue
---
![homePage](/upload/homePage/20181105161101.jpg)
<!--more-->

## 情景
新项目中使用Vue2.0在如下代码中，提示warning：Emitted value instead of an instance of Error。

```
<mt-cell-swipe v-for="row in data" :title="row.name" :label="row.title">
    <!-- 省略 -->
</mt-cell-swipe>
```

虽然是警告，并不影响运行，但是看着每次都输出这一串也是比较烦，索性彻底解决一下。

## 原因
在Vue2.0中，v-for相对之前版本修改了语法，丢弃了$index和$key，新语法循环数组代码如下。

```
<mt-cell-swipe v-for="(row, index) in data" :key="index" :title="row.name" :label="row.title">
    <!-- 省略 -->
</mt-cell-swipe>

<!-- 或者 -->

<mt-cell-swipe v-for="row in data" :key="row.id" :title="row.name" :label="row.title">
    <!-- 省略 -->
</mt-cell-swipe>
```

其中key是不可省略的，否则就会出现该警告。

更新详情见：[Vue 2.0 的变化（二）之其他重大更改](https://segmentfault.com/a/1190000007018605)