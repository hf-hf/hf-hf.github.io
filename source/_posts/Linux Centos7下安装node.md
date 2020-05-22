---
title: Linux Centos7下安装node
date: 2020-05-22 15:14:53
tags:
    - linux
    - nodejs
    - npm
---
![homePage](/upload/homePage/20200522152500.jpg)
<!--more-->

## 下载node.js安装包并解压
```
wget https://nodejs.org/dist/v12.16.0/node-v12.16.0-linux-x64.tar
tar -xvf node-v12.16.0-linux-x64.tar
```
如需安装其他版本，请自行到[nodejs官网](https://nodejs.org/en/download/)下载。

## 添加软链接
```
ln -s /usr/bak/nodejs/node-v12.16.0-linux-x64/bin/node /usr/local/bin/node
ln -s /usr/bak/nodejs/node-v12.16.0-linux-x64/bin/npm /usr/local/bin/npm
```

## 验证
软链接添加完后，在命令行通过node -v 和 npm -v 验证，输出版本号信息即安装成功。

```
[root@xxxxx ~] node -v
v12.16.0
[root@xxxxx ~] npm -v
6.13.4
```



