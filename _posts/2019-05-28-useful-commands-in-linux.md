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

```bash
lsof -i:8081
```
回显：
```bash
COMMAND   PID USER   FD   TYPE     DEVICE SIZE/OFF NODE NAME
java    40168 apps   16u  IPv4 1472358374      0t0  TCP *:tproxy (LISTEN)
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

## 查看linux版本信息
```bash
1、lsb_release -a，即可列出所有版本信息：Description: CentOS release 6.5 (Final)
2、cat /etc/redhat-release，这种方法只适合Redhat系的Linux：CentOS release 6.7 (Final)
3、cat /etc/issue，此命令也适用于所有的Linux发行版：CentOS release 6.7 (Final)
```

## 查找文件/文件夹位置
```bash
find / -name 'tomcat7' -type d #文件夹
find / -name '*tomcat*' # 文件 模糊查找	
```

## 根据关键字扫描和查找整个文件夹下的日志文件
```bash
grep -r -E "WARN|ERROR" /var/log/
```

## 根据关键字杀死进程(不好用，即使杀掉了也会报错)
```bash
sudo kill -s 9 `ps -aux | grep nodemanager | awk '{print $2}'`
```

## 查看CPU核数
```bash
# 总核数 = 物理CPU个数 X 每颗物理CPU的核数 
# 总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数

# 查看物理CPU个数
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l

# 查看每个物理CPU中core的个数(即核数)
cat /proc/cpuinfo| grep "cpu cores"| uniq

# 查看逻辑CPU的个数
cat /proc/cpuinfo| grep "processor"| wc -l
```

## 删除指定日期前的文件
```bash
显示20分钟前的文件
find /home/prestat/bills/test -type f -mmin +20 -exec ls -l {} \;

删除20分钟前的文件
find /home/prestat/bills/test -type f -mmin +20 -exec rm {} \;

显示20天前的文件
find /home/prestat/bills/test -type f -mtime +20 -exec ls -l {} \;

删除20天前的文件
find /home/prestat/bills/test -type f -mtime +20 -exec rm {} \;
```

## 显示各文件夹的大小（当前文件夹下各文件夹的大小）
```
du -h --max-depth=1
```

## 显示总大小（/下全部文件占用大小）
```
du -sh /* | sort -nr
```

## systemctl 配置文件在 /etc/systemd/system/nodemanager.service 修改后，必须执行`sudo systemctl daemon-reload`才可生效
```bash
sudo systemctl daemon-reload
sudo systemctl stop nodemanager
sudo systemctl start nodemanager
```

## linux sed 批量替换多个文件中的字符串
```bash
sed -i "s/oldstring/newstring/g" `grep oldstring -rl yourdir`

 # 例如：替换/home下所有文件中的www.bcak.com.cn为bcak.com.cn

sed -i "s/www.bcak.com.cn/bcak.com.cn/g" `grep www.bcak.com.cn -rl /home
```
