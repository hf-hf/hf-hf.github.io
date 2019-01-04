---
title: 分享Idea Maven Helper插件，快捷解决依赖冲突问题
date: 2019-01-04 11:14:45
tags:
    - Idea
    - Maven
    - Maven Helper
---
![homePage](/upload/homePage/20190104095504.jpg)
<!--more-->

## 情景
在修改一些旧Maven项目时，我们引入新的依赖，有时候会提示冲突，Maven Helper能够快捷的帮我们找到相同依赖包所有引用的版本，能够有效的解决依赖冲突、减少重复性的引用。

## 安装说明
在Idea中点击Ctrl + Alt + S或者选择File -> Settings ->Plugins，选择Browse Repositories，在搜索框中输入Maven Helper，安装结果项的第一个即可。

![maven_helper_1](/upload/mavenHelper/maven_helper_1.png)

## 解决冲突
安装完插件后，在项目中选择pom.xml，可以看到在界面中多出了一个Tag页：Dependency Analyzer，选择该界面可以看到如下界面：

![maven_helper_2](/upload/mavenHelper/maven_helper_2.png)

选择Conflicts，我们可以看到当前项目中存在的冲突的依赖，选择任意一项，可以看到当前依赖的所有版本以及是否为项目直接引用等信息。若想解决依赖冲突，只需在左侧列表中选中你想解决的依赖名称，在其右侧列表中，选择想要排除的版本（不管是否为项目直接引用），右击选择Exclude即可，Maven Helper会自动添加exclusion，帮助你排除选中的依赖包。

![maven_helper_3](/upload/mavenHelper/maven_helper_3.png)

点击Exclude后，pom.xml中自动添加的exclusion。

![maven_helper_4](/upload/mavenHelper/maven_helper_4.png)
