---
layout: post
title: "一些实用的Python 片段"
subtitle: "Some Useful Python Snippets"
author: "qingshan"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.4
tags:
  - HTTPS
  - Linux
---

1. 域名转 IP
2. 查看指定进程状态
3. 启动/停止/重启进程

```python
# coding=utf-8


import socket
import requests
import os
import random
import commands, time


# 下载网页
def download(url):
    res = requests.get(url)
    return res.text


# 域名转IP
def host_to_ip(host):
    '''
    Returns the IP address of a given hostname

    CLI Example:

    .. code-block:: bash

        salt '*' network.host_to_ip example.com
    '''
    try:
        family, socktype, proto, canonname, sockaddr = socket.getaddrinfo(
            host, 0, socket.AF_UNSPEC, socket.SOCK_STREAM)[0]

        if family == socket.AF_INET:
            ip, port = sockaddr
        elif family == socket.AF_INET6:
            ip, port, flow_info, scope_id = sockaddr
    except Exception:
        ip = None
    return ip


def service_status(service_name):
    cmd = "ps -ef |grep \"%s\"|grep -v grep |awk '{print $2}'" % service_name
    result = os.popen(cmd).read().strip()
    try:
        service_pid = result.split()[0]
        if service_pid:
            print "\033[32;1m%s monitor service is running...\033[0m" % service_name
            return "Running"
    except IndexError:
        print "\033[31;1m%s service is not running....\033[0m" % service_name
        return "Dead"


def stop_service(ss_config='all', local_port=0, customize=""):
    if customize:
        script = customize
    elif ss_config == 'all':
        script = "ss-local"
    else:
        server, port, pwd, method = ss_config
        script = "ss-local -s {0} -p {1} -k {2} -m {3} -l {4}".format(server, port, pwd, method, local_port)
    service_name = script

    cmd = "ps -ef| grep \"%s\"|grep -v grep |awk '{print $2}'|xargs kill -9" % service_name

    if service_status(service_name) == 'Running':
        cmd_result = os.system(cmd)
        if cmd_result == 0:
            print '..............\n'
            time.sleep(1)
            print '\033[31;1m%s stopped! \033[0m' % service_name
        else:
            print '\033[31;1mCannot stop %s service successfully,please manually kill the pid!\033[0m' % service_name
    else:
        print 'Service is not running...,nothing to kill! '


def start_service(ss_config, local_port):
    server, port, pwd, method = ss_config
    script = "ss-local -s {0} -p {1} -k {2} -m {3} -l {4}".format(server, port, pwd, method, local_port)
    print "Checking service status....."
    if service_status(script) == 'Running':
        print "\033[33;1mNetflow service is already running!\033[0m\n"
    else:
        print "Starting service...."
        cmd = 'nohup %s >> phantom.log 2>&1 &' % script
        print cmd

        if os.system(cmd) == 0:
            time.sleep(5)
            print '\033[32;1mNetflow service started successfully!\n\033[0m'
        else:
            print '\033[32;1mNetflow service started failed!\n\033[0m'


def restart_service(service_name, ss_config, local_port):
    server, port, pwd, method = ss_config
    script = "ss-local -s {0} -p {1} -k {2} -m {3} -l {4}".format(server, port, pwd, method, local_port)
    print "Checking service status....."
    if service_status(service_name) == 'Running':
        print "\033[33;1mNetflow service is already running!\033[0m\n"
    else:
        print "Starting service...."
        cmd = 'nohup %s >> phantom.log 2>&1 &' % script
        print cmd

        if os.system(cmd) == 0:
            time.sleep(5)
            print '\033[32;1mNetflow service started successfully!\n\033[0m'
        else:
            print '\033[32;1mNetflow service started failed!\n\033[0m'


def check_ss_live(ss_config):
    server, port, pwd, method = ss_config
    local_port = 8800 + random.randrange(10, 99)

    start_service(ss_config, local_port)

    cmd = "curl --socks5 127.0.0.1:{} http://ip111cn.appspot.com".format(local_port)

    start = time.time()
    result = commands.getstatusoutput(cmd)
    stop = time.time()

    print(stop - start)

    stop_service(ss_config, local_port)

    if server in result[1]:
        print ss_config, "OK!"
        return stop - start, ss_config
    else:
        return False, ss_config

```

