---
layout: post
title: "寻找MySQL8.0 初始密码"
subtitle: "How to find the password after mysql8.0 installation"
author: "qingshan"
header-img: "img/about-bg-walle.jpg"
header-mask: 0.4
tags:
  - Mysql
  - Linux

---



### 1 确保 MySQL 服务已经启动

```shell
sudo systemctl start mysqld
```

### 2 在启动日志里根据关键字查找

```shel
2021-07-03T23:24:35.550630Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: Q1yLhoUzru;3
```

### 3 修改密码

```shell
mysql -uroot -p
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'qMn7^#w7Q2#l';
```



## 如果上述方法不行，也可以采取安全启动的方式

略



# 开启外部访问

```mysql
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> update user set host='%' where user ='root';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.01 sec)

mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'WITH GRANT OPTION;
Query OK, 0 rows affected (0.01 sec)
```



# 新建用户并授权

```mysql
CREATE USER 'my2'@'%' IDENTIFIED BY 'yBy%d^92a2rN';
GRANT SELECT,INSERT,UPDATE ON my2.* TO 'my2'@'%';

# 撤销授权
REVOKE ALL ON my2.* FROM 'my2'@'%';
```



# 查看/修改端口号

```mysql
show global variables like 'port';  
```

```bash
sudo vim /etc/my.cnf

[mysqld]  
port=3506  
datadir=/var/lib/mysql  
socket=/var/lib/mysql/mysql.sock  
user=mysql  
# Disabling symbolic-links is recommended to prevent assorted security risks  
symbolic-links=0  
  
[mysqld_safe]  
log-error=/var/log/mysqld.log  
pid-file=/var/run/mysqld/mysqld.pid  
```

