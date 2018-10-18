---
title: Idea修改import顺序，与Eclipse一致
date: 2018-06-21 18:25:50
tags:
    - idea
    - import
    - Eclipse
---

## 情景：
之前切换到Idea之后，一直在和身边的人安利，但是总有部分顽固分子（偷瞄一眼对面的同学 0.0）还在使用Eclipse，Idea和Eclipse提交同一个Java文件，会导致import部分顺序不一致，提交时进行比对时会发现很多报红的，其实只是引用顺序换了换而已。
    
## 调查：
上网搜了一下，发现Idea可以调整import的顺序，具体配置在：Settings -> code style -> java -> imports。
    
## 解决方案
调整Settings -> code style -> java -> imports下的引用顺序，改为如下：
![change_import_1.png](/upload/changeImport/change_import_1.png) 
另外Idea默认会将超过一定数量的import，自动合并成import *，修改不进行合并也是在此处，将下图中的两个数值改到999，点击保存即可。
![change_import_2.png](/upload/changeImport/change_import_2.png)   