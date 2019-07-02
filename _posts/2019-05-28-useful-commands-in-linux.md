---
layout: post
title: "Linux下好用的命令"
subtitle: "Useful command in Linux"
author: "qingshan"
header-img: "img/post-bg-unix-linux.jpg"
header-mask: 0.4
tags:
  - 工作
  - Linux
---



## 查看端口占用
```bash
netstat -anp|grep 80 
```
回显：
```
tcp        0      0 127.0.0.1:51501             127.0.0.1:8082              ESTABLISHED 14814/nginx
tcp        0      0 127.0.0.1:50254             127.0.0.1:8082              ESTABLISHED 14814/nginx
tcp        0      0 :::8082                     :::*                        LISTEN      12670/node
tcp        0      0 ::ffff:127.0.0.1:8082       ::ffff:127.0.0.1:50254      ESTABLISHED 12670/node
tcp        0      0 ::ffff:127.0.0.1:8082       ::ffff:127.0.0.1:51501      ESTABLISHED 12670/node
```

## 查看某个目录下的文件占用排行
```bash
cd /path/to/some/where
du -hsx * | sort -rh | head -10
```

```bash
11M	apache-maven
1.8M	lib
604K	share
88K	include
12K	fbsys
4.0K	src
4.0K	sbin
4.0K	libexec
4.0K	lib64
4.0K	games
```

## 根据进程名字模糊匹配kill进程
```bash
# 先通过ps将进程ID得到，然后再kill。并在这条命令上加个判断，如果存在则运行kill，不存在则不执行kill。
ps -ef | grep demo.jar | grep -v grep | awk '{print $2}' | xargs --no-run-if-empty kill
```

