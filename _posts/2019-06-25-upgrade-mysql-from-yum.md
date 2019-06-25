---
layout: post
title: "使用Yum对MySQL进行半自动升级"
subtitle: "Upgrade MySQL with Yum"
author: "qingshan"
header-img: "img/post-bg-js-version.jpg"
header-mask: 0.4
tags:
  - 工作
---

最近因为业务需要在MySQL中建立了一张表，里面有两个“CURRENT_TIMESTAMP”字段。但是在开发环境好好的，上测试环境怎么也建立不成功了。猜测可能是MySQL版本问题。于是查了一下，发现开发环境是5.7.12，测试环境居然还是5.1的老版本。决定对测试环境的MySQL进行升级。

因为系统是CentOS 6.x，原来的MySQL是通过yum安装的，经过检索发现也可以通过yum升级。这样就可以省去了“卸载--安装--配置”的步骤了，看起来很不错。

## 0. 备份老版本MySQL数据

要动数据库，先无脑备份一下有备无患。
```
sudo mysqldump -pparrot -p parrot > parrot.sql
```

## 1. 配置MySQL的Yum源
在MySQL官网找到官方提供的Yum源：

引用页：
```
https://dev.mysql.com/downloads/repo/yum/
```
下载链接：
```
wget -c https://dev.mysql.com/get/mysql80-community-release-el6-3.noarch.rpm
```
安装yum源
```
rpm -ivh mysql80-community-release-el6-3.noarch.rpm
```

## 2. 配置升级后的目标MySQL版本
第1步中的源是中官方提供的全部版本，如果不配置就升级的话，默认安装是最新的8.0。我们暂时用不了这么新的版本，选择5.7.22即可。所以必须得先配置一下：
```
sudo vim /etc/yum.repos.d/mysql-community.repo
```

下面配置中的`enabled`参数即是选择开关。这里打开5.7版本的enabled，关闭8.0的：
```
[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql80-community]
name=MySQL 8.0 Community Server
baseurl=http://repo.mysql.com/yum/mysql-8.0-community/el/6/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

## 3. 开始Yum升级
```
sudo yum update mysql-server
```
命令执行之后，yum会将MySQL相关的组件全部扫描并升级，包括客户端，连接器等等。

## 4. 重启并完成新版本MySQL的配置
在升级完成之后，新版本的MySQL才可以生效。我这里使用系统Service命令进行管理。
```
sudo service mysqld restart
```
但是此时会发现MySQL怎么也启动不起来，报错信息：
```
MySQL Daemon Failed to Start 
Starting mysqld: [failed]
```

查看MySQL的启动脚本：
```
sudo vim /etc/rc.d/init.d/mysqld

$exec $MYSQLD_OPTS --datadir="$datadir" --socket="$socketfile" \
        --pid-file="$mypidfile" \
        --basedir=/usr --user=mysql $extra_opts >/dev/null &
safe_pid=$!
# Spin for a maximum of N seconds waiting for the server to come up;
# exit the loop immediately if mysqld_safe process disappears.
# Rather than assuming we know a valid username, accept an "access
# denied" response as meaning the server is functioning.
ret=0
TIMEOUT="$STARTTIMEOUT"
while [ $TIMEOUT -gt 0 ]; do
    RESPONSE=$(/usr/bin/mysqladmin --no-defaults --socket="$adminsocket" --user=UNKNOWN_MYSQL_USER ping 2>&1) && break
    echo "$RESPONSE" | grep -q "Access denied for user" && break
    if ! /bin/kill -0 $safe_pid 2>/dev/null; then
        echo "MySQL Daemon failed to start."
        ret=1
        break
    fi
    sleep 1
    let TIMEOUT=${TIMEOUT}-1
done
```
似乎是启动过程中的问题，导致MySQL不能拉起。查看系统日志，定位具体问题：
```
sudo cat /var/log/mysqld.log

Fatal error: mysql.user table is damaged. Please run mysql_upgrade
```
在日志中发现这么一句话，大意是`mysql.user`表被破环，导致问题。要我使用`mysql_upgrade`进行修复/升级。大胆猜测是因为新版本的MySQL使用了老版本的user表，但是没有找到能用的有足够高权限的用户，所以无法进行后续初始化动作。
进行搜索之后，印证了我的猜测。于是准备先让新版本MySQL不检查权限，完成新版本MySQL的初始化再说。

## 5. 初始化新版本MySQL
```
sudo vim /etc/my.cnf

# 添加skip-grant-tables不检查权限
[mysqld]
skip-grant-tables

# 重启mysql让新版本生效
sudo service mysqld restart
```

## 5.1 新版本MySQL无法启动的处理办法
如果此时新版本MySQL还是无法启动，则最有可能的原因是，原来在`/etc/my.cnf`位置的默认MySQL配置文件被新版本的数据库读取了。因为新版本的配置已经无法兼容老版本的，所以出现了这个问题。解决办法如下：
```
# 重命名（或者删除）旧配置文件
sudo /etc/my.cnf /etc/my.cnf.old

# 重新安装新版本数据库
sudo yum reinstall mysql-server

# 回到第五步初始化新版本MySQL
sudo vim /etc/my.cnf

# 添加skip-grant-tables不检查权限
[mysqld]
skip-grant-tables

# 重启mysql让新版本生效
sudo service mysqld restart
```

## 6. 自动升级数据库数据
成功拉起新版本MySQL之后，执行`mysql_upgrade`进行自动升级、修复。完成后进入直接最高权限进入MySQL查看表数据是否正确。
```
sudo mysql
# 不用密码，直接默认就是root权限
```

刚好原来的root密码还不知道，可以用以下命令顺便修改一下root用户密码。方便下次使用。
```
update user set authentication_string=password('新密码') where user='root';
```

如果`mysql_upgrade`运行完毕后提示有错误，则可以在MySQL命令行手动导入旧数据的数据：

```
source parrot.sql
```

## 7. 最后别忘了把MySQL的配置文件改回来
```
# 删除skip-grant-tables不检查权限
[mysqld]
# skip-grant-tables

# 重启MySQL生效
sudo mysqld restart
```





