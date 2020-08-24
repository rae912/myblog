---
layout: post
title: "Tmux和Htop自动安装过程"
subtitle: "Install Tmux and Htop automatically"
author: "qingshan"
header-img: "img/post-bg-product-manager.webp"
header-mask: 0.4
tags:
  - 工作
  - Linux
---


使用Java Spring Boot 开发后端API，要用到用户登录状态，所以在`Webconfigure`里设置了拦截器，将除了`/login`之外没有鉴权的API都重定向到`/login`，拦截器代码如下：

```java
    @Bean
    public SecurityInterceptor getSecurityInterceptor() {
        return new SecurityInterceptor();
    }

    private class SecurityInterceptor extends HandlerInterceptorAdapter {
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws IOException {
            Cookie[] cookies= request.getCookies();

            if (cookies == null) {
                response.sendRedirect("/api/v1/auth/login");
                return false;
            }

           // Judge if there is user's session
            if (new CookieClient().validateCookie(cookies)) {
                return true;
            } else {
                logger.warn("The user's identify is fake");
                response.sendRedirect("/api/v1/auth/login");
                return false;
            }
        }
    }
```

但是项目是用Swaager2来生成文档的，所以这样一来，默认的`swagger-ui.html`也无法访问了，所以要将swagger相关的一系列接口统统排除在外。需要注意的是swagger不仅仅使用一个接口。而是很多：

```java
    //添加拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        InterceptorRegistration addInterceptor = registry.addInterceptor(getSecurityInterceptor());

        addInterceptor.addPathPatterns("/**")
                .excludePathPatterns("/v2/api-docs")
                .excludePathPatterns("/configuration/ui")
                .excludePathPatterns("/swagger-resources/**")
                .excludePathPatterns("/configuration/security")
                .excludePathPatterns("/swagger-ui.html")
                .excludePathPatterns("/webjars/**");

    }
```

以上路径配置后，就可以访问swagger了。

> https://stackoverflow.com/questions/37671125/how-to-configure-spring-security-to-allow-swagger-url-to-be-accessed-without-aut
