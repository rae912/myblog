---
layout: post
title: "Java Spring boot启动过程中一个奇怪的报错"
subtitle: "A strange question in spring boot start process"
author: "qingshan"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.4
tags:
  - 工作
  - Java
  - Spring
---

今天spring-boot启动时候出现一个奇怪的异常：
```sh
The bean 'xxx' could not be injected as a 'xx.xxxx' because it is a JDK dynamic proxy that implements:

Action:  
  
Consider injecting the bean as one of its interfaces or forcing the use of CGLib-based proxies by setting proxyTargetClass=true on @EnableAsync and/or @EnableCaching.
```

查找了一圈，发现可能的原因，最后也的确是这样解决的：
>* 主要问题是名字和类名不一样导致 Resource 注入失败，但是刚好又有一个 xxxService 同名的类存在，就会报这个错误
>* 这里要说一下，@Resource 是根据名字找对象，名字是什么的名字呢？
>* 先根据变量名字找，如果找到就直接使用，如果找不到就才根据类的名字找。
>* 这个时候刚好有一个同名的对象存在，直接返回赋值，但是这个时候类型和对象是不一致的，所以就报异常。
>* 解决方法就是 名字改成和类一样

```java
@Resource
private AaaXxxService aaaXxxService;
```