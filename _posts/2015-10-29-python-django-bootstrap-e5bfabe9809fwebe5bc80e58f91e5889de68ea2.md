---
id: 21
title: 'python + django + bootstrap 快速web开发初探'
date: '2015-10-29T23:02:45+08:00'
author: huangdc
layout: post
guid: 'http://www.huangdc.com/?p=21'
permalink: /21
views:
    - '25765'
bigfa_ding:
    - '16'
duoshuo_thread_id:
    - '6214583362342880001'
categories:
    - Python
    - 技术杂谈
tags:
    - django
---

 **Python** 是一种面向对象、解释型计算机程序设计语言，由Guido van Rossum于1989年底发明，第一个公开发行版发行于1991年。Python语法简洁而清晰，具有丰富和强大的类库。

 **Django** 是一个开放源代码的Web应用框架，由Python写成。采用了MVC的软件设计模式，即模型M，视图V和控制器C。它最初是被开发来用于管理劳伦斯出版集团旗下的一些以新闻内容为主的网站的，即是CMS（内容管理系统）软件。

 **Bootstrap** 是Twitter推出的一个开源的用于前端开发的工具包。它由Twitter的设计师Mark Otto和Jacob Thornton合作开发，是一个CSS/HTML框架。Bootstrap提供了优雅的HTML和CSS规范，它即是由动态CSS语言Less写成。Bootstrap一经推出后颇受欢迎，一直是GitHub上的热门开源项目，包括NASA的MSNBC（微软全国广播公司）的Breaking News都使用了该项目。

初学初学python + django + bootstrap 快速web开发，有问题望大家指点，谢谢

下面我们来看看安装python/django/bootstrap及使用python + django + bootstrap开发一个最简单的web

## **1、源代码安装 Python：**

友好提示：在编译安装python 之前，我们最好先检查系统是否安装 readline-devel 安装包 ；为什么呢？ 因为很多朋友会遇到一个问题：通过编译安装的 python ，在使用 删除键（Backspace）为什么不能用？ 删除不了内容只会出来一个小方框是怎么回事，只能用Del来删除。移到像这的问题：

[![233250_Moy0_588586](/assets/wp-content/uploads/2015/10/233250_Moy0_588586.jpg)](/assets/wp-content/uploads/2015/10/233250_Moy0_588586.jpg)

<span style="color: #333333; font-family: 'Microsoft YaHei', Verdana, sans-serif, 宋体; font-size: 12.5px; line-height: 22.5px; background-color: #ffffff;">所以，如果通过编译安装python的时候，最好先安装好 </span><span style="color: #333333; font-family: 'Microsoft YaHei', Verdana, sans-serif, 宋体; font-size: 12.5px; line-height: 22.5px; background-color: #ffffff;">readline-devel 安装包：</span>

```
#
[root@localhost tools]# yum install readline-devel
```

<span style="color: #333333; font-family: 'Microsoft YaHei', Verdana, sans-serif, 宋体; font-size: 12.5px; line-height: 22.5px; background-color: #ffffff;">接着通过编译安装 python最新版 </span>

```
#
##查看系统默认安装python版本
[root@localhost tools]# python -V
Python 2.4.3
 
##这里安装现在最新版 python 2.7.8，下载并解压
[root@localhost tools]# wget https://www.python.org/ftp/python/2.7.8/Python-2.7.8.tgz
[root@localhost tools]# tar zxf Python-2.7.8.tgz
[root@localhost tools]# cd Python-2.7.8
 
##指定安装目录 /usr/local/python278/   加上--enable-shared 以后需要用到libpython 库
[root@localhost Python-2.7.8]# ./configure --enable-shared --prefix=/usr/local/python278/
[root@localhost Python-2.7.8]# make
[root@localhost Python-2.7.8]# make install
 
##查看 python执行文件路径,并将python软连为最新版本
[root@localhost Python-2.7.8]# which python
/usr/local/bin/python
[root@localhost Python-2.7.8]# ln -sf /usr/local/python278/bin/python /usr/local/bin/python
[root@localhost Python-2.7.8]# python -V
Python 2.7.8
```

## **2、安装Django**

<span style="color: #333333; font-family: 'Microsoft YaHei', Verdana, sans-serif, 宋体; font-size: 12.5px; line-height: 22.5px; background-color: #ffffff;">在安装django之前，我们先安装 setuptools 工具，不然在安装 django的时候汇报一个错误，如下：</span>

```
#
## 错误
[root@localhost Django-1.7.1]# python setup.py  install
Traceback (most recent call last):
  File "setup.py", line 4, in <module>
    from setuptools import setup, find_packages
ImportError: No module named setuptools
 
## 安装 setuptools ， 详情：https://pypi.python.org/pypi/setuptools
[root@localhost Django-1.7.1]# wget https://bootstrap.pypa.io/ez_setup.py --no-check-certificate -O - | python
```

<span style="color: #333333; font-family: 'Microsoft YaHei', Verdana, sans-serif, 宋体; font-size: 12.5px; line-height: 22.5px; background-color: #ffffff;">接着安装django</span>

