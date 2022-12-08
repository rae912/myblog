---
layout: post
title: "解决宿主机不能访问容器IP的问题"
subtitle: "Solve the docker connection problem from host to container on MacOS"
author: "qingshan"
header-img: "img/in-post/post-bg-digital-native.jpg"
header-mask: 0.4
tags:

- Docker
- MacOS

---

# 背景

今天有一个需求，要在macOS上使用docker，从宿主机访问容器内的服务，并且访问IP还只能用容器IP。例如，容器IP是 `172.17.0.2`，要访问的入口之能是 `http://172.17.0.2:8030`。自己在Docker上试了一下，是访问不了的。只能通过传统的 -p 参数，将容器内的服务端口映射到宿主机，然后宿主机访问本地 `127.0.0.1:8030`来使用服务。但是这样就不符合需求了。

这个问题卡了我两天，想了各种办法，都不能很好解决。在GitHub上得到了明确的答复是由于macOS的系统限制，是无法从宿主机访问容器IP的，

> 原文如下：https://github.com/docker/for-mac/issues/2670
> 
> At the moment it's not easy in Docker for Mac to connect to the internal IP addresses used by containers, because they're exposed in a tiny VM rather than on the host. Ideally specific ports should be published with `docker run -p` which sets up a tunnel from the Mac to the VM. However if that doesn't work or is impractical for your use-case, then perhaps you could try this experimental build which contains a SOCKS server:
> 
> https://download-stage.docker.com/mac/bysha1/52ea5bcc410a8b62f03f09aa04ad4b7ffb9eed0c/Docker.dmg
> 
> It reports its version as
> 
> > Version 18.03.0-ce-rc2-mac56 (23206)
> > Channel: pr
> > 52ea5bcc41
> 
> To enable the proxy first shutdown the app, then enable the experimental SOCKS
> server on port 8888: (this requires the `jq` tool available from homebrew)
> 
> ```
> cd ~/Library/Group\ Containers/group.com.docker/
> mv settings.json settings.json.backup
> cat settings.json.backup | jq '.["socksProxyPort"]=8888' > settings.json
> ```
> 
> Restart the app again and, once it's running, go to
> 
> Apple System Preferences -> Network -> Advanced -> Proxies
> 
> <img width="666" alt="screen shot 2018-03-12 at 16 07 13" src="https://user-images.githubusercontent.com/198586/37295201-77fd5dcc-260f-11e8-8d31-8d1fd554f057.png">
> 
> and enable "SOCKS Proxy" using "localhost:8888", hit OK and then Apply.
> 
> If you open safari and try browsing, the traffic should be routed via Docker for Mac.
> 
> If you start an nginx container:
> 
> ```
> docker run -d --name nginx nginx
> ```
> 
> Query the internal IP:
> 
> ```
> docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' nginx
> ```
> 
> It should be possible to open `http://<IP>` in Safari.
> 
> Let me know if this is helpful or not!
> 
> _Originally posted by @djs55 in https://github.com/docker/for-mac/issues/2670#issuecomment-372365274_

# 解决方案

经过不懈努力，终于找到了一种hack的方式实现了。原文文章关键部分摘录如下：
## docker-connector

### 1. 安装docker-connector服务
使用brew安装docker-connector
```bash
brew install wenjunxiao/brew/docker-connector
```

执行下面命令将docker所有 bridge 网络都添加到docker-connector路由

```bash
docker network ls --filter driver=bridge --format "{{.ID}}" | xargs docker network inspect --format "route {{range .IPAM.Config}}{{.Subnet}}{{end}}" >> /opt/homebrew/etc/docker-connector.conf

（/usr/local/etc/docker-connector.conf是安装docker-connector后生成的配置文件）
```

### 2. 使用sudo启动docker-connector服务
```bash
sudo brew services start docker-connector

如果启动失败,要检查配置文件是否正确: /opt/homebrew/etc/docker-connector.conf
```

使用下面命令创建wenjunxiao/mac-docker-connector容器，要求使用 host 网络并且允许 NET_ADMIN
```bash
docker run -it -d --restart always --net host --cap-add NET_ADMIN --name connector wenjunxiao/mac-docker-connector

docker-connector容器启动成功后，macOS宿主机即可访问其它容器网络
```
### 3. 其他
如果macOS里面需要使用代理，proxychains4是比较好的选择。

参考文章：
[macOS宿主机连接容器网络](http://bigbigben.com/2022/03/03/about-docker-for-mac/)