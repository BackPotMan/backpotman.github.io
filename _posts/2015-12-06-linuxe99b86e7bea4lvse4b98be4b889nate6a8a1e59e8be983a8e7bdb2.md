---
id: 199
title: Linux集群LVS之三NAT模型部署
date: '2015-12-06T13:13:19+08:00'
author: huangdc
layout: post
guid: 'http://www.huangdc.com/?p=199'
permalink: /199
views:
    - '3748'
duoshuo_thread_id:
    - '6225038874477855490'
bigfa_ding:
    - '6'
categories:
    - Linux
    - LVS
tags:
    - nat
---

大家都知道从**Linux2.4** 内核以后，已经完全内置了LVS的各个功能模块ipvs 。 ipvs 是提供 LVS 功能的核心模块，ipvsadm 实际上是linux系统中为用户提供的 ipvs的管理工具 ；类似大家所熟悉的 iptables 实质上是linux系统中为用户提供的 netfilter管理工具，linux 包过滤是通过内核来实现的 , 内核是使用netfilter架构来实现防火墙能力的

在开始nat 模型实验前，先来讲一下 ipvsadm 管理工具 。

## ipvsadm 管理工具

**1、安装ipvsadm**

```
## 安装ipvsadm
yum -y install ipvsadm

## 查看
rpm -qa |grep ipvsadm
```

**2、简单man ipvsadm 查看一下手册**

```
#### 这里贴出部分
NAME
       ipvsadm - Linux Virtual Server administration

SYNOPSIS
       ipvsadm -A|E -t|u|f service-address [-s scheduler]
               [-p [timeout]] [-M netmask]
       ipvsadm -D -t|u|f service-address
       ipvsadm -C
       ipvsadm -R
       ipvsadm -S [-n]
       ipvsadm -a|e -t|u|f service-address -r server-address
               [-g|i|m] [-w weight] [-x upper] [-y lower]
       ipvsadm -d -t|u|f service-address -r server-address
       ipvsadm -L|l [options]
       ipvsadm -Z [-t|u|f service-address]
       ipvsadm --set tcp tcpfin udp
       ipvsadm --start-daemon state [--mcast-interface interface]
               [--syncid syncid]
       ipvsadm --stop-daemon state
       ipvsadm -h
```

**3、ipvsadm 命令参考**

**（1）、管理集群服务**

添加 ： ipvsadm -A -t|u|f service-address \[-s scheduler\]

-t ：TCP协议的集群

-u：UDP协议的机器

-f : FWM防火墙标记

修改：ipvsadm -E -t|u|f service-address \[-s scheduler\]

删除：ipvsadm -D -t|u|f service-address

\## ipvsadm -A -t 192.168.49.131:80 -s rr

**（2）、管理集群服务中的RS**

添加：ipvsadm -a -t|u|f service-address -r server-address \[-g|i|m\] \[-w weight\] \[-x upper\] \[-y lower\]

-t|u|f service-address ：之前定义的集群服务

-r server-address ： 后端RS的地址，在NAT模型中可以使用ip:port实现端口映射

\[-g|i|m\] ：LVS类型

-g ： DR

-i ： TUN

-m ： NAT

\[-w weight\] ：定义服务器权重

修改：ipvsadm -e -t|u|f service-address -r server-address \[-g|i|m\] \[-w weight\] \[-x upper\] \[-y lower\]

删除：ipvsadm -d -t|u|f service-address -r server-address

\## ipvsadm -a -t 192.168.49.131:80 -r 192.168.1.103 -m

\## ipvsadm -a -t 192.168.49.131:80 -r 192.168.1.104 -m

**（3）、查看**

ipvsadm -L|l \[options\]

-n : 数字格式显示主机地址和端口

–stats ： 统计数据

–rate ： 速率

–timeout ： 显示tcp 、tcpfin 和 udp 的会话超时时长

**（4）、删除所有集群服务**

ipvsadm -C ： 清空ipvs 规则

**（5）、保存规则**

ipvsadm -S &gt; /path/to/somefile ： 保存ipvs 规则

**（6）、载入规则**

ipvsadm -R &lt; /path/to/somefile ： 载入ipvs 规则

好了，ipvsadm 命令大体了解，下面来实现NAT模型

## NAT环境

系统内核及网络信息：

DS\_LB ：CentOS release 6.7 (Final) / 2.6.32-573.el6.x86\_64 (两张网卡)  
eth0：192.168.49.134 （VIP）  
eth1：192.168.1.105（内网）  
real server1 : 192.168.1.103 getway：192.168.1.105  
real server2 : 192.168.1.104 getway：192.168.1.105

拓扑图：

[![nat20151206123604](/assets/wp-content/uploads/2015/12/nat20151206123604.png)](/assets/wp-content/uploads/2015/12/nat20151206123604.png)

## RS 客户端配置

1、安装 web 服务 httpd

```
yum install httpd
```

2、检查网络信息，重点：检查Real Server 的网关，必须指向DS 的内网ip

```
## 查看RS01 网关
[root@RS01 ~]# cat /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=RS01
GATEWAY=192.168.1.105

## 查看RS02 网关
[root@RS02 ~]# cat /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=RS02
GATEWAY=192.168.1.105
```