```
##
##下载并解压,安装
[root@localhost tools]# wget https://www.djangoproject.com/download/1.7.1/tarball/
[root@localhost tools]# tar zxf Django-1.7.1.tar.gz
[root@localhost tools]# cd Django-1.7.1
[root@localhost Django-1.7.1]# python setup.py install
 
##检查安装是否成功
[root@localhost Django-1.7.1]# django-admin.py --version
1.7.1
## 检验 import django
[root@localhost Django-1.7.1]# python
Python 2.7.8 (default, Nov 14 2014, 01:25:43) 
[GCC 4.1.2 20080704 (Red Hat 4.1.2-54)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import django
>>> 
## 说明成功安装django
```

## **3、创建django 项目project和应用app**

```
##
[root@localhost data]# pwd
/data
## 创建 myproject 项目 ，及查看project目录结构
[root@localhost data]# django-admin.py startproject myproject
[root@localhost data]# ls
myproject  mysql
[root@localhost data]# cd myproject/ 
[root@localhost myproject]# ll
total 16
-rwxr-xr-x 1 root root  252 Nov 14 07:56 manage.py
drwxr-xr-x 2 root root 4096 Nov 14 07:56 myproject
 
## 在 myproject 项目中创建 myapp 应用， 及查看app目录结构
[root@localhost myproject]# django-admin.py startapp myapp
[root@localhost myproject]# ll 
total 24
-rwxr-xr-x 1 root root  252 Nov 14 07:56 manage.py
drwxr-xr-x 3 root root 4096 Nov 14 07:56 myapp
drwxr-xr-x 2 root root 4096 Nov 14 07:56 myproject
 
## 修改myproject的配置文件 settings.py  
[root@localhost myproject]# vim settings.py
## 1、在INSTALLED_APPS增加一个app，如下增加 myapp
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myapp',
)
##2、database 链接默认使用的是sqlite3，我们这里测试一下，先注释掉
DATABASES = {
    #'default': {
    #    'ENGINE': 'django.db.backends.sqlite3',
    #    'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    #}
}
##3、其他配置
LANGUAGE_CODE = 'zh-cn'
DEFAULT_CHARSET = 'utf-8'
FILE_CHARSET = 'utf-8'
TIME_ZONE = 'Asia/Shanghai'
```

## **4、测试 django 应用**

```
##
## 在myproject目录下，开启测试模式 8888 端口
[root@localhost myproject]# pwd
/data/myproject
[root@localhost myproject]# python manage.py runserver 0.0.0.0:8888
Performing system checks...
 
System check identified no issues (0 silenced).
November 14, 2014 - 16:03:08
Django version 1.7.1, using settings 'myproject.settings'
Starting development server at http://0.0.0.0:8888/
Quit the server with CONTROL-C.
```

