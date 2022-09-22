---
layout: post
title: "解决连接字符串中的特殊字符"
subtitle: "Resolve special characters in connector string"
author: "qingshan"
header-img: "img/in-post/post-bg-digital-native.jpg"
header-mask: 0.4
tags:

- MySQL

---


# 解决连接字符串中的特殊字符
在实用SQLAlchemy连接MySQL时，需要事先在配置文件或者代码中定义连接字符串，例如：
```
sql_alchemy_conn = mysql+mysqlconnector://airflow_user:Airflow1234@#$@localhost:3306/airflow
```

如果连接字符串中包含`@#$`等特殊字符，会报错，如下：
```
sqlalchemy.exc.InterfaceError: (mysql.connector.errors.InterfaceError) 2003: Can't connect to MySQL server on '%-.100s:%u' (%s) (Warning: %u format: a number is required, not str)
```

这是因为SQLAlchemy在处理连接字符串时，会`@#$`等特殊字符解析为分隔符，用于提取连接字符串中的用户名、密码、主机名、端口号等信息，而`@#$`等特殊字符在MySQL中是合法的密码字符，因此会报错。

此时正确的做法是分以下两步骤：

## 1. 将连接字符串中的特殊字符转义为URL编码
通过一些在线的URLEncoding网站，可以将密码编码成URL格式

```
Airflow1234@#$
Airflow1234%40%23%24
```
## 2. 将编码后的字符串中的百分号`%`转义为`%%`
```
Airflow1234%%40%%23%%24
```
再将转义后的字符串拼接到连接字符串中即可。
所以，正确的连接字符串应该是：
```
sql_alchemy_conn = mysql+mysqlconnector://airflow_user:Airflow1234%%40%%23%%24@localhost:3306/airflow
```

问题解决。