---
id: 201
title: Linux集群LVS之四DR模型部署
date: '2015-12-07T08:00:04+08:00'
author: huangdc
layout: post
guid: 'http://www.huangdc.com/?p=201'
permalink: /201
duoshuo_thread_id:
    - '6225194489984582402'
views:
    - '5115'
bigfa_ding:
    - '6'
categories:
    - Linux
    - LVS
tags:
    - lvs
---

大家好哈，前面已经讲了 ipvsadm 和 nat 模式搭建，本人将搭建DR模型 。大家都知道在公司使用最多的是DR 模型，DR支持大规模的网络负载均衡

## DR环境

系统内核及网络信息：

DS\_LB ：CentOS release 6.7 (Final) / 2.6.32-573.el6.x86\_64  
eth0：192.168.1.105  
**eth0:0**：192.168.1.107 (VIP)  
real server1 :  
eth0：192.168.1.103  
**lo:0**：192.168.1.107 (VIP)  
real server2 :  
eth0：192.168.1.104  
**lo:0**：192.168.1.107 (VIP)

拓扑图：

[![dr](/assets/wp-content/uploads/2015/12/dr.png)](/assets/wp-content/uploads/2015/12/dr.png)

## RS 客户端配置

1、real server 配置vip网络 及 开启转发功能等

```
## 在real server 01 和 real server 02 运行一下命令

VIP=192.168.1.107
/sbin/ifconfig lo:0 $VIP broadcast $VIP netmask 255.255.255.255 up
/sbin/route add -host $VIP dev lo:0
echo "1" > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo "2" > /proc/sys/net/ipv4/conf/lo/arp_announce
echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce

## ifconfig 可查看到信息
##lo:0 Link encap:Local Loopback 
## inet addr:192.168.1.107 Mask:255.255.255.255
## UP LOOPBACK RUNNING MTU:65536 Metric:1

```

2、启动 web 服务 httpd 及 配置web

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

3、单独测试访问 real server 的web服务

[![web0120151206130717](/assets/wp-content/uploads/2015/11/web0120151206130717.png)](/assets/wp-content/uploads/2015/11/web0120151206130717.png) [![web0220151206130717](/assets/wp-content/uploads/2015/11/web0220151206130717.png)](/assets/wp-content/uploads/2015/11/web0220151206130717.png)

## DS 服务器端配置

1、Director server 配置vip 网络

```
## 配置vip 及 路由
VIP=192.168.1.107
/sbin/ifconfig eth0:0 $VIP broadcast $VIP netmask 255.255.255.255 up
/sbin/route add -host $VIP dev eth0:0

## 查看
[root@DS_LB ~]# ifconfig eth0:0
eth0:0 Link encap:Ethernet HWaddr 00:0C:29:8A:4A:DE 
 inet addr:192.168.1.107 Bcast:192.168.1.107 Mask:255.255.255.255
 UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
```

2、ipvsadm命令设置DR模式负载均衡群集

```
## 清空ipvs规则 ,添加集群服务及添加后端realserver
[root@DS_LB ~]# ipvsadm -C
[root@DS_LB ~]# ipvsadm -A -t 192.168.1.107:80 -s rr
[root@DS_LB ~]# ipvsadm -a -t 192.168.1.107:80 -r 192.168.1.103:80 -g
[root@DS_LB ~]# ipvsadm -a -t 192.168.1.107:80 -r 192.168.1.104:80 -g
[root@DS_LB ~]# ipvsadm -L
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.1.107:http rr
  -> 192.168.1.103:http           Route   1      0          0         
  -> 192.168.1.104:http           Route   1      0          0         
[root@DS_LB ~]# 

## 保存规则
[root@DS_LB ~]# ipvsadm -S > /etc/sysconfig/ipvsadm
[root@DS_LB ~]# cat /etc/sysconfig/ipvsadm
-A -t 192.168.1.107:http -s rr
-a -t 192.168.1.107:http -r 192.168.1.103:http -g -w 1
-a -t 192.168.1.107:http -r 192.168.1.104:http -g -w 1

```

配置好了，在**两台**不同的设备 测试访问看看:

[![natrr](/assets/wp-content/uploads/2015/11/natrr.png)](/assets/wp-content/uploads/2015/11/natrr.png)

好了，dr 说完了

转载请注明：[Huangdc](https://www.huangdc.com) » [Linux集群LVS之四DR模型部署](https://www.huangdc.com/201)