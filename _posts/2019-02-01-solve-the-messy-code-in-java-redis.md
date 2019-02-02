---
layout: post
title: "解决Spring Boot Redis Key乱码问题"
subtitle: "Solve the messy code problem in spring boot"
author: "qingshan"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.4
tags:
  - 工作
  - Java
---

在使用Spring Boot连接Redis时，在读写Redis成功之后，如果通过redis-client连接上去查看，会发现所有的reids key都以一串类似乱码的UTF-8字符开头。尽管不影响程序运行，也不影响业务，但是通过cli查看还是略有不便。

起初是以为key中混入了中文字符。因为中文编码在英文环境里经常就是这个样子。但是，在查阅了相关资料以后，才发现原因是因为spring-data-redis的RedisTemplate<K, V>模板类在操作redis时默认使用JdkSerializationRedisSerializer来进行序列化。而默认是没有设置编码的。

解决的办法也很简单，就是在redisTemplate初始化时候，将字符类型指定为stringSerializer就可以了。

代码如下：

```java
private RedisTemplate redisTemplate;

@Autowired(required = false)
public void setRedisTemplate(RedisTemplate redisTemplate) {
    RedisSerializer stringSerializer = new StringRedisSerializer();
    redisTemplate.setKeySerializer(stringSerializer);
    redisTemplate.setValueSerializer(stringSerializer);
    redisTemplate.setHashKeySerializer(stringSerializer);
    redisTemplate.setHashValueSerializer(stringSerializer);
    this.redisTemplate = redisTemplate;
}
```