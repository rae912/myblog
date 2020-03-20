---
layout: post
title: "解决MacOS上的Mysql不能被访问的问题"
subtitle: "Solve the problem which mysql on MacOS cannot be connected"
author: "qingshan"
header-img: "img/post-sample-image.jpg"
header-mask: 0.4
tags:
  - 工作
  - MySQL
---

最近工作上开发环境需要与其他环境交换测试数据。写好程序后，死活连不上开发机。反复折腾配置环境、防火墙、网络之后，总算发现原来是MacOS上brew的锅！

MacOS上的MySQL一般都是通过brew安装的，brew默认的MySQL配置为了安全考虑，默认是只允许本地访问的。但是这一条限制并没有明确的在任何地方说明。所以最终找到这个原因的时候，也是大跌眼镜。下面是排查过程：

##### 1 使用命令检查MySQL开放端口
```shell
lsof -i:3306
mysqld    50249  qingshan   21u  IPv4 0x4dd6e74ea03c4dcb      0t0  TCP localhost:mysql (LISTEN)
```

可以看到，MySQL只监听localhost本地，这就是为什么其他的客户端无法访问的原因。

##### 2 解决方法
通过查询brew安装方式MySQL的配置文件地址，找到对应的MySQL配置。强行在MySQL配置文件中加上允许外网监听的配置选项。

查找MySQL配置文件路径
```shell
cd $(brew --prefix mysql)/
```

修改配置文件
```shell
vim homebrew.mxcl.mysql.plist

  <array>
    <string>/usr/local/opt/mysql/bin/mysqld_safe</string>
    <string>--datadir=/usr/local/var/mysql</string>
    <string>--bind-address=0.0.0.0</string>
  </array>
```

重启MySQL
```shell
brew services restart mysql
Stopping `mysql`... (might take a while)
==> Successfully stopped `mysql` (label: homebrew.mxcl.mysql)
==> Successfully started `mysql` (label: homebrew.mxcl.mysql)
```

##### 3 问题解决
再从其他机器上尝试访问本台机器上的MySQL
```shell
telnet 10.xx.xx.100 3306
```

提示已经可以连上。问题解决。