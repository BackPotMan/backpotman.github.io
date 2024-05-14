---
id: 180
title: Linux集群LVS之一LVS类型详解
date: '2015-11-29T13:04:30+08:00'
author: huangdc
layout: post
guid: 'http://www.huangdc.com/?p=180'
permalink: /180
views:
    - '5603'
duoshuo_thread_id:
    - '6222383770834567938'
bigfa_ding:
    - '8'
categories:
    - Linux
    - LVS
tags:
    - lvs
---

大家好哈，LVS 就是Linux虚拟服务器（Linux Virtual Server），在1998年5月，由章文嵩博士成立了Linux Virtual Server的自由软件项目，进行Linux服务器集群的开发工作。同时，Linux Virtual Server项目是国内最早出现的自由软件项目之一。官方站点：<http://www.linuxvirtualserver.org/> 大家可以到官方站点详细了解

Linux Virtual Server项目的目标 ：使用集群技术和Linux操作系统实现一个高性能、高可用的服务器，它具有很好的可伸缩性（Scalability）、可靠性（Reliability）和可管理性（Manageability）。

目前 LVS 已经是Linux 标准内核的一部分，在**Linux2.4**内核以前，使用LVS时必须要重新编译内核以支持LVS功能模块，但是从**Linux2.4** 内核以后，已经完全内置了LVS的各个功能模块，无需给内核打任何补丁，可以直接使用LVS提供的各种功能。

在说lvs 类型之前我们可以了解一下Linux 集群可以分为：

1、**Load Balancing** ， LB 负载均衡 ，重点考虑的是 并发处理能力 （比如有lvs 、haproxy）  
2、**High Availability** ，HA 高可用 ， 重点考虑的是 在线可用服务时间，我们经常听到的所为提供的99%，99.9% , 99.99% …. 多少个9 的服务标准 （比如有heartbeat、keepalived、RHCS）  
3、**High Performance** ，HP 高性能 ， 重点考虑的是 并行处理计算能力 （比如…）

这里要讲的LVS 就属于Load Balancing 负载均衡技术 , 负载均衡技术有很多实现方案，有基于DNS域名轮流解析的方法、有基于客户端调度访问的方法、有基于应用层系统负载的调度方法，还有基于IP地址的调度方法，在这些负载调度算法中，执行效率最高的是IP负载均衡技术

## LVS类型

LVS-NAT：Network address translation （地址转换协议）（类DNAT）

LVS-DR：Direct routing （直接路由）（通常配置使用的模型）

LVS-TUN：IP tunneling （隧道）

## VS/NAT 地址转换协议模型

[![nat_20151129103110](/assets/wp-content/uploads/2015/11/nat_20151129103110.png)](/assets/wp-content/uploads/2015/11/nat_20151129103110.png)

Real Server ：后端真实提供应用服务器 ， 这里用R1、R2 、R3 表示，ip 地址用 RIP 表示 ，，后续雷同

Directior Server : LVS 服务器 ipvs ， 这里用 Director 表示， ip 用 DIP 表示 ，，，VIP 表示 ，， 后续雷同

好，我们来看看， 如图当一个用户访问进来后，报文头在流程中的变化，修改源ip和目标ip

**VS/NAT 流程：**

1、首先，用户请求访问的ip报文 的源ip为用户的 CIP ，目标ip是 VIP

2、数据包到达 Director Server（DS） 后，DR 将源ip 改为Director Server的DIP ，目标ip 改为Real Server（RS） 的RIP ，转发到RS

3、数据包到达RS后并经过处理，RS 将源ip 改为 RIP ，目标ip 改为 DIP ，发送到DS

4、最后DS 将源ip 改为 VIP ，目标ip 改为CIP ，响应发送给客户端

可以看到，数据包入站出站都需要经过 DS ， 接下来我们看看 VS/NAT 模型的特点

**VS/NAT 特点：**

1、集群节点必须要同在一个**IP网络**中

2、RIP地址通常都是私有地址，仅用于各集群节点间通信

3、Director 位于Client和Real Server之间，负责处理进出的所有通信

4、Real Server 的网关必须指向DIP

5、支持端口映射

6、Real Server 可以使用任意的操作系统

7、较大规模应用场景中，Director Server 是整个架构中的瓶颈

## VS/DR 直接路由模型

[![DR_20151129103448](/assets/wp-content/uploads/2015/11/DR_20151129103448.png)](/assets/wp-content/uploads/2015/11/DR_20151129103448.png)

**VS/DR 流程：**

1、首先，用户访问请求的ip报文 的源ip为用户的 CIP ，目标ip是 VIP ；源mac地址为 CMAC ，目标mac地址为 DMAC

2、数据包到达DS后，源ip和目标ip不变，仅仅修改目标mac地址为 RMAC ，并转发到RS

3、RS收到数据包并经过处理，RS 将源ip 改为 VIP ，目标ip 改为 CIP ； 源mac地址改为 RMAC ，目标mac地址改为CMAC ，直接响应发送给客户端

可以看到，数据包入站需要经过 DS ，但是数据包在RS 是直接发送出去的， 看看 VS/DR 模型的特点

**VS/DR 特点：**

1、集群节点必须要同在一个**物理网络**中

2、RIP地址可以是公网地址

3、Director Server 只负责入站的请求，响应报文直接由Real Server发送给客户端

4、Real Server 的网关一定不能使用DIP

5、无法支持端口映射

6、Real Server 可以支持大多数的操作系统

7、支持大规模的网络场景

## VS/TUN 隧道模型

[![Tun_20151129125121](/assets/wp-content/uploads/2015/11/Tun_20151129125121.png)](/assets/wp-content/uploads/2015/11/Tun_20151129125121.png)

**VS/TUN 流程：**

1、首先，用户访问请求的ip报文 的源ip为用户的 CIP ，目标ip是 VIP

2、数据包到达深圳 DS后，源数据包不变，DS 重新封包，加一层，源ip为DIP，目标mac地址为 RIP ，并转发到RS

3、RS收到数据包并拆解处理，RS 将源ip 改为 VIP ，目标ip 改为 CIP ，直接响应发送给客户端

可以看到，数据包入站需要经过 DS ，但是数据包在RS 是直接发送出去的， 看看 VS/TUN 模型的特点

**VS/TUN 特点：**

1、集群节点可以跨越互联网

2、RIP地址必须是公网地址

3、Director Server 只负责入站的请求，响应报文直接由Real Server发送给客户端

4、Real Server 的网关一定不能使用DIP

5、无法支持端口映射

6、只有支持隧道模式的os 才能用于Real Server

先说到这里，如有错误的地方，希望大家指点

转载请注明：[Huangdc](https://www.huangdc.com) » [Linux集群LVS之一LVS类型详解](https://www.huangdc.com/180)