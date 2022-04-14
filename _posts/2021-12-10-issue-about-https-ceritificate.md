---
layout: post
title: "解决 HTTPS 的证书问题"
subtitle: "How to handle issue about the HTTPS certificate"
author: "qingshan"
header-img: "img/post-bg-rwd.jpg"
header-mask: 0.4
tags:
  - HTTPS
  - Linux
---


因为一些合规方面的需要，所有涉及 HTTP 请求都需要升级为安全性更高的 HTTPS 协议。所以必须配置 SSL Certificates。在配置的过程中，发生了一些小插曲。特此在这里记录一下。

## 证书的申请

作为实验性质，首先考虑的是免费的证书。所以选择的是`Let's Encrypt` 和 `Zero SSL`。申请的过程就略过了。这里简单记录一下：

### 1 Let's Encrypt

`Let's Encrypt` 可以选用官网提供的 certbot 命令方便的自动验证和申请。前提是需要打开 80 和 443 端口，并保证在验证期间没有被占用。证书会被自动下载到服务器的`/etc/letsencrypt/live/{domain}` 目录下。

### 2 Zero SSL

`Zero SSL` 的是通过 DNS 来验证，意味着全程不用登陆服务器操作。只需要去 DNS 供应商按`Zero SSL`的要求改成指定验证信息即可。证书随后在`Zero SSL`的网站上，通过浏览器下载 ZIP 包的方式传输到服务器。

## 遇到的问题和解决

### 1 Let's Encrypt 

`Let's Encrypt` 会在服务器的 `/etc/letsencrypt/live/{domain}` 位置存放证书，这下面默认有如下文件：

```bash
lrwxrwxrwx. 1 root root  35 Dec 24 15:03 cert.pem -> ../../archive/{domain}/cert2.pem
lrwxrwxrwx. 1 root root  36 Dec 24 15:03 chain.pem -> ../../archive/{domain}/chain2.pem
lrwxrwxrwx. 1 root root  40 Dec 24 15:03 fullchain.pem -> ../../archive/{domain}/fullchain2.pem
lrwxrwxrwx. 1 root root  38 Dec 24 15:03 privkey.pem -> ../../archive/{domain}/privkey2.pem
````

正常情况，`privkey.pem` 是私钥，`cert.pem`是证书。但是实际上，如果选用`cert.pem` 会提示 X509 证书无效。
解决方法：选用 `fullchain.pem` 作为证书。不知道 `Let's Encrypt`  为什么要这么设计。


### 2 Zero SSL

 `Zero SSL` 从网上下载证书 ZIP 压缩包后，解压后文件如下：

```bash
-rw-rw-r--. 1 root root 2432 Nov 21 07:58 ca.crt
-rw-rw-r--. 1 root root 2286 Nov 21 07:57 cert.crt
-rw-rw-r--. 1 root root 1676 Nov 21 07:57 priv.crt
```

正常情况，`priv.crt` 是私钥，`cert.crt`是证书。但是实际上，如果选用`cert.crt` 也会提示 X509 证书无效。
解决方法：

```bash
# 执行命令
cat ca.cert cert.crt > full.crt
```
将 ca 和 cert 的内容合并为一个 full 证书。选用这个合成后的新 `full.crt` 作为证书，即可恢复正常。


特此记录。

