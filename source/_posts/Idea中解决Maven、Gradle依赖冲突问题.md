---
title: Idea中解决Maven、Gradle依赖冲突问题
date: 2018-10-17 15:47:46
tags:
    - idea
    - maven
    - gradle
---
![homePage](/upload/homePage/20181017164415.jpg)
<!--more-->

## 情景
今天迁移项目，代码都迁移完了，最后启动时报如下错误：

```
Detected both log4j-over-slf4j.jar AND slf4j-log4j12.jar on the class path, preempting StackOverflow
```

显而易见，这是由于jar包冲突，log4j-over-slf4j.jar和slf4j-log4j12.jar都class目录导致的。

## Maven解决方案
在idea中右侧菜单选择Maven Projects，点击Show Dependencies，之后将展开当前项目的依赖关系图。

![Idea中解决Maven、Gradle依赖冲突问题_1](/upload/idea中解决Maven、Gradle依赖冲突问题/Idea中解决Maven、Gradle依赖冲突问题_1.png)

在如下依赖关系中，查询冲突jar包所在的dependency。

![Idea中解决Maven、Gradle依赖冲突问题_2](/upload/idea中解决Maven、Gradle依赖冲突问题/Idea中解决Maven、Gradle依赖冲突问题_2.png)

修改dependency，排除不必要的冲突资源。

```
<dependency>
<groupId>org.apache.kafka</groupId>
<artifactId>kafka_2.11</artifactId>
<version>1.0.1</version>
<exclusions>
    <exclusion>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
    </exclusion>
</exclusions>
```

## Gradle解决方案
在Gradle的项目中，查看依赖关系与Maven有一点不同，在idea中右侧菜单选择Gradle，展开需要显示依赖的项目，点击Tasks -> help -> dependencies。

![Idea中解决Maven、Gradle依赖冲突问题_3](/upload/idea中解决Maven、Gradle依赖冲突问题/Idea中解决Maven、Gradle依赖冲突问题_3.png)

之后在下方Run中会显示选中项目的依赖关系，若显示信息与上图不符，则需要点击一下上图左侧圈出的Toggle tasks executions/text mode，之后在依赖关系中搜索冲突jar包即可，排除不必要的冲突资源。

```
compile (group: 'org.apache.kafka', name:'kafka_2.11', version: '1.0.1'){
    exclude(module: 'slf4j-log4j12')
}
```