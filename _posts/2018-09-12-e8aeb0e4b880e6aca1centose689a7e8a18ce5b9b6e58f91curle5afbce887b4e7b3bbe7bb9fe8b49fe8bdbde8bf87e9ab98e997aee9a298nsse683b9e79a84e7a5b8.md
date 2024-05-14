---
id: 611
title: '记一次Centos执行并发curl导致系统负载过高问题:NSS惹的祸'
date: '2018-09-12T10:29:48+08:00'
author: huangdc
layout: post
guid: 'http://www.huangdc.com/?p=611'
permalink: /611
views:
    - '6101'
bigfa_ding:
    - '6'
image: /wp-content/uploads/2016/04/20160427125107-200x150.jpg
categories:
    - Linux
---

### 问题描述：

php开发新增定时任务每天中午12点都会通过并发curl请求https://\*\*facebook.com/\*\*\* ,导致系统cpu负载极高，系统告警了呀

### 问题排查：

**1、跟开发协商将并发的curl数量降低，但是问题依旧。那会是什么问题呢**

**2、用strace追踪curl执行吧,看看都干了什么**

**A、**先来统计一下系统调用 ，发现一堆access系统调用，而且接近全部都是错误的，尴尬了

```
strace -T -s 1024 -c curl "https://graph.facebook.com/v2.6/me/messages?*********"
```

[![](/assets/wp-content/uploads/2018/09/curl_nss.jpg)](/assets/wp-content/uploads/2018/09/curl_nss.jpg)

**B、**继续strace追踪看看access什么东西，发现一堆这些 /etc/pki/nssdb/\*\*\* , 如下所示

```
strace -T -s 1024 -e access curl "https://graph.facebook.com/v2.6/*****"
```

```
access("/etc/pki/nssdb/.2438912988_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000016>
access("/etc/pki/nssdb/.2438912989_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438912990_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438912991_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438912992_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000014>
access("/etc/pki/nssdb/.2438912993_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438912994_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000014>
access("/etc/pki/nssdb/.2438912995_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438912996_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438912997_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438912998_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438912999_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438913000_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438913001_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438913002_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438913003_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438913004_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438913005_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000014>
access("/etc/pki/nssdb/.2438913006_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438913007_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438913008_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438913009_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438913010_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438913011_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/etc/pki/nssdb/.2438913012_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000014>
access("/etc/pki/nssdb/cert9.db", F_OK) = 0 <0.000014>
access("/etc/pki/nssdb/cert9.db-journal", F_OK) = -1 ENOENT (No such file or directory) <0.000014>
access("/etc/pki/nssdb/cert9.db-wal", F_OK) = -1 ENOENT (No such file or directory) <0.000014>
access("/var/tmp", R_OK|W_OK|X_OK)      = 0 <0.000015>
access("/var/tmp/etilqs_F0dXOKTvM4QXZ2y", F_OK) = -1 ENOENT (No such file or directory) <0.000017>
access("/var/tmp/.2438911946_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000014>
access("/var/tmp/.2438911947_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/var/tmp/.2438911948_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/var/tmp/.2438911949_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/var/tmp/.2438911950_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/var/tmp/.2438911951_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000016>
access("/var/tmp/.2438911952_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000014>
access("/var/tmp/.2438911953_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/var/tmp/.2438911954_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/var/tmp/.2438911955_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000014>
access("/var/tmp/.2438911956_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000014>
access("/var/tmp/.2438911957_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/var/tmp/.2438911958_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/var/tmp/.2438911959_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000016>
access("/var/tmp/.2438911960_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/var/tmp/.2438911961_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/var/tmp/.2438911962_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>
access("/var/tmp/.2438911963_dOeSnotExist_.db", F_OK) = -1 ENOENT (No such file or directory) <0.000015>


......
```

NSS 是什么 ，NSS 是开源软件，和 OpenSSL 一样，是一个底层密码学库，包括 TLS 实现。网上很多反馈NSS内存泄漏，或者其他低版本的问题。

**C、**通过 curl –version 查看版本，会发现有个NSS

```
curl --version
##
curl 7.20.0 (x86_64-unknown-linux-gnu) libcurl/7.53.1 NSS/3.28.4 zlib/1.2.8 libidn2/0.16 libpsl/0.6.2 (+libicu/50.1.2) libssh2/1.4.2 nghttp2/1.21.1
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp scp sftp smb smbs smtp smtps telnet tftp
Features: AsynchDNS IDN IPv6 Largefile NTLM SPNEGO SSL libz
```

既然NSS有很多问题，那就把NSS换成opensSSL吧，重新编译curl ，编译参数加上 –without-nss 去掉NSS即可，加上 –with-ssl 修改为openSSL 。当然你要下载curl的软件包吧，自己下吧。编译参数如下

```
./configure --prefix=/usr/local --without-nss --with-ssl --enable-file --enable-http --enable-ftp --enable-ipv6 && make && make install
```

再次curl –version 查看版本，会发现NSS被替换成了openSSL了

```
curl --version

###
curl 7.20.0 (x86_64-unknown-linux-gnu) libcurl/7.20.0 OpenSSL/1.0.2k zlib/1.2.8 libidn/1.18
Protocols: dict file ftp ftps http https imap imaps pop3 pop3s rtsp smtp smtps telnet tftp
Features: IDN IPv6 Largefile NTLM SSL libz
```

最后检查并发curl也正常了

注意：curl都重新编译了，记得php也要重新编译哦

转载请注明：[Huangdc](https://www.huangdc.com) » [记一次Centos执行并发curl导致系统负载过高问题:NSS惹的祸](https://www.huangdc.com/611)