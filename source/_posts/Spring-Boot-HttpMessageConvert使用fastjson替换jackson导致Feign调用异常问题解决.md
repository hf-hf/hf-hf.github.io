---
title: Spring Boot HttpMessageConvert使用fastjson替换jackson导致Feign调用异常问题解决
date: 2019-01-04 14:43:49
tags:
    - Spring Boot
    - fastjson
    - Feign
---
![homePage](/upload/homePage/20190104095506.jpg)
<!--more-->

## 情景
SpringBoot项目中使用fastjson替换了原jackson的HttpMessageConverter，并配置了序列化的策略，代码如下：
```
@Bean
public HttpMessageConverter<Object> converter() {
    FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
    converter.setFeatures(
        SerializerFeature.WriteMapNullValue,
        SerializerFeature.WriteNullStringAsEmpty,
        SerializerFeature.WriteNullListAsEmpty,
        SerializerFeature.WriteNullNumberAsZero,
        SerializerFeature.WriteNullBooleanAsFalse,
        SerializerFeature.WriteDateUseDateFormat
    );
    return converter;
}
```

因这部分代码运行已有很长时间，序列化策略已具有业务意义，不能随意修改，但在最近使用Feign调用接口时，发现其中的序列化策略对于调用远端接口有影响，调用报错提示没有合适的HttpMessageConvert。

![spring_boot_encoder_1.png](/upload/Spring-Boot-Encoder/spring_boot_encoder_1.png)

## 原因分析
分析一下问题原因，首先产生该问题肯定是由于替换了SpringBoot默认的HttpMessageConvert，在报错信息中找一下，我们能够定位到这个接口：feign.codec.Encoder。Feign调用接口之前需要先通过Encoder将参数编码后再传输，接下来再看一下Encoder接口的实现类，

![spring_boot_encoder_2.png](/upload/Spring-Boot-Encoder/spring_boot_encoder_2.png)

可以看到Encoder接口有一个名为SpringEncoder的实现类，该类构造器需要传递一个ObjectFactory<HttpMessageConverters>，很明显这是一个HttpMessageConverters的工厂，是用来获取HttpMessageConverters的。再往上翻一翻代码，我们能够分析出来，feign的Encoder默认使用的Convert是SpringBoot的HttpMessageConvert，而我们之前使用fastjson的版本将其替换了，所以feign会使用我们替换后的FastJsonHttpMessageConverter，以至于调用远端接口异常。

![spring_boot_encoder_3.png](/upload/Spring-Boot-Encoder/spring_boot_encoder_3.png)

## 解决方案
这样我们如何解决这个问题就很清晰了，只需要将feign使用的Encoder中的HttpMessageConvert替换成无自定义序列化策略的版本即可，修改代码如下：

```
//可用lamda表达式简化一下代码，但是为了便于理解这里就提供了简化前的版本
@Bean
public Encoder feignEncoder() {
    ObjectFactory<HttpMessageConverters> objectFactory = new ObjectFactory<HttpMessageConverters>() {
        @Override
        public HttpMessageConverters getObject() throws BeansException {
            return new HttpMessageConverters(new FastJsonHttpMessageConverter());
        }
    };
    return new SpringEncoder(objectFactory);
}
```

这样我们提供了一个新的Encoder，这个Encoder中没有自定义的序列化策略，再试一下，调用远端接口正常了。

## Feign上传文件问题
添加以上代码修改后，普通的远程调用接口都已经正常了，但是通过Feign上传文件、图片的接口发现还是有问题，我们提供的Encoder并不能编码文件和图片，那么倒回去再看一下Encoder的实现类，其中有一个类名为SpringFormEncoder。

![spring_boot_encoder_4.png](/upload/Spring-Boot-Encoder/spring_boot_encoder_4.png)

可以看到这个类位于feign-form-spring包中，我们正常通过web页面上传文件/图片都是通用form-data，而feign也是有相关的form编码器，因此我们应该使用这个替换之前的SpringEncoder，看一下SpringFormEncoder的构造器，发现它需要一个Encoder，那么正好我们把之前构造的SpringEncoder传递进去，最终修改后代码如下:

```
@Bean
public Encoder feignFormEncoder() {
    ObjectFactory<HttpMessageConverters> objectFactory = new ObjectFactory<HttpMessageConverters>() {
        @Override
        public HttpMessageConverters getObject() throws BeansException {
            return new HttpMessageConverters(new FastJsonHttpMessageConverter());
        }
    };
    return new SpringFormEncoder(new SpringEncoder(objectFactory));
}
```

测试一下，文件上传接口调用正常，对原来的数据接口也没有影响，问题解决！
