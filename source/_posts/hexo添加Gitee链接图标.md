---
title: hexo添加Gitee链接图标
date: 2019-03-25 10:30:54
tags:
    - hexo
    - gitee
categories: 
	- hexo
---
![homePage](/upload/homePage/20190325103507.jpg)
<!--more-->

## 主题配置
修改themes\next\\_config.yml，添加如下数据：
```
social:
    Gitee: https://gitee.com/hf-hf || gitee
```

## 自定义图标配置
因hexo所使用的图标库Font Awesome中没有gitee的图标，因此我们需要手动添加CSS，修改自定义样式文件themes\next\source\css\\_custom\custom.styl，添加如下数据：
```
.fa-gitee{
    background-image: url(/images/gitee.png);
    width: 13.7px;
    height: 16px;
    background-size: cover;
    background-repeat: no-repeat;
    background-position: center;
    transform: translateY(20%);
}
```

并将下方图标添加到source/images/目录下，命名为gitee.png。

![gitee](/images/gitee.png)

重新部署发布hexo即可。

## 最终效果
![gitee_icon_1](/upload/giteeIcon/gitee_icon_1.png)

你也可以以这种形式添加自己的网站图标^^。
