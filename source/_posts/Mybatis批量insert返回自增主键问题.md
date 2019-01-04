---
title: Mybatis批量insert返回自增主键问题
date: 2019-01-04 11:11:23
tags:
---
![homePage](/upload/homePage/20190104095505.jpg)
<!--more-->

## 背景
Mybatis3.3.1及其之后的版本支持批量插入返回自增主键ID，在3.3.1之前的版本该功能存在bug，相关问题在此链接上有详细解释：[Mybatis之foreach批量insert，返回主键id列表（修复Mybatis返回null的bug）](https://my.oschina.net/zudajun/blog/674946)。

## 代码

TestMapper.xml

```
<!-- 批量添加数据，并返回主键字段 -->
<insert id="insertBatchTest" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO user(email, name, password, create_time) 
    VALUES
        <foreach collection="list" separator="," item="item">
        (#{item.email},#{item.name},#{item.password},#{item.createTime})
        </foreach>
</insert>
```

返回的主键值在执行批量插入的list中的每个对象的id字段中，id字段对应keyProperty="id"。

## 原理

其实现依托于Jdbc插入后返回主键的策略，下面以单条插入返回自增长主键为例：

```
//构建单条插入sql
String sql = "INSERT INTO user(email, name, password, create_time) VALUES('test','test','123456','2019-01-04 00:00:00')";
// 指定返回生成的主键 
PreparedStatement pstmt = conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
// 真正执行插入
pstmt.executeUpdate(); 
// 检索执行完成后返回的ResultSet，在其中有插入数据行的自增长主键
ResultSet rs = pstmt.getGeneratedKeys(); 
int id;
if (rs.next()) { 
    id = rs.getInt(1);//主键的数据类型为int
}
return id;
```
