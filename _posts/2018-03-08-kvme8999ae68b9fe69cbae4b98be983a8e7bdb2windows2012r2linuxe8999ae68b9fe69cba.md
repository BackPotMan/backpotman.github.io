---
id: 555
title: KVM虚拟机之二部署windows2012R2/linux虚拟机
date: '2018-03-08T10:32:35+08:00'
author: huangdc
layout: post
guid: 'http://www.huangdc.com/?p=555'
permalink: /555
views:
    - '6498'
bigfa_ding:
    - '10'
image: /wp-content/uploads/2016/09/u27613692694049952580fm21gp0-220x150.jpg
categories:
    - Linux
---

之前写过一篇《KVM虚拟机之环境安装部署》,大家可以先看看环境配置链接 http://www.huangdc.com/551  
我已经在内网部署以及使用KVM将近一年了，主要作为内网测试以及服务器使用，从使用情况来看，非常稳定而且操作也很方便  
下面给大家简单介绍下创建linux虚拟机以及windows虚拟机步骤

## 一、创建linux虚拟机

**1、准备安装镜像** /data/kvm\_data/iso/CentOS-6.9-x86\_64-minimal.iso  
可以到官网下载最新版本哈 http://www.centos.org

**2、创建虚拟磁盘** (用于挂载根目录)

```
qemu-img create -f qcow2 -o preallocation=metadata /data/kvm_data/system/test_centos02/test_centos02.qcow2 10G
```

**须知：qcow2格式磁盘文件的预分配(preallocation)策略**

<div><div>使用qemu-img创建qcow2格式磁盘文件的预分配(preallocation)策略，及对虚拟磁盘性能的影响</div><div><div>qcow2是QEMU的虚拟磁盘映像格式，它是一种非常灵活的磁盘格式，支持瘦磁盘（类似稀疏文件）格式，可选的AES加密，zlib压缩及多快照功能。在使用qemu-img创建qcow2虚拟磁盘，可以设置磁盘预分配策略，其支持4种格式：</div><div>- off模式：缺省预分配策略，即不使用预分配策略
- metadata模式：分配qcow2的元数据(metadata)，预分配后的虚拟磁盘仍然属于稀疏映像类型(allocates qcow2 metadata, and it’s still a sparse image.)
- full模式：分配所有磁盘空间并置零，预分配后的虚拟磁盘属于非稀疏映像类型(allocates zeroes and makes a non-sparse image)
- falloc模式：使用posix\_fallocate()函数分配文件的块并标示它们的状态为未初始化，相对full模式来说，创建虚拟磁盘的速度要快很多(which uses posix\_fallocate() to “allocate blocks and marking them as uninitialized”, and is relatively faster than writing out zeroes to a file)

</div></div></div>**3、创建虚拟机**(虚拟机名称为test\_centos02)

```
virt-install --name=test_centos02 --ram 1024 --vcpu=2 --disk path=/data/kvm_data/system/test_centos02/test_centos02.qcow2,size=10,format=qcow2 --accelerate --cdrom /data/kvm_data/iso/CentOS-6.9-x86_64-minimal.iso --graphics vnc,listen=0.0.0.0,password=123456,port=5911 --network bridge=br1,model=virtio --force --autostart
```

\#接着就可以通过VNC 连接进去进行图形界面安装了，我使用的软件是VNC viewer ，大家可以自己下载然后进行安装哈，这里就不截图了  
**4、动态添加硬盘**

```
##创建新的虚拟磁盘
qemu-img create -f qcow2 -o preallocation=metadata /data/kvm_data/system/test_centos02/test_centos02_data.qcow2 100G
```

\##挂载新的磁盘，修改配置文件:(切记需关机，然后进行编辑，最后启动虚拟机)

```
## 编辑test_centos02的配置文件
virsh edit test_centos02
## 找到磁盘内容，注意unit参数值不能重复，修改如下：
<disk type='file' device='disk'>
<driver name='qemu' type='qcow2' cache='none'/>
<source file='/data/kvm_data/system/test_centos02/test_centos02.qcow2'/>
<target dev='hda' bus='ide'/>
<address type='drive' controller='0' bus='0' target='0' unit='0'/>
</disk>
<disk type='file' device='disk'>
<driver name='qemu' type='qcow2' cache='none'/>
<source file='/data/kvm_data/system/test_centos02/test_centos02_data.qcow2'/>
<target dev='hdb' bus='ide'/>
<address type='drive' controller='0' bus='0' target='0' unit='1'/>
</disk>
```

\##切记切记，不要使用attach-disk动态添加新硬盘挂载，会出现挂载格式化后qcow2格式镜像变为raw镜像，没有查到什么原因，尝试了好几次不一样的操作，都是这样。

\## 然后就登录test\_centos02 ，fdisk -l 查看，mkfs.ext4 格式化，mount 挂载即可

## 二、创建windows虚拟机

**1、准备安装镜像** cn\_windows\_server\_2012\_r2\_x64\_dvd\_2707961.iso  
下载地址 https://msdn.itellyou.cn  
我这里下载到 /data/kvm\_data/iso/ 目录  
https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso

**2、创建虚拟磁盘**

```
qemu-img create -f qcow2 /data/kvm_data/system/test_windows2012/test_windows2012.qcow2 60G
```

**3、创建虚拟机**

```
virt-install --name=test_windows2012 --ram 2048 --vcpu=2 --arch=x86_64 --os-type=windows --os-variant='win2k' --disk path=/data/kvm_data/system/test_windows2012/test_windows2012.qcow2,size=60,format=qcow2 --accelerate --cdrom /data/kvm_data/iso/cn_windows_server_2012_r2_x64_dvd_2707961.iso --graphics vnc,listen=0.0.0.0,password=123456,port=5912 --network bridge=br1,model=virtio --force --autostart
```

\#接着就可以通过VNC 连接进去进行图形界面安装了，我使用的软件是VNC viewer ，大家可以自己下载然后进行安装哈，这里就不截图了

**4、密钥 Windows Server 2012 R2**（ServerStandard）  
安装密钥:  
NB4WH-BBBYV-3MPPC-9RCMV-46XCB  
产品密钥:  
MMPXK-NBJDQ-JPM34-WX3FM-G276W

**5、windows 2012 r2 网卡驱动更新**

```
#windows cdrom在线挂载iso
virsh attach-disk test_windows2012 /data/kvm_data/iso/virtio-win.iso hdc --type cdrom --mode readonly
#登录机器，控制面板-设备管理-网络适配器-更新驱动程序软件(NetKVM\xp\x86\)
```

转载请注明：[Huangdc](https://www.huangdc.com) » [KVM虚拟机之二部署windows2012R2/linux虚拟机](https://www.huangdc.com/555)