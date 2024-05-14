---
id: 533
title: '记一次libpng版本导致的错误：imagepng(): gd-png: fatal libpng error: Incompatible libpng version in application and library'
date: '2017-06-22T10:40:10+08:00'
author: huangdc
layout: post
guid: 'http://www.huangdc.com/?p=533'
permalink: /533
views:
    - '10433'
bigfa_ding:
    - '11'
categories:
    - Linux
---

<div>记一次libpng版本导致的错误：imagepng(): gd-png: fatal libpng error: Incompatible libpng version in application and library</div><div>**1、收到开发同事反馈，在某台服务器上php调用imagepng()的时候报错，错误信息如下：**</div><div>```
libpng warning: Application was compiled with png.h from libpng-1.6.27
libpng warning: Application  is  running with png.c from libpng-1.2.49
PHP Fatal error:  imagepng(): gd-png: fatal libpng error: Incompatible libpng version in application and library
```

怎么会有2个libpng版本呢，gd-png？

**2、检查，查看系统libpng 及php 信息如下：**

```
[root@xxxx ~]# /usr/local/libpng/bin/libpng-config --version
1.6.27
[root@xxxx ~]# libpng-config --version
1.2.49
[root@xxxx ~]# php -v
PHP 7.1.1 (cli) (built: Feb  7 2017 14:36:00) ( NTS )
Copyright (c) 1997-2017 The PHP Group
Zend Engine v3.1.0, Copyright (c) 1998-2017 Zend Technologies

[root@xxxx ~]# ldd /usr/local/php/bin/php |grep libpng
        libpng16.so.16 => /usr/local/libpng/lib/libpng16.so.16 (0x00007f41dc7e2000)
        libpng12.so.0 => /usr/lib64/libpng12.so.0 (0x00007f41daa88000)

[root@xxxx ~]# rpm -qa |grep libpng
libpng-1.2.49-2.el6_7.x86_64
libpng-devel-1.2.49-2.el6_7.x86_64
```

<div>php版本为PHP 7.1.1 ，确实有2个版本的依赖库，根据错误信息可以知道，php调用的是libpng-1.2.49 （Application is running with png.c from libpng-1.2.49）</div><div>通过phpinfo() 信息查看到gd依赖的是libpng-1.6.27 ，因为我编译libgd的时候自定义了–prefix=/usr/local/libpng</div><div></div><div>好吧，那就是应该版本冲突所致，应该不需要重新编译php</div><div></div><div></div><div>**3、重新编译libgd库，依赖系统安装的libpng-1.2.49**</div><div>```
tar zxvf libgd-2.2.3.tar.gz  && cd libgd-2.2.3/ && ./configure --prefix=/usr/local/libgd --with-jpeg --with-png --with-freetype=/usr/local/freetype && make && make install ;
```

再次查看phpinfo() 信息查看到gd依赖的是libpng-1.2.49 ，开发也反馈正常了

</div></div><div></div><div>留下一个问题，在编译php的时候 ，带了这个参数 –with-png-dir=/usr/local/libpng ，/usr/local/libpng的版本是libpng-1.6.27 ，怎么还会有libpng-1.2.49呢？</div><div></div><div></div><div></div>转载请注明：[Huangdc](https://www.huangdc.com) » [记一次libpng版本导致的错误：imagepng(): gd-png: fatal libpng error: Incompatible libpng version in application and library](https://www.huangdc.com/533)