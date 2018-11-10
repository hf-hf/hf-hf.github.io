---
title: Spring Boot Logback在/tmp目录下生成spring.log文件问题
date: 2018-11-10 21:13:47
tags:
    - Spring Boot
categories: 运维日志
---
![homePage](/upload/homePage/20181110231002.jpg)
<!--more-->

## 情景
在服务器/tmp目录会生成spring.log文件，占用系统磁盘资源，手动删除后因程序仍占用该文件，必须重启服务后这部分空间才会得到释放，spring.log文件中的内容为相应Spring Boot服务的debug日志。

```
[root@iZm5eiwj8z018g7vbitl2sZ tmp]# ls
spring.log
spring.log.1
spring.log.2
spring.log.3
spring.log.4
spring.log.5
spring.log.6
spring.log.7
...
```

## 原因
项目使用的日志框架为logback，并在resource下配置了logback-spring.xml，在logback配置的日志目录是会正常生成日志的，并且每天自动切分，/tmp/spring.log下的日志就纯属是多余的，那为什么会打印这部分多余的日志呢？

首先因为使用的logback是Spring Boot logging包中自带的，所以可以基本排除是jar包版本问题，那么我们对其进行配置的唯一途径只有配置文件：logback-spring.xml，其内容如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
	<include resource="org/springframework/boot/logging/logback/base.xml" />
	<root level="INFO">
		<appender-ref ref="CONSOLE" />
	</root>
	<!-- INFO级别日志 -->
	<appender name="infoAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
	    <file>/kindle-manager-logs/com/kindle/quartz/info.log</file>
	    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
	        <fileNamePattern>/kindle-manager-logs/com/kindle/quartz/info-%d{yyyy-MM-dd}.log</fileNamePattern>
	    </rollingPolicy>  
	    <encoder>  
	        <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{40} - %msg%n</pattern>  
	    </encoder>
	    <filter class="ch.qos.logback.classic.filter.LevelFilter"><!-- 只打印Info日志 -->  
            <level>INFO</level>  
            <onMatch>ACCEPT</onMatch>  
            <onMismatch>DENY</onMismatch>  
        </filter>
	</appender>
	<!-- DEBUG级别日志 -->
	<appender name="debugAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">  
	    <file>/kindle-manager-logs/com/kindle/quartz/debug.log</file>
	    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
	        <fileNamePattern>/kindle-manager-logs/com/kindle/quartz/debug%d{yyyy-MM-dd}.log</fileNamePattern>
	    </rollingPolicy>  
	    <encoder>  
	        <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{40} - %msg%n</pattern>  
	    </encoder>
	    <filter class="ch.qos.logback.classic.filter.LevelFilter"><!-- 只打印debug日志 -->  
            <level>DEBUG</level>  
            <onMatch>ACCEPT</onMatch>  
            <onMismatch>DENY</onMismatch>  
        </filter>
	</appender>
	<!-- ERROR级别日志 -->
	<appender name="errorAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">  
        <file>/kindle-manager-logs/com/kindle/quartz/error.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
            <fileNamePattern>/kindle-manager-logs/com/kindle/quartz/error-%d{yyyy-MM-dd}.log</fileNamePattern>
        </rollingPolicy>  
        <encoder>  
            <pattern>%d{HH:mm:ss.SSS} %thread %X{invokeNo} %logger{40} %msg%n</pattern>  
        </encoder>  
        <filter class="ch.qos.logback.classic.filter.LevelFilter"><!-- 只打印错误日志 -->  
            <level>ERROR</level>  
            <onMatch>ACCEPT</onMatch>  
            <onMismatch>DENY</onMismatch>  
        </filter>
    </appender>
    <!-- JAVA程序日志 -->
	<logger name="com.kindle" level="DEBUG" >
		<appender-ref ref="infoAppender" />
		<appender-ref ref="debugAppender" />
		<appender-ref ref="errorAppender" />
	</logger>
	<!-- dev,test环境下日志 -->
	<springProfile name="dev,test">
		<logger name="com.kindle" level="DEBUG" />
	</springProfile>
	<!-- prod环境下日志 -->
	<springProfile name="prod">
		<logger name="com.kindle" level="INFO" />
	</springProfile>
</configuration>
```

在网上搜索了一下，发现是很常见的logback配置，在各种博客中都有相似的配置内容。

没办法，单纯从配置上并不能看出问题所在，我们只有查一下源码中打印该日志的位置，在IDE中搜索一下spring.log，结果如下:

![Spring-Boot-Logback_1.png](/upload/homePage/Spring-Boot-Logback/Spring-Boot-Logback_1.png)

可以看到仅在base.xml和LogFile的toString方法中有spring.log。

![Spring-Boot-Logback_2.png](/upload/Spring-Boot-Logback/Spring-Boot-Logback_2.png)

通过base.xml第9行，我们可以看到应该就是这个配置文件导致了spring.log的生成。

再继续搜索一下哪里调用的LogFile，LogFile调用的地方就比较多了，我们每个都点开看一下，最后在下图的类中发现了关键的信息。

![Spring-Boot-Logback_3.png](/upload/Spring-Boot-Logback/Spring-Boot-Logback_3.png)

在上图中可以看到有一个类名为DefaultLogbackConfiguration，在该类第85行，有调用LogFile的toString方法。

那么我们怀疑可能就是这个类追加的spring.log文件内容，再看一下该类的注释，在其注释上有一行信息，内容如下：

![Spring-Boot-Logback_4.png](/upload/Spring-Boot-Logback/Spring-Boot-Logback_4.png)

查看一下注释上的这个配置文件file-appender.xml的内容。

![Spring-Boot-Logback_5.png](/upload/Spring-Boot-Logback/Spring-Boot-Logback_5.png)

调查到这里我们能够确定就是这个file-appender.xml，追加的spring.log文件内容。

## 解决方案
我们分析之后，确定了spring.log的生成和内容追加的代码位置，那么我们只要相应屏蔽这两部分的代码，那么就可以解决日志重复生成的问题了。

base.xml是在logback-spring.xml第一行引用的，我们将其内容拷出，注释其引用，并将原配置中的File相关标签移除，最终修改后配置如下：

```
<configuration>
	<!--<include resource="org/springframework/boot/logging/logback/base.xml" />-->
	<include resource="org/springframework/boot/logging/logback/defaults.xml" />
	<include resource="org/springframework/boot/logging/logback/console-appender.xml" />
	<root level="INFO">
		<appender-ref ref="CONSOLE" />
	</root>
	...
</configuration>	
```

## 结论
修改后，替换测试环境logback日志配置文件，重启服务后，spring.log没有重新生成，确认问题解决。




