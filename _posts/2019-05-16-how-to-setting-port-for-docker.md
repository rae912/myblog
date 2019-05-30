---
layout: post
title: "Docker环境下给指定的容器设置端口映射"
subtitle: "How to setting port with docker running containtors"
author: "qingshan"
header-img: "img/home-bg-o.jpg"
header-mask: 0.4
tags:
  - 工作
  - Docker
---

最近定制了一个docker镜像，用于线上快速部署。可是上线后发现有点问题，于是在容器里进行了修改。可是修改完之后，发现端口映射并不能对容器进行设置，而针对的是镜像。

经过查阅资料，发现给容器指定端口可以使用以下两种方法：

### 方法1: 提交容器为镜像
提交一个运行中的容器为镜像，就可以使用docker -p命令针对镜像设置端口映射。
```
docker commit containerId newImageName
```

运行镜像并添加端口
```
docker run -d -p 8000:80 newImageName /bin/bash
```

### 方法2: 修改iptable转发端口 (不推荐)
这种做法有一定的风险，实际上修改了宿主机的端口配置。
1、获得容器IP
将container_name 换成实际环境中的容器名
```
docker inspect `container_name` | grep IPAddress
```

2、iptable转发端口
将容器的8000端口映射到docker主机的8001端口

代码如下:
```
iptables -t nat -A  DOCKER -p tcp --dport 8001 -j DNAT --to-destination 192.168.x.x:8000
```


## 另：docker1.7 上传文件到容器报错 Error: Path not specified的解决办法
docker1.7 上传文件到容器有个Bug，就是使用 `docker cp`命令时会提示：Error: Path not specified。根本解决办法就是升级到1.8版本。

但是对于线上暂时无法升级的环境，有个变通的办法就是：
docker containtor 实际上就是宿主机上的一个目录，其文件系统对应着宿主机上的一个文件夹，只要找到文件夹直接复制过去就可以看到了。
方法如下：

首先要获取容器的真实id: (markdown转义字符不准确，其实就是两个花括号里面有.Id字符)
```
docker inspect -f '\{\{.Id\}\}\' containtorName
```
得到例如：
```
88dc769b9b6e0e2d05161751f70f033f902f23c14b0c330efeea1a22f0xxxxx
```

复制需要拷贝的文件到 到 此路径 
```
cp openvas-install.sh /var/lib/docker/devicemapper/mnt/88dc769b9b6e0e2d05161751f70f033f902f23c14b0c330efeea1a22f0xxxxx/rootfs/home
```

最后的`home`对应的就是容器的home目录，依次类推，`rootfs`就是容器的`/`根目录。