3 、 配置web 服务

```
## RS01 index.html
[root@RS01 html]# cat /var/www/html/index.html 
</br>
welcome to www.huangdc.com
</br>
NAT : WEB01

## RS01 httpd 服务启动
[root@RS01 html]# service httpd start
Start httpd:                                            [  OK  ]


## RS02 index.html
[root@RS02 html]# cat /var/www/html/index.html 
</br>
welcome to www.huangdc.com
</br>
NAT : WEB02

## RS02 httpd 服务启动
[root@RS02 html]# service httpd start
Start httpd:                                            [  OK  ]
```

访问看看：

[![web0120151206130717](/assets/wp-content/uploads/2015/11/web0120151206130717.png)](/assets/wp-content/uploads/2015/11/web0120151206130717.png)[![web0220151206130717](/assets/wp-content/uploads/2015/11/web0220151206130717.png)](/assets/wp-content/uploads/2015/11/web0220151206130717.png)

## DS 服务器端配置

1、检查VS服务

```
[root@DS_LB ~]# grep -i 'ip_vs' /boot/config-2.6.32-573.el6.x86_64
CONFIG_IP_VS=m
CONFIG_IP_VS_IPV6=y
# CONFIG_IP_VS_DEBUG is not set
CONFIG_IP_VS_TAB_BITS=12
CONFIG_IP_VS_PROTO_TCP=y
CONFIG_IP_VS_PROTO_UDP=y
CONFIG_IP_VS_PROTO_AH_ESP=y
CONFIG_IP_VS_PROTO_ESP=y
CONFIG_IP_VS_PROTO_AH=y
CONFIG_IP_VS_PROTO_SCTP=y
CONFIG_IP_VS_RR=m
CONFIG_IP_VS_WRR=m
CONFIG_IP_VS_LC=m
CONFIG_IP_VS_WLC=m
CONFIG_IP_VS_LBLC=m
CONFIG_IP_VS_LBLCR=m
CONFIG_IP_VS_DH=m
CONFIG_IP_VS_SH=m
CONFIG_IP_VS_SED=m
CONFIG_IP_VS_NQ=m
CONFIG_IP_VS_FTP=m
CONFIG_IP_VS_PE_SIP=m
```

可以看到支持的协议和算法模块

2、打开路由转发功能

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

3、配置ipvs服务及添加real server

```
## 添加一个vs服务 并 查看
[root@DS_LB ~]# ipvsadm -A -t 192.168.49.134:80 -s rr
[root@DS_LB ~]# ipvsadm -L
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.49.134:http rr
[root@DS_LB ~]# 

## 添加real server 
[root@DS_LB ~]# ipvsadm -a -t 192.168.49.134:80 -r 192.168.1.103:80 -m
[root@DS_LB ~]# ipvsadm -a -t 192.168.49.134:80 -r 192.168.1.104:80 -m
[root@DS_LB ~]# 
[root@DS_LB ~]# ipvsadm -L
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.49.134:http rr
  -> 192.168.1.103:http           Masq    1      0          0         
  -> 192.168.1.104:http           Masq    1      0          0         
[root@DS_LB ~]# 
```

配置好了，访问看看:

[![natrr](/assets/wp-content/uploads/2015/11/natrr.png)](/assets/wp-content/uploads/2015/11/natrr.png)

可以看到，我们用 rr 轮询算法，访问刷新都是循环分配到 web01 和 web02

4、修改 rr 算法为 wrr

```
## 修改 为 wrr 算法
[root@DS_LB ~]# ipvsadm -E -t 192.168.49.134:80 -s wrr
[root@DS_LB ~]# ipvsadm -L
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
 -> RemoteAddress:Port Forward Weight ActiveConn InActConn
TCP 192.168.49.134:http wrr
 -> 192.168.1.103:http Masq 1 0 0 
 -> 192.168.1.104:http Masq 1 0 1 
[root@DS_LB ~]# 

## 修改权重
[root@DS_LB ~]# ipvsadm -e -t 192.168.49.134:80 -r 192.168.1.103:80 -m -w 20
[root@DS_LB ~]# ipvsadm -e -t 192.168.49.134:80 -r 192.168.1.104:80 -m -w 10
[root@DS_LB ~]# 
[root@DS_LB ~]# ipvsadm -L
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
 -> RemoteAddress:Port Forward Weight ActiveConn InActConn
TCP 192.168.49.134:http wrr
 -> 192.168.1.103:http Masq 20 0 0 
 -> 192.168.1.104:http Masq 10 0 0 
[root@DS_LB ~]# 

```

再次访问试试看 ：

[![natwrr](/assets/wp-content/uploads/2015/11/natwrr.png)](/assets/wp-content/uploads/2015/11/natwrr.png)

可以看到 192.168.1.130 的权重比较大，当访问3次时，有2次转发到了web01 ，1次转发到 web02

好了，nat 模型到这里，ths

转载请注明：[Huangdc](https://www.huangdc.com) » [Linux集群LVS之三NAT模型部署](https://www.huangdc.com/199)