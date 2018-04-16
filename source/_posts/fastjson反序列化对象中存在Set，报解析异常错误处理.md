---
title: fastjson反序列化对象中存在Set，报解析异常错误处理
date: 2018-04-16 11:10:25
tags:
    - fastjson
    - Set序列化
---

## 情景：
使用fastjson替换jackson，作为json解析器，某对象序列化到Redis中后格式为：

``` bash
{
	"@type": "com.diditech.iov.alarm.domain.AlarmChartQueryParam",
	"almTypeSet": Set[1, 2, 3],
	"almYear": "2018",
	"chartType": 1,
	"defaultFlag": 1,
	"type": 2
}
``` 

其中"almTypeSet": Set[1, 2, 3]，字段类型为Set<Integer>，反序列时报错，如下图：
![fastjson-serializable1.png](/upload/fastjson/fastjson-serializable1.png)

将字段类型更换为List<Integer>后，问题暂时解决，但是为什么Set反序列化会出现问题？

比对List和Set生成的Json数据：
List<Integer>   ------------->    [1, 2, 3]
Set<Integer>    ------------->    Set[1, 2, 3]

Set类型生成的json数据中写入了Class的类型，因在使用fastjson替换原Spring Boot默认的jackson，实现了Redis序列化的接口，重写了序列化和反序列化的方法，其中有这么一句，JSON.toJSONBytes(o, SerializerFeature.WriteClassName)，SerializerFeature.WriteClassName表示序列化时会写入类名，完整代码如下：

``` bash
public class FastJsonRedisSerializer implements RedisSerializer<Object> {
    @Override
    public byte[] serialize(Object o) throws SerializationException {
        if (o == null) return null;
        return JSON.toJSONBytes(o, SerializerFeature.WriteClassName);
    }

    @Override
    public Object deserialize(byte[] bytes) throws SerializationException {
        if (bytes == null) return null;
        return JSON.parse(bytes);
    }
}
``` 

但是依然不清楚为何Set不能反序列化，而List就可以，继续追查后找到解决方案，如下：

在GitHub fastjson Issues #386，发现有人提过相同的问题

``` bash
ParserConfig.getGlobalInstance().setAsmEnable(false);//关闭asm.
``` 

关闭asm后，再进行反序列化就没有问题了。

asm：获取java bean的属性值，需要调用反射，fastjson引入了asm的来避免反射导致的开销。fastjson内置的asm是基于objectweb asm 3.3.1改造的，只保留必要的部分，fastjson asm部分不到1000行代码，引入了asm的同时不导致大小变大太多。

未待完续，asm待深入了解。。。



