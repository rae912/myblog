---
layout: post
title: "Linux下好用的命令"
subtitle: "Useful command in Linux"
author: "qingshan"
header-img: "img/post-bg-unix-linux.jpg"
header-mask: 0.4
tags:
  - 工作
---



## 查看端口占用
```
netstat -anp|grep 80 
```

## 查看某个目录下的文件占用排行
```
cd /path/to/some/where
du -hsx * | sort -rh | head -10
```