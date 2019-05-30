---
layout: post
title: "安装Python3.7过程中报错的解决方法"
subtitle: "How to install python3.7 with SSL error"
author: "qingshan"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.4
tags:
  - 工作
  - Python
---

最近工作中需要用到最新的Python3.7版在本地安装一切顺利，到准备在测试环境安装就发现了问题。装好python虚拟环境后，运行程序，发现报错。好像是与模块缺失相关。

```python
>>> import _ssl
Traceback (most recent call last):
 File "<stdin>", line 1, in ?
ImportError: No module named _ssl
```

### Round 1
虽然不清楚_ssl对应的是什么模块，但是感觉应该是和openssl有关系。搜索了一番，发现原来Python3.7默认的configuration里面把ssl相关的编译给注销了，需要手工取消注释。那么就重新编译吧。那么就先修改配置文件。
在python源代码目录的 Modules/Setup.dist文件里修改。 ***注意socket和ssl两部分都要取消注释！***
```shell
# Socket module helper for socket(2)
_socket socketmodule.c

# Socket module helper for SSL support; you must comment out the other
# socket line above, and possibly edit the SSL variable:
SSL=/usr/local/openssl   # 默认路径, 如果手动安装到别的路径, 这里需要改
_ssl _ssl.c \
    -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
    -L$(SSL)/lib -lssl -lcrypto
```

修改后重新编译
```shell
make clean && ./configurate && make && make install
```

但是又有新的报错：
```
./Modules/_ssl.c:73:6: error: #error "libssl is too old and does not support X509_VERIFY_PARAM_set1_host()"
./Modules/_ssl.c: In function ‘_ssl_configure_hostname’:
./Modules/_ssl.c:853: error: implicit declaration of function ‘SSL_get0_param’
./Modules/_ssl.c:853: warning: initialization makes pointer from integer without a cast
./Modules/_ssl.c:855: error: implicit declaration of function ‘X509_VERIFY_PARAM_set1_host’
./Modules/_ssl.c:861: error: implicit declaration of function ‘X509_VERIFY_PARAM_set1_ip’
./Modules/_ssl.c: In function ‘_ssl__SSLContext_impl’:
./Modules/_ssl.c:2947: error: ‘X509_CHECK_FLAG_NO_PARTIAL_WILDCARDS’ undeclared (first use in this function)
./Modules/_ssl.c:2947: error: (Each undeclared identifier is reported only once
./Modules/_ssl.c:2947: error: for each function it appears in.)
./Modules/_ssl.c:3052: error: implicit declaration of function ‘SSL_CTX_get0_param’
./Modules/_ssl.c:3052: warning: assignment makes pointer from integer without a cast
./Modules/_ssl.c:3058: error: implicit declaration of function ‘X509_VERIFY_PARAM_set_hostflags’
./Modules/_ssl.c: In function ‘get_verify_flags’:
./Modules/_ssl.c:3348: warning: assignment makes pointer from integer without a cast
./Modules/_ssl.c: In function ‘set_verify_flags’:
./Modules/_ssl.c:3361: warning: assignment makes pointer from integer without a cast
./Modules/_ssl.c: In function ‘set_host_flags’:
./Modules/_ssl.c:3524: warning: assignment makes pointer from integer without a cast
make: *** [Modules/_ssl.o] Error 1
```

### Round2
从报错第一行来看，是SSL版本太旧了，于是从官网下载最新的openssl进行默认安装：
```
wget -c http://www.openssl.org/source/openssl-1.0.2r.tar.gz
make
make test
make install
```

安装完之后重新编译Python3.7，问题还是没有解决，依然是提示"libssl is too old"

### Round3
从网上检索相关信息，看来应该还需要将openssl编译成**动态连接库**，并替换系统默认库，才可以生效。于是继续进行操作：
```shell
# 编译安装生成动态库
./config shared zlib-dynamic --prefix=/usr/local/openssl
make && make install

# 备份旧的版本
mv /usr/bin/openssl /usr/bin/openssl.old
mv /usr/include/openssl /usr/include/openssl.old

# 为新的版本建立软链
ln -s /usr/local/ssl/bin/openssl /usr/bin/openssl
ln -s /usr/local/ssl/include/openssl /usr/include/openssl

# 建立软链（因为 /usr/local/lib64/ 读取优先级高于 /usr/lib64/）
ln -s /usr/local/ssl/lib/libssl.so /usr/local/lib64/libssl.so

# 增加依赖的动态库
ln -s /usr/local/openssl/lib/libssl.so.1.1 /usr/lib64/libssl.so.1.1
ln -s /usr/local/openssl/lib/libcrypto.so.1.1 /usr/lib64/libcrypto.so.1.1

# 备份旧的库
mv /usr/lib64/libssl.so /usr/lib64/libssl.so.old
ln -s /usr/local/ssl/lib/libssl.so /usr/lib64/libssl.so
```

检查一下新版本是否生效
```shell
openssl version
OpenSSL 1.0.2r  26 Feb 2019
```

新版本openssl生效之后，再重新编译python前修改Python安装目录的Module/Setup文件：
```
#SSL=/usr/local/ssl
#_ssl _ssl.c \
#        -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
#        -L$(SSL)/lib -lssl -lcrypto
```

### Round4
再次make编译之后，发现新的提示：
```
Could not build the ssl module!
Python requires an OpenSSL 1.0.2 or 1.1 compatible libssl with X509_VERIFY_PARAM_set1_host().
LibreSSL 2.6.4 and earlier do not provide the necessary APIs, https://github.com/libressl-portable/portable/issues/381
```
大意是说LibreSSL版本也是太老了，二话不说，再次更新。
```shell
wget -c wget https://ftp.openbsd.org/pub/OpenBSD/LibreSSL/libressl-2.9.1.tar.gz
tar -xvf libressl-2.9.1.tar.gz
cd libressl-2.9.1
./configure && make && make install
```
再验证LibreSSL，确认已经更新到了目标版本：
```shell
#openssl version
LibreSSL 2.9.1
```

再次尝试编译，发现问题依然在。而且通过查询，发现LibreSSL只是OpenSSL的一个分支！因此OpenSSL如果装好的话，就不用装LibreSSL。

最终重新网上介绍的方法，配置LD_LIBRARY_PATH
```
export LD_LIBRARY_PATH=/usr/local/lib:/usr/local/lib64
sudo ldconfig
```
按照Round3的步骤，重新配置了环境变量和软链接，确认都替换到了openssl相关的库到最新，再尝试编译，问题解决。


### Python 安装前的系统准备
```shell
yum update
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel libffi-devel gcc make
```

### Python virtualenv pip自定义镜像源
默认的python pip比较慢，可以通过在pip.conf自定义镜像源。默认的pip.conf在
```shell
~/.pip/pip.conf
```

但是在virtualenv环境，就可以放在虚拟环境根目录下即可。
```
venv/pip.conf
```

国内较快的镜像使用阿里就可以了
```shell
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
```


如果还想再快一点，可以将上面的https换成http就会更快一点，但是https会更安全。
```shell
[global]
index-url = http://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
```