<span style="color: #333333; font-family: 'Microsoft YaHei', Verdana, sans-serif, 宋体; font-size: 12.5px; line-height: 22.5px; background-color: #ffffff;">在浏览器可以访问，这里的url地址为 </span>[http://192.168.16.128:8888](http://192.168.16.128:8888/)<span style="color: #333333; font-family: 'Microsoft YaHei', Verdana, sans-serif, 宋体; font-size: 12.5px; line-height: 22.5px; background-color: #ffffff;"> , 如图说明已成功启用一个web app：</span>

![](http://7xntjj.com1.z0.glb.clouddn.com/000809_6Yrb_588586.png)

## **5、bootstrap 网站模板，及django配置**

```
##
##在项目目录(/data/myproject/)下创建一个 static 目录
[root@localhost myproject]# pwd
/data/myproject
[root@localhost myproject]# mkdir static
[root@localhost myproject]# ll
total 32
-rwxr-xr-x 1 root root  252 Nov 14 07:56 manage.py
drwxr-xr-x 3 root root 4096 Nov 14 07:56 myapp
drwxr-xr-x 2 root root 4096 Nov 14 08:05 myproject
drwxr-xr-x 2 root root 4096 Nov 16 17:46 static
 
## 将 bootstrap 文件下载到 static 目录并解压，得到css、fonts、js目录
[root@localhost static]# wget https://github.com/twbs/bootstrap/releases/download/v3.3.1/bootstrap-3.3.1-dist.zip
[root@localhost static]# unzip bootstrap-3.3.1-dist.zip
## 目录结构如下
[root@localhost static]# tree
.
|-- css
|   |-- bootstrap-theme.css
|   |-- bootstrap-theme.css.map
|   |-- bootstrap-theme.min.css
|   |-- bootstrap.css
|   |-- bootstrap.css.map
|   `-- bootstrap.min.css
|-- fonts
|   |-- glyphicons-halflings-regular.eot
|   |-- glyphicons-halflings-regular.svg
|   |-- glyphicons-halflings-regular.ttf
|   `-- glyphicons-halflings-regular.woff
`-- js
    |-- bootstrap.js
    |-- bootstrap.min.js
    `-- npm.js
 
3 directories, 13 files
```

<span style="color: #333333; font-family: 'Microsoft YaHei', Verdana, sans-serif, 宋体; font-size: 12.5px; line-height: 22.5px; background-color: #ffffff;">配置django</span>

```
##
## 1、修改myproject的配置文件 settings.py  
[root@localhost myproject]# vim settings.py
 
###配置静态文件路径
# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/1.6/howto/static-files/
STATIC_URL = '/static/'
STATICFILES_DIRS = (
        '/data/myproject/static/',
)
 
## 2、修改myproject的配置文件 urls.py
 
##添加一条 (r'^$', 'myapp.views.index'), 
##简单理解 当访问根时 ，由 myapp 的 views.py 中的 index 函数处理
##参数含义 这里先不做过多介绍
urlpatterns = patterns('',
    # Examples:
    # url(r'^$', 'myproject.views.home', name='home'),
    # url(r'^blog/', include('blog.urls')),
 
    url(r'^admin/', include(admin.site.urls)),
       (r'^$', 'myapp.views.index'),
)  
 
## 3、创建一个 templates文件：/data/myproject/myapp/
[root@localhost myapp]# mkdir /data/myproject/myapp/templates
## templates 是默认的模板目录名称
```

<span style="color: #333333; font-family: 'Microsoft YaHei', Verdana, sans-serif, 宋体; font-size: 12.5px; line-height: 22.5px; background-color: #ffffff;">下面是index.html 文件内容，路径为：/data/myproject/myapp/templates/index.html </span>

```

<!-- saved from url=(0050)http://getbootstrap.com/examples/starter-template/ -->
<html><head><meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="">
    <meta name="author" content="">
    <link rel="icon" href="http://getbootstrap.com/favicon.ico">
 
    <title>Starter Template for Bootstrap</title>
    <!-- Bootstrap core CSS -->
    <link href="/static/css/bootstrap.min.css" rel="stylesheet">
  </head>
 
  <body>
 
    <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">
      <div>
        <div>
          <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar" aria-expanded="false" aria-controls="navbar">
            <span>Toggle navigation</span>
            <span></span>
            <span></span>
            <span></span>
          </button>
          <a href="http://getbootstrap.com/examples/starter-template/#">Project name</a>
        </div>
        <div id="navbar" class="collapse navbar-collapse">
          <ul class="nav navbar-nav">
            <li><a href="http://getbootstrap.com/examples/starter-template/#">Home</a></li>
            <li><a href="http://getbootstrap.com/examples/starter-template/#about">About</a></li>
            <li><a href="http://getbootstrap.com/examples/starter-template/#contact">Contact</a></li>
          </ul>
        </div><!--/.nav-collapse -->
      </div>
    </nav>
 
    <div>
 
      <div>
        <h1>Bootstrap starter template</h1>
        <p>Use this document as a way to quickly start any new project.<br> All you get is this text and a mostly barebones HTML document.</p>
      </div>
 
    </div><!-- /.container -->
 
 
    <!-- jquery JavaScript -->
    <script src="/static/js/jquery.min.js"></script>
    <!-- Bootstrap core JavaScript -->
    <script src="/static/js/bootstrap.min.js"></script> 
   
 
</body></html>
```

<span style="color: #333333; font-family: 'Microsoft YaHei', Verdana, sans-serif, 宋体; font-size: 12.5px; line-height: 22.5px; background-color: #ffffff;">好了，最后来配置一下 myapp 的 视图文件 views.py</span>

```
###添加一个 我们前面提到的 index函数
[root@localhost myapp]# vim views.py
from django.shortcuts import render,render_to_response
from django.http import HttpResponse,HttpResponseRedirect
#coding=utf-8
#import time,simplejson,json,os,commands
 
# Create your views here.
def index(req):
        ##
        ##
        #return HttpResponse("aa")
        return render_to_response('index.html',)
```

<span style="color: #333333; font-family: 'Microsoft YaHei', Verdana, sans-serif, 宋体; font-size: 12.5px; line-height: 22.5px; background-color: #ffffff;">这样就ok 了，我们来启动测试模式，测试看看</span>

```
[root@localhost myproject]# pwd
/data/myproject
[root@localhost myproject]# python manage.py runserver 0.0.0.0:8888
Performing system checks...
 
System check identified no issues (0 silenced).
November 17, 2014 - 03:18:45
Django version 1.7.1, using settings 'myproject.settings'
Starting development server at http://0.0.0.0:8888/
Quit the server with CONTROL-C.
```

<span style="color: #333333; font-family: 'Microsoft YaHei', Verdana, sans-serif, 宋体; font-size: 12.5px; line-height: 22.5px; background-color: #ffffff;">ok，浏览器打开看看:</span>

![](http://7xntjj.com1.z0.glb.clouddn.com/112221_2Usk_588586.jpg)

相关学习链接：

简明python教程：<http://sebug.net/paper/python/>

Django教程：<http://djangobook.py3k.cn/2.0/>

bootstrap ：<http://getbootstrap.com/>

转载请注明：[Huangdc](https://www.huangdc.com) » [python + django + bootstrap 快速web开发初探](https://www.huangdc.com/21)

</body></html>