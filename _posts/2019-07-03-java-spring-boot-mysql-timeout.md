---
layout: post
title: "Spring Boot中MySQL超时问题的解决"
subtitle: "Time out in Java Spring Boot"
author: "qingshan"
header-img: "img/home-bg-o.jpg"
header-mask: 0.4
tags:
  - 工作
  - Java
  - Spring
---

在Java Spring Boot添加了多个MySQL连接后，经常发现新加的MySQL报错：
```sh
com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: No operations allowed after connection closed. ] which will not be reported to listeners!
com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: No operations allowed after connection closed.
```

大意是MySQL链接超时了。之前在Python中也发现过，原因是MySQL默认在一个连接建立后只保持8小时的有效期，如果在8小时内没有使用，会从资源利用的角度单方面释放空闲的连接。当程序再用已被释放的连接的时候，就会报这个错。
当时Python中的解决办法是捕获这个错误，然后当发现连接Timeout时，再重新建立一下就可以了。

查阅了java spring 的相关资料后，发现有更简单的办法。即在MySQL配置文件中，加入相关的配置即可
```sh
spring.datasource.etl.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.etl.max-idle=10
spring.datasource.etl.max-wait=10000
spring.datasource.etl.min-idle=5
spring.datasource.etl.initial-size=5
spring.datasource.etl.validation-query=SELECT 1
spring.datasource.etl.test-on-borrow=true
spring.datasource.etl.test-while-idle=true
spring.datasource.etl.time-between-eviction-runs-millis=18800
```


