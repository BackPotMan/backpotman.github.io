---
id: 96
title: 'python + flask + uwsgi + gevent + nginx 环境搭建（非阻塞）'
date: '2015-11-04T22:00:36+08:00'
author: huangdc
layout: post
guid: 'http://www.huangdc.com/?p=96'
permalink: /96
views:
    - '9994'
duoshuo_thread_id:
    - '6214583362846196481'
bigfa_ding:
    - '11'
categories:
    - Python
    - 技术杂谈
tags:
    - flask
---

Flask是Python中一个微型的Web开发框架。在debug 模式 或 单纯的 uwsgi模式下，flask是阻塞模式的，也就是说一次只能效应一个请求，或者在uwsgi 开启多进程，响应已知的请求个数；我们这里使用 uwsgi 和 gevent 配合nginx 解决flask的阻塞模式。

## <span style="font-size: 18pt;">**1、环境**</span>

CentOS Linux release 7.0.1406 (Core)

Python 2.7.5

## <span style="font-size: 18pt;">**2、安装类库**</span>

```
yum install python-devel zlib-devel bzip2-devel pcre-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc
```

## <span style="font-size: 18pt;">**3、下载并安装python 包管理软件 pip** </span>

查看python 版本

```
[root@localhost ~]# python -V
Python 2.7.5
```

下载 pip-7.1.0.tar.gz

```
wget https://pypi.python.org/packages/source/p/pip/pip-7.1.0.tar.gz#md5=d935ee9146074b1d3f26c5f0acfd120e

```

解压安装

```
tar zxf pip-7.1.0.tar.gz
cd pip-7.1.0
python setup.py install
 
## 提示下列信息，，且无报错，说明安装成功
Installed /usr/lib/python2.7/site-packages/pip-7.1.0-py2.7.egg
Processing dependencies for pip==7.1.0
Finished processing dependencies for pip==7.1.0
```

下载 setuptools-18.1.tar.gz

```
wget https://pypi.python.org/packages/source/s/setuptools/setuptools-18.1.tar.gz#md5=f72e87f34fbf07f299f6cb46256a0b06

## 解压安装
tar zxf setuptools-18.1.tar.gz 
cd setuptools-18.1
python setup.py install
 
## 提示下列信息，，且无报错，说明安装成功
Installed /usr/lib/python2.7/site-packages/setuptools-18.1-py2.7.egg
Processing dependencies for setuptools==18.1
Finished processing dependencies for setuptools==18.1
```

## <span style="font-size: 18pt;">**4、安装 flask / uwsgi / gevent / nignx** </span>

```
### 安装 flask
pip install flask
 
## 提示下列信息，，且无报错，说明安装成功
Successfully installed Jinja2-2.8 MarkupSafe-0.23 Werkzeug-0.10.4 flask-0.10.1 itsdangerous-0.24

### 安装uwsgi
pip install uwsgi
 
## 提示下列信息，，且无报错，说明安装成功
Collecting uwsgi
  Using cached uwsgi-2.0.11.1.tar.gz
Installing collected packages: uwsgi
  Running setup.py install for uwsgi
Successfully installed uwsgi-2.0.11.1

###  安装gevent
pip install gevent
## 提示下列信息，，且无报错，说明安装成功
  Running setup.py install for greenlet
  Running setup.py install for gevent
Successfully installed gevent-1.0.2 greenlet-0.4.7

### 下载 并 编译安装nginx 
useradd www
wget http://nginx.org/download/nginx-1.8.0.tar.gz
tar zxf nginx-1.8.0.tar.gz 
cd nginx-1.8.0
 
./configure --prefix=/usr/local/nginx --user=www --group=www --with-http_stub_status_module --with-http_ssl_module
make
make install
```

## <span style="font-size: 18pt;">**5、 新建flask app 文件 /data/wwwroot/myapp.py**</span>

```
[root@localhost ~]# mkdir /data/wwwroot -p
[root@localhost ~]# cd /data/wwwroot/
[root@localhost wwwroot]# cat myapp.py
from flask import Flask
app = Flask(__name__)
 
@app.route('/')
def hello_world():
    return 'Hello World!'
 
if __name__ == '__main__':
    app.run()
```

## <span style="font-size: 18pt;">**6、配置 uWSGI 及 gevent** </span>

（1）配置 uwsgi.ini 文件 vim /usr/local/nginx/conf/uwsgi.ini

```
[uwsgi]
    chmod-socket = 666
    socket = /tmp/uwsgi.sock
    master = true
    pidfile = /var/run/uwsgi.pid
    processes = 4
    workers = 1
    pythonpath = /data/wwwroot  ##flask app 目录
    pythonpath = /data/
    callable = app
    profiler= true
    memory-report=true
    enable-threads = true
    logdate=true
    limit-as=6048
    daemonize=/data/wwwroot/uwsgi_sapublish.log
    gevent = 100       ## 加入 gevent = 100 ，非阻塞模式
```

（2） 再次配置 cat /data/wwwroot/myapp.py

```
## 在文件头加入下面两行
from gevent import monkey
monkey.patch_all()
 
from flask import Flask
app = Flask(__name__)
 
@app.route('/')
def hello_world():
    return 'Hello World!'
 
if __name__ == '__main__':
    app.run()
```

（3）启动 uwsgi ，当然你可以自己写个启动脚本，这里测试就不写了

```
/usr/bin/uwsgi --ini /usr/local/nginx/conf/uwsgi.ini
```

## <span style="font-size: 18pt;">**7、配置nginx**</span>

（1）添加配置文件 cat /usr/local/nginx/conf/vhosts/uwsgi\_flask.conf

```
server {
     listen       8082 default_server;
     server_name  192.168.1.105 ;

     location / {
         #uwsgi_pass 127.0.0.1:5000;
         include uwsgi_params;
         uwsgi_pass unix:/tmp/uwsgi.sock;
         uwsgi_param UWSGI_CHDIR  /data/wwwroot/;
         uwsgi_param UWSGI_SCRIPT myapp; ### myapp.py app文件名称
         access_log /data/wwwroot/app_access.log;
        }

}
```

（2）修改nginx ；加入include vhosts/uwsgi\_flask.conf;

（3）启动nginx

```
cd /usr/local/nginx/sbin/
./nginx
```

转载请注明：[Huangdc](https://www.huangdc.com) » [python + flask + uwsgi + gevent + nginx 环境搭建（非阻塞）](https://www.huangdc.com/96)