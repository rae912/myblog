---
layout: post
title: "NodeManager无法启动的原因"
subtitle: "Why the nodemanager cannot start"
author: "qingshan"
header-img: "img/about-bg-walle.jpg"
header-mask: 0.4
tags:

- 工作
- Yarn
- Hadoop

---

今天因为业务需要，需要将Yarn从2.6.0升级到2.7.6。将hadoop组件包用新版本替换后，修改config文件`yarn-site.xml`，再使用`yarn-daemon.sh stop/start nodemanager`，再使用jps命令查看nodemanager进程。观察到nodemanager进程启动后很快就自己退出了。

通过查询日志，得知：
```bash
protocol message contained an invalid tag (zero)
``` 

再对比ResourceManager那边的日志，发现NodeManager的通讯信息根本就没有抵达到RM。那么可以定位是protocol的问题。本来以为是谁在合并代码的时候，动了protocol这一块逻辑。问了一圈下来，没有发现线索。后来一想，会不会是老版本的container在NM启动的时候还是加载的原来的业务逻辑，于是找到了container在NM stop时候，存放缓存文件的路径，并将其清空：
```bash
sudo rm -rf /home/yarn/nm_recovery
```

再使用`yarn-daemon.sh stop/start nodemanager`就正常了。看起来就是这里的问题。NM 在start的时候，还是会尝试继续没有完成的任务，沿用原来老版本的业务逻辑和配置。清空之后，就会切换到新的版本上来。问题解决。

