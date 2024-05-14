---
id: 597
title: KVM虚拟机之四虚拟机vmware转kvm
date: '2018-06-25T15:15:49+08:00'
author: huangdc
layout: post
guid: 'http://www.huangdc.com/?p=597'
permalink: /597
views:
    - '7947'
bigfa_ding:
    - '17'
image: /wp-content/uploads/2016/04/20160427125109-200x150.jpg
categories:
    - Linux
---

当我们需要把vmware虚拟机迁移到kvm时，就伴随着镜像格式的转换，vmdk→img

此方法转载网上方法，我已经成功将vmware虚拟机迁移到kvm，但是忘了原文网址是哪个了，感谢那位朋友的分享

转换过程如下：

**1、检查vmware虚拟机保存目录，查看是否为独立的vmdk文件，如果不是独立文件需要对其进行合并**

还有如果这台虚拟机有快照，需要将快照导出为完整虚拟机！

[![](/assets/wp-content/uploads/2018/06/1529910612854.jpg)](/assets/wp-content/uploads/2018/06/1529910612854.jpg)

合并方法：以管理员身份运行cmd，进入到C:\\Program Files (x86)\\VMware\\VMware Workstation&gt;

用vmware自带的工具vmware- vdiskmanager.exe来合并多个文件，命令如下：

vmware-vdiskmanager.exe -r “C:\\Winxp\\Winxp.vmdk” -t 0 “C:\\Winxpvm.vmdk”

可以看到合并成功，Winxpvm.vmdk就是合并后的独立文件。

[![](/assets/wp-content/uploads/2018/06/1529910706475.jpg)](/assets/wp-content/uploads/2018/06/1529910706475.jpg)

**2、将vmdk文件拷贝到KVM Linux主机，运行**

\[root@localhost ~\]# qemu-img convert Winxp.vmdk -O qcow2 Winxpvm.img

**3、启动virtmanager**，导入镜像创建虚拟机。这时启动的虚拟机可能会发生蓝屏状况（windows虚拟机会有这种情况发生），你需要强制关闭蓝屏虚拟机。

转载请注明：[Huangdc](https://www.huangdc.com) » [KVM虚拟机之四虚拟机vmware转kvm](https://www.huangdc.com/597)