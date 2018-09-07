---
title: Vue调试原始代码
date: 2018-08-29 10:56:52
tags:
    - vue
---

## 情景
使用Spring Boot + Vue开发前后端分离的项目，其中前端vue使用chrome调试代码时，发现F12 -> Source中的代码不是开发时的原始代码，调试十分不方便。

## 调查
调查后发现该原因与Webpack的devtool和source maps配置有关系。

Webpack打包生成的.map后缀文件，使得我们的开发调试更加方便，它能帮助我们链接到断点对应的源代码的位置进行调试（//# souceURL），而devtool就是用来指定source-maps的配置方式的。

以下为官方文档说明：

### source maps

![source maps](/upload/vue调试原始代码/vue调试原始代码_1.png)

### 开发工具(Devtool)

此选项控制是否生成，以及如何生成 Source Map。以下是官方文档的配置选项：

![devtool](/upload/vue调试原始代码/vue调试原始代码_2.png)

devtool配置选项

其中一些值适用于开发环境（从表格中各种方式的构建速度来看，可以看出eval方式可大幅提高持续构建效率，这对经常需要边改边调的同学而言非常重要），一些适用于生产环境。对于开发环境，通常希望更快速的 Source Map，需要添加到 bundle 中以增加体积为代价，但是对于生产环境，则希望更精准的 Source Map，需要从 bundle 中分离并独立存在。

## 解决方案
修改index.js，修改内容如下：

```
// devtool: 'cheap-source-map',
devtool: 'source-map',
```

使用source-map替换原来的cheap-source-map即可。

