---
id: 103
title: 'nginx + python + uWSGI  环境搭建'
date: '2015-11-07T21:18:51+08:00'
author: huangdc
layout: post
guid: 'http://www.huangdc.com/?p=103'
permalink: /103
views:
    - '7971'
bigfa_ding:
    - '12'
duoshuo_thread_id:
    - '6214583362938471169'
categories:
    - Python
    - 技术杂谈
tags:
    - nginx
    - python
    - uwsgi
---

在了解 uWSGI 之前，我们不妨先了解一下

## **python http服务器？**

要使 Python 写的程序能在 Web 上被访问，还需要搭建一个支持 Python 的 HTTP 服务器，列举几个如 Gunicorn 、uWSGI 、FAPWS3、Aspen、Mod\_WSGI等等

## **WSGI是什么？**

WSGI，全称 Web Server Gateway Interface，或者 Python Web Server Gateway Interface ，是为 Python 语言定义的 Web 服务器和 Web 应用程序或框架之间的一种简单而通用的接口。自从 WSGI 被开发出来以后，许多其它语言中也出现了类似接口。

WSGI 的官方定义是，the Python Web Server Gateway Interface。从名字就可以看出来，这东西是一个Gateway，也就是网关。网关的作用就是在协议之间进行转换。

WSGI 是作为 Web 服务器与 Web 应用程序或应用框架之间的一种低级别的接口，以提升可移植 Web 应用开发的共同点。WSGI 是基于现存的 CGI 标准而设计的。

很多框架都自带了 WSGI server ，比如 Flask，webpy，Django、CherryPy等等。当然性能都不好，自带的 web server 更多的是测试用途，发布时则使用生产环境的 WSGI server或者是联合 nginx 做 uwsgi 。

好了，接下来看看

## **什么是uWSGI ？**

uWSGI是一个Web服务器，它实现了WSGI协议、uwsgi、http等协议。Nginx中HttpUwsgiModule的作用是与uWSGI服务器进行交换。

要注意 WSGI / uwsgi / uWSGI 这三个概念的区分。

a、WSGI看过前面小节的同学很清楚了，是一种通信协议。

b、uwsgi同WSGI一样是一种通信协议。

c、而uWSGI是实现了uwsgi和WSGI两种协议的Web服务器。

**为什么有了uWSGI为什么还需要nginx？**

因为nginx具备优秀的静态内容处理能力，然后将动态内容转发给uWSGI服务器，这样可以达到很好的客户端响应

## **部署配置**

1、python + django + bootstrap (略)

可查看：http://www.huangdc.com/21

2、下载并安装 **uWSGI**

```
[root@localhost tools]# wget --no-check-certificate http://projects.unbit.it/downloads/uwsgi-2.0.8.tar.gz
[root@localhost tools]# tar zxf uwsgi-2.0.8.tar.gz 
[root@localhost tools]# cd uwsgi-2.0.8
[root@localhost uwsgi-2.0.8]# make
[root@localhost uwsgi-2.0.8]# cp uwsgi /usr/bin/
```

3、nginx 配置

这里的项目路径是 /data/myproject/

```
## vim /usr/local/nginx/conf/nginx.conf
server {
        listen  80;
        server_name 192.168.16.128;
 
        location / {
            root /data/myproject/;
            include     uwsgi_params;
            uwsgi_pass   127.0.0.1:9000;
            uwsgi_param UWSGI_CHDIR  /data/myproject;
            uwsgi_param UWSGI_SCRIPT django_wsgi;
            access_log /usr/local/nginx/logs/access.log;
            }
 
        location /static {
                expires 30d;
                autoindex on;
                add_header Cache-Control provate;
                alias /data/myproject/static;
 
        }
}
  
## reload nginx
[root@localhost sbin]# service nignx reload
```

在nginx目录中添加一个uwsgi配置文件：

```
### vim /usr/local/nginx/conf/uwsgi.xml
 
<uwsgi>
 <socket>127.0.0.1:9000</socket>
 <listen>200 </listen>
 <master>true </master>
 <pidfile>/usr/local/nginx/uwsgi.pid </pidfile>
 <processes>8 </processes>
 <pythonpath>/data/myproject/ </pythonpath>
 <pythonpath>/data </pythonpath>
 <module>django_wsgi</module>
 <profiler>true </profiler>
 <memory-report>true </memory-report>
 <enable-threads>true </enable-threads>
 <logdate>true </logdate>
 <limit-as>6048 </limit-as>
 <daemonize>/dev/null</daemonize>
</uwsgi>
```

在项目目录下增加django\_wsgi.py 目录

```
##vim /data/myproject/django_wsgi.py
import os
os.environ['DJANGO_SETTINGS_MODULE'] = 'myproject.settings'
import django.core.handlers.wsgi
application = django.core.handlers.wsgi.WSGIHandler()
```

4、增加一个 uwsgi 启动文件

\## vim /etc/init.d/uwsgi

\## 记得加执行权限 chmod +x /etc/init.d/uwsgi

```
#!/bin/bash
# uwsgi script
# it is v.0.0.1 version.
# chkconfig: - 89 19
# description: uwsgi
# processname: uwsgi
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
 
uwsgi_config=/usr/local/nginx/conf/uwsgi.xml
uwsgi_pn=`ps aux|grep -v "grep"|grep -c "uwsgi"`
uwsgi_pid=`ps -eo pid,comm|grep uwsgi|sed -n 1p|awk '{print $1}'`
uwsgi_PID=/usr/local/nginx/logs/uwsgi.pid
uwsgi=/usr/bin/uwsgi
RETVAL=0
prog="uwsgi"
# Source function library.
.  /etc/rc.d/init.d/functions
 
 
if [ $(id -u) != "0" ]; then
    printf "Error: You must be root to run this script!\n"
    exit 1
fi
 
 
# Start nginx daemons functions.
start() {
if [ $uwsgi_pn -gt 5 ];then
        action "uwsgi is running!" /bin/true
    exit 0
fi
    daemon $uwsgi -x ${uwsgi_config}
        action "uwsgi start ..." /bin/true
}
# Stop nginx daemons functions.
stop() {
if [ $uwsgi_pn -gt 5 ]
then
        #kill -9 `ps -eo pid,comm|grep uwsgi|sed -n 1p|awk '{print $1}'`
        ps -eo pid,comm|grep uwsgi|awk '{print $1}' |xargs kill -9
    RETVAL=$?
        action "uwsgi stopping ..." /bin/true
else
        action "uwsgi not running!" /bin/false
fi
}
 
# See how we were called.
case "$1" in
start)
        start
        ;;
stop)
        stop
        ;;
reload)
        reload
        ;;
restart)
        stop
        start
        ;;
*)
        echo $"Usage: $prog {start|stop|restart}"
        exit 1
esac
exit $RETVAL
```

启动

```
[root@localhost init.d]# service uwsgi start
[uWSGI] parsing config file /usr/local/nginx/conf/uwsgi.xml
uwsgi start ...                                            [  OK  ]
[root@localhost conf]# netstat -ntlp |grep 9000
tcp        0      0 127.0.0.1:9000              0.0.0.0:*                   LISTEN      9426/uwsgi
```

转载请注明：[Huangdc](https://www.huangdc.com) » [nginx + python + uWSGI 环境搭建](https://www.huangdc.com/103)