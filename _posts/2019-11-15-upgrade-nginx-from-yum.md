---
layout: post
title: "通过yum更新较新版本的Nginx"
subtitle: "Upgrade nginx from yum"
author: "qingshan"
header-img: "img/post-bg-product-manager.webp"
header-mask: 0.4
tags:
  - 工作
  - Nginx 
---

偶然发现生产服务器上面的nginx版本还是很旧的，但是通过yum尝试upgrade，却发现yum提示是最新的。不得不吐槽万年不更新的yum默认源。找了一下，发现可以通过安装官方的nginx源来更新，步骤如下：

# 1 安装官方yum源

```bash
sudo rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

安装之后，在`/etc/yum.repos.d/nginx.repo`目录下应该会有nginx的专用源。

# 2 查看可用的nginx版本

```bash
yum list |grep nginx
```

# 3 升级nginx

```bash
yum update nginx
```

# 4 异常处理

如果出现下面的提示：
```bash
nginx: [emerg] module "/usr/lib64/nginx/modules/ngx_http_geoip_module.so" version 1012002 instead of 1014002 in /usr/share/nginx/modules/mod-http-geoip.conf:1
```

解决办法：
```bash
yum remove nginx-mod* &&
yum install nginx-module-*
```
