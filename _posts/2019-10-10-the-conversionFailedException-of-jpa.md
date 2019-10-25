---
layout: post
title: "关于JPA的ConversionFailedException报错"
subtitle: "The ConversionFailedException of JPA"
author: "qingshan"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.4
tags:
  - 工作
  - Java
---

今天在项目中修改一个Bug，使用JPA在MySQL中做多表联合查询。因为只需要结果中的某几个字段，于是就自定义了一个model。使用@Query将查询结果绑定到自定义model上，却报错：
```
ConversionFailedException: Failed to convert from type [java.lang.Object[]] to type
```

仔细检索后，得知原因是：
```
对接接口的时候发现这个错误，经过查找发现是在Repository中的JPA nativeQuery中，直接使用了对象返回，但是nativeQuery返回的东西应该是一个Map 、 List<Map> 或者 [java.lang.Object[]]之类的不能直接转化为对象。
```

解决办法：

## 方法一
使用Object[]数组来接收数据 ，Object[]中的每一个元素值就是对应列的值
```java
@Query(select p.id,p.name,p.age from Person p)
 List<Object[]> findPersonResult();
```

## 方法二 (没有成功)
使用作为查询结果的类型：

```java
List<Map<String,Object>> findConsumeRule(@Param("deviceId") Integer deviceId);
```
