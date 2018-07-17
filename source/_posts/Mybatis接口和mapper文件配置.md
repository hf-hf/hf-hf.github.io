---
title: Mybatis接口和mapper文件配置
date: 2018-07-17 16:14:57
tags:
    -- druid
---

### 配置说明

使用maven构建工程，目录结构有：src/main/java和src/main/resources，前者用来存放java源代码，后者则是存放一些资源、配置文件。

在打包的时候，对于src/main/java目录只会打包源代码，而不会打包其他文件。所以如果将Mybatis的mapper.xml文件放在src/main/java目录下，打包时不会输出到target目录。

因此在开发中，通常会将mapper.xml文件放到src/main/resources下，并且包路径与接口文件相同，这样在maven打包的时候就会将src/main/java和src/main/resources相同包下的文件合并到同一包中。

但是在Mybatis中接口和对应的mapper.xml文件不一定要放在同一个包下面，放在一起的目的是为了Mybatis进行自动扫描，并且要保持java接口的名称与mapper.xml文件的名称一致。

如果接口和mapper.xml文件不在同一个包下面，Mybatis不能进行自动扫描解析，因此需要对接口和mapper文件位置进行配置。

使用MapperScan注解标明接口文件所在位置，通过PathMatchingResourcePatternResolver扫描对应包下的mapper.xml，并设置sqlSessionFactoryBean的MapperLocations，详细见下方代码。

### 配置类代码

```
// 配置类
@Configuration("defaultDruidDBConfig")
// 扫描接口类，这个配置只能扫描该包下的接口，不能扫描mapper文件
@MapperScan(basePackages = { "com.kindle.**.repository" },
        sqlSessionFactoryRef = "defaultSqlSessionFactoryBean")
public class DruidDBConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean(name = "defaultDataSource")
    @Primary
    public DataSource dataSource(){
        return new DruidDataSource();
    }
    
    @Bean(name = "defaultSqlSessionFactoryBean")
    public SqlSessionFactory sqlSessionFactoryBean(@Qualifier("defaultDataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        sqlSessionFactoryBean.setFailFast(true);
		PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
		// 扫描对应包下的的mapper.xml文件
		Resource[] mapperLocations = resolver.getResources("classpath*:/com/kindle/**/*Mapper.xml");
        sqlSessionFactoryBean.setMapperLocations(mapperLocations);
        //注册mybatis拦截器
        sqlSessionFactoryBean.setPlugins(new Interceptor[]{pageInterceptor()});
        return sqlSessionFactoryBean.getObject();
    }
    
    //...以下省略
}
```



