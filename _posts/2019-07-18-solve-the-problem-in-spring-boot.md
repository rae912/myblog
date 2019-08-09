---
layout: post
title: "Java Spring Boot CORS问题的解决方案"
subtitle: "Solve the problem in spring boot"
author: "qingshan"
header-img: "img/post-bg-rwd.jpg"
header-mask: 0.4
tags:
  - 工作
  - Java
---

最近调试另外一个同事开发的项目时，发现遇到了CORS问题。这是很常见的资源限制问题，初衷是限制不安全的跨域访问。但是在内部使用的项目中，这个安全限制不管是在开发过程中，还是跨系统调用中，都非常不方便。于是整理了两个切实有效的方案。

#### 1. 网络转发法
这个是前端妹子提供的方案，这个方案原理是需将跨域访问的资源进行本地网络转发（或者rewrite）为自定义的开发域，规避浏览器的限制。
需要用到的工具：**Fiddler**(Windows) **Charles**(MacOS)
配置如图：
Fiddler下叫AutoResponser
![](https://ae01.alicdn.com/kf/HTB1mJ8saQH0gK0jSZPi761vapXaV.png)

Charles下叫ReWrite
![](https://ae01.alicdn.com/kf/HTB1OFXraND1gK0jSZFs762ldVXa3.png)

具体设置方法如图所示。


#### 2. 后端配置法
其实限制的关键控制是在后端，只要后端显式的声明允许跨域访问，问题也就迎刃而解。手上这个项目是Java Spring Boot，配置方法如下:
```java
    // 解决CORS问题 去掉大部分的跨域限制
    @Configuration
    public class CorsConfig {
        @Bean
        public FilterRegistrationBean corsFilter() {
            UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
            CorsConfiguration config = new CorsConfiguration();
            config.setAllowCredentials(true);
            // 设置你要允许的网站域名，如果全允许则设为 *
            config.addAllowedOrigin("*");
            // 如果要限制 HEADER 或 METHOD 请自行更改
            config.addAllowedHeader("*");
            config.addAllowedMethod("*");
            source.registerCorsConfiguration("/**", config);
            FilterRegistrationBean bean = new FilterRegistrationBean(new CorsFilter(source));
            // 这个顺序很重要哦，为避免麻烦请设置在最前
            bean.setOrder(0);
            return bean;
        }
    }
```
代码中星号的部分可以根据业务需要灵活设置。如果保留星号，就是允许全部的意思。

== END ==

