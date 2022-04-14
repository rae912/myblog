---
layout: post
title: "解决 Oracle Cloud Server 防火墙配置无效的问题"
subtitle: "How to solve the firewall problem on OCI"
author: "qingshan"
header-img: "img/post-bg-rwd.jpg"
header-mask: 0.4
tags:
  - Linux
---


在 Oracle Cloud 上面开了一台服务器，默认是 Ubuntu 20 的系统。今天发现一些服务怎么也访问不了。经过查询后，发现自带的 `ufw` 命令不起作用，而是要使用下面的命令：

```bash
$ sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT
$ sudo netfilter-persistent save
```


原因是，OCI 初始化定义了一批防火墙规则，写入到了 `iptables` 和 `netfilter-persistent`.

最直接的做法是 `iptables` 和 `netfilter-persistent` 都关掉，直接用控制台的硬件防火墙规则。

```bash
service iptables stop
netfilter-persistent start
netfilter-persistent stop
netfilter-persistent flush
netfilter-persistent save
```

