---
title: Nginx Map模块
date: 2018-08-29 10:00:25
tags:
    - nginx
---

## 情景
搭建ele、美团一键红包程序时，其nginx配置文件中有如下内容：
```
map $http_origin $corsHost {
    default 0;
    "~https://www.mtdhb.com" https://www.mtdhb.com;
    "~http://localhost:4001" http://localhost:4001;
    "~http://127.0.0.1:4001" http://127.0.0.1:4001;
}
```
从字面上来看，就是nginx配置的跨域处理，每次请求动态赋值origin。

## Map使用调查
Map在这里意思为“映射”，即将$http_origin和$corsHost建立映射关系。
情景中的例子，其映射规则如下：
1) 花括号中第一行default 0，是一个特殊的匹配条件，当其他条件都不满足时，$corsHost会映射为0。
2) 第二行"~https://www.mtdhb.com" https://www.mtdhb.com;，该行表示若$http_origin以https://www.mtdhb.com开头，则将$corsHost映射为$corsHost。
3) 第三、四行与第二行映射方式相同。

该段映射规则以Java代码描述如下：
```
if($http_origin.startsWith("https://www.mtdhb.com")){
    $corsHost = https://www.mtdhb.com;
} else if($http_origin.startsWith("http://localhost:4001")){
    $corsHost = http://localhost:4001;
} else if($http_origin.startsWith("http://127.0.0.1:4001")){
    $corsHost = http://127.0.0.1:4001;
} else{
    $corsHost = 0;
}
```



