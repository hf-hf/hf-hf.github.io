---
title: Spring Boot加载配置，配置文件位置
date: 2018-08-29 10:19:19
tags:
    - Spring Boot
---

## 情景
一个Spring Boot项目同时部署3个实例，使用阿里负载均衡这三个端口，其中每个实例外部的配置文件各不相同，配合jenkins自动部署需要编写Shell脚本。

初版编写的Shell脚本内容如下：
```
#!/bin/bash
echo "pulish alarm start"
cd /app/diditech-iov-unify-alarm
dir_base=/app/diditech-iov-unify-alarm
echo $dir_base
jar_name='diditech-iov-unify-alarm'
jar_prefix='.jar'
now_date=`date +%y%m%d%H%M%S`

echo 'now_date:'${now_date}
export JAVA_HOME=/usr/jdk
echo ${JAVA_HOME}
for dir in $(ls)
do
  if [ -d $dir ]
then
   echo $dir #先判断是否是目录，然后再输出
   children_dir_base=${dir_base}/$dir
   echo ${children_dir_base}
   # 重命名原文件
   mv ${children_dir_base}/${jar_name}${jar_prefix} ${children_dir_base}/${jar_name}-${now_date}${jar_prefix}
   cp -r $dir_base/${jar_name}${jar_prefix} ${children_dir_base}/${jar_name}${jar_prefix}
   pid=`ps -ef | grep ${jar_name} | grep $dir | grep "java" | awk '{print $2}'`
   if [ -n "$pid" ]
   then
      kill -9 $pid
      echo 'kill '$pid
   fi
   sleep 5
   #cd ${children_dir_base}
   java -jar  ${children_dir_base}/${jar_name}${jar_prefix} >/dev/$dir &
   pid_tmp=`netstat -ntulp | grep $dir | awk '{print $4}'`
   echo $pid_tmp
   echo 'java -jar  '${children_dir_base}/${jar_name}${jar_prefix} ' >/dev/dir &'
   sleep 2
   cd $dir_base
   echo 'ok!'
fi
done
echo "pulish alarm success！"
```

其中注释掉了一行cd ${children_dir_base}，即每次启动Spring Boot都是在父级目录执行的java -jar。

## 问题
使用上述初版脚本启动时，发现虽然三个实例配置文件中的端口号配置的各不相同，但是使用脚本启动时，并没有读取外部的配置文件，而是使用的发布包中的配置文件，因此三个实例都启动了同一端口，导致只能起来一个实例。

## 调查原因
该问题发生原因为Spring Boot加载配置文件的位置，调查结果如下：

以下内容转载自[Spring Boot加载配置文件](https://www.cnblogs.com/moonandstar08/p/7368292.html)

1) 默认位置：
Spring Boot默认的配置文件名称为application.properties，SpringApplication将从以下位置加载application.properties文件，并把它们添加到Spring Environment中：
当前目录下的/config子目录
当前目录
一个classpath下的/config包
classpath根路径（root）
这个列表是按优先级排序的（列表中位置高的将覆盖位置低的）。

注：Spring-boot配置文件的加载，先在与jar同级下查找，如果没有就去同级的config下查找；如果再没有，就在jar包中去查找相应的配置文件，如果再没有，就去jar包中的config下去查找。当查找到对应配置片段时，采用增量替换的方式来进行替换。

2) 自定义位置：
如果不喜欢application.properties这个文件名或者需要自定义配置文件位置，在启动Spring应用的时候，可以传入一个spring.config.location参数指定配置文件位置。
例如：
java -jar xxxxx.jar   --spring.config.location=classpath:/default.properties,classpath:/override.properties
上述例子加载了两个配置文件，分别位于根目录下的：default.properties，override.properties。

## 结论
因为执行Shell脚本时，是在jar所在目录的父级目录，因此Spring Boot加载配置文件时，当前目录下的/config子目录、当前目录都无application.properties配置文件，其最终使用的为classpath根路径（即jar包内）的配置文件。

## 解决方案
修改脚本cd到jar包所在目录，再执行java -jar即可，启动时会加载当前目录下的配置文件。
