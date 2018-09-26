---
title: Spring Cloud Feign调用远端ModelAttribute接口问题解决
date: 2018-09-25 15:20:00
tags:
    - Spring Cloud
    - Feign
---

![homePage](/upload/homePage/20180925152201.jpg)
<!--more-->

## 情景

公司有个项目从SpringMVC往Spring Cloud演进，原来有部分接口请求方式为Get，使用@ModelAttribute接收请求参数。

SpringMVC远端接口：

```
@RequestMapping(value = "/api/test", method = RequestMethod.GET)
public ResponseMessage test(@ModelAttribute AlarmPlanQueryParam queryParam){
    //...
}
```

Spring Cloud Feign接口：

```
@FeignClient(name = "service-webapi", fallback = PlatformApiFeignClientFallback.class)
public interface PlatformApiFeignClient {

    @RequestMapping(value = "/api/test", method = RequestMethod.GET)
    ResponseMessage test(@RequestHeader("access-token") String token,
                         @ModelAttribute QueryParam queryParam);
    
    class PlatformApiFeignClientFallback implements PlatformApiFeignClient {
        @Override
        public ResponseMessage test(String token, QueryParam queryParam) {
            throw new BusinessException("feign访问异常!");
        }
    }
                        
}
```

现通过Feign调用该接口时，发现请求参数无法接收到。

## 原因

经过调查发现Spring Cloud Feign传递复杂参数仅能通过请求体，即使用@RequestBody接收请求参数。

[Support object parameters in feign #617](https://github.com/spring-cloud/spring-cloud-netflix/issues/617)

这样来看解决方案有两种，
1、新增接口/修改原接口，将请求方式修改为Post，并修改参数接收方式为@RequestBody。
2、拦截Feign请求，将请求参数转换后拼到url后，使其能够被@ModelAttribute方式接收到。

但是目前原接口现在仍在使用，不能修改原请求方式和参数接收方式；另外从代码重用角度来看，重新新增接口也不利于后期的代码维护，而且这种@ModelAttribute需要Feign远程调用的接口也不少，因此最终解决方案选择了第二种。

## 最终解决方案

新增自定义注解@FeignModelAttribute。

```
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * feign ModelAttribute接口请求参数注解
 * @author hefan
 * @date 2018/9/25 11:19
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface FeignModelAttribute {

}
```

标记远程调用@ModelAttribute接口的请求对象。

```
@FeignModelAttribute
public class QueryParam {
    //...
}
```

添加Feign编码器，通过反射获取标记在请求对象上的注解@FeignModelAttribute，若存在则将请求参数转换拼接在url之后。

```
import java.lang.reflect.Field;
import java.util.ArrayList;
import java.util.List;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.diditech.iov.picc.api.annotation.FeignModelAttribute;

import feign.RequestTemplate;
import feign.codec.Encoder;

/**
 * feign调用远程ModelAttribute接口解决参数问题
 * @author hefan
 * @date 2018/9/25 11:19
 */
@Configuration
public class FeignEncoder {

    /**
     * 自定义feign encoder
     * @author hefan
     * @date 2018/9/25 11:21
     */
    @Bean
    public Encoder encoder() {
        SpringEncoder springEncoder = new SpringEncoder(messageConverters);
        //Encoder defaultEncoder = new Encoder.Default();
        return (object, bodyType, template) -> {
            Class clazz = (Class)bodyType;
            if (null != clazz.getAnnotation(FeignModelAttribute.class) ) {
                modelAttributeParamEncoder(clazz,object,template);
            } else {
                defaultEncoder.encode(object, bodyType, template);
            }
        };
    }

    /**
     * modelAttribute参数转换
     * @author hefan
     * @date 2018/9/25 11:21
     */
    public void modelAttributeParamEncoder(Class clazz, Object object, RequestTemplate template){
        for (Field field : clazz.getDeclaredFields()) {
            field.setAccessible(true);
            try {
                Object data = field.get(object);
                if(data == null) continue;
                if(field.getType() == List.class){
                    List<String> s = new ArrayList<>();
                    for(Object o : (List<?>) data){
                        s.add(o.toString());
                    }
                    template.query(field.getName(), s.toArray(new String[]{}));
                } else {
                    template.query(field.getName(), data.toString());
                }
            } catch (IllegalAccessException e) {
                continue;
            }
        }
    }

}
```

