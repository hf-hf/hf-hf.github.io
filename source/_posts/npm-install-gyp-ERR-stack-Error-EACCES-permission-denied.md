---
title: 'npm install gyp ERR! stack Error: EACCES: permission denied'
date: 2020-05-22 15:41:04
tags:
    - nodejs
    - npm
---
![homePage](/upload/homePage/20200522160200.jpg)
<!--more-->

## 情景
之前部署Nuxt.js项目，新添加依赖后在服务器编译报错：
```
gyp ERR! stack Error: EACCES: permission denied, mkdir '/usr/local/lib/node_modules/bcrypt/.node-gyp'
```

## 解决方案
复制报错信息在网上搜了一下相关的内容有很多，解决方案大部分都是说npm install的时候添加--unsafe-perm参数，加上试了一下，难得的成功了0.0。

## 原因分析
虽然问题解决了，但是原因是什么呢？
从报错内容一眼应该就能看出来，提示permission denied那么肯定是因为创建后面的目录没有权限，但是为什么会没有权限呢？
为了找到根本原因我去官网查了查，npmjs官方文档中对于[unsafe-perm](https://docs.npmjs.com/misc/config#unsafe-perm)的说明见下图：
![npm-install-permission-denied_1](/upload/npm-install-permission-denied/1.png)

从文档中可以看出unsafe-perm如果是在root用户下默认值是false，否则为true，其作用是unsafe-perm设置为true可以在运行包脚本时抑制UID/GID切换，如果显式设置为false，则使用非root用户安装将会失败。

> 什么是UID和GID？
> 系统管理器授权每个进场使用一个给定的UID标识符(USER Ientification)。每个被启动的进程都有一个启动该进程的的用户UID。子进程拥有和父进程一样的UID。用户可以是某个组的成员，每个组也有一个GID标识(Group Identification)。

因为我是在root下操作的，所以在我运行npm install时unsafe-perm默认值为false，也就是不会抑制UID/GID切换，既然这里的报错是permission denied，也就是说我的操作在执行到安装node-gyp之前当前的用户肯定是发生了切换，它应该是被切换到了一个无权限的用户，此时再执行安装node-gyp就报出了permission denied的错误。

分析到这里，我现在怀疑这是node-gyp的一个问题，于是我到Github上搜了一下issues，果然找到了一个[node-gyp-issues-454](https://github.com/nodejs/node-gyp/issues/454)，大致看了一下，原因应该就是我是使用root用户执行的命令，你如果使用其他用户加sudo也是一样的意思，issues中的一些评论见下图。

![npm-install-permission-denied_2](/upload/npm-install-permission-denied/2.png)

综合来说，就是这个问题我们需要设置unsafe-perm=true，用来阻碍在root或者sudo下来运行npm命令时导致的UID/GID切换，而且从这个issue的时间来看，项目维护者不太关心这个问题，或者说他们认为使用root或者sudo执行npm install本身就不是合规的操作。

我又去stackoverflow搜了一下，[这篇文章](https://stackoverflow.com/questions/45734002/error-when-trying-to-install-firebase-with-npm)中解释了npm切换用户的原因如下：

```
If npm detects it is running as root it drops to a non-privileged user which then doesn't have permissions to write to /root/.node-gyp. The --unsafe-perm option stops it from changing user.
# 如果npm检测到它运行在root用户下，那么它将会降级到一个非特权用户（non-privileged user），这个用户没有写入/root/.node-gyp的权限，--unsafe-perm参数会阻止它切换用户。

nvm doesn't have this problem when not using sudo because it stores everything under the current users' home directory.
# nvm在不使用sudo时不会有这个问题，因为它将所有内容存储在当前用户的home目录。
```

另外在npmjs官方文档中我也找到了所谓的非特权用户（non-privileged user）到底是什么，见其配置[user](https://docs.npmjs.com/misc/config#user)：

![npm-install-permission-denied_3](/upload/npm-install-permission-denied/3.png)

也就是如果包脚本运行在root下，那么UID就会被设置为user，而user的默认值为nobody。

## 总结
到这里问题的原因和解决方案都很明显了，我们应该使用非root用户来执行npm命令，如果不得不使用root用户执行，可以加上--unsafe-perm参数，完整命令见下方。

```
npm install -g --unsafe-perm xxx
```

如果不加那么在npm运行期间current user会自动切换到npm user配置的用户（default:nobody），导致某些操作没有权限，并最终导致本次安装失败。