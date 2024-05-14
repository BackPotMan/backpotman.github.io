---
id: 370
title: 运维自动化平台iamdc项目二之工单系统
date: '2016-05-22T07:38:47+08:00'
author: huangdc
layout: post
guid: 'http://www.huangdc.com/?p=370'
permalink: /370
views:
    - '18386'
duoshuo_thread_id:
    - '6287080546904310530'
bigfa_ding:
    - '28'
categories:
    - 运维自动化平台
tags:
    - iamdc
    - 工单系统
    - 运维自动化平台
---

前面一篇文章已经说过了：[运维自动化平台iamdc项目一之用户管理](http://www.huangdc.com/368)，这篇文章我们就来说一下 **运维自动化平台iamdc项目二之工单系统** ，工单系统花的时间还是比较多的 , 现在也是马马虎虎的做了个大体

**同样的，代码已同步至github ，大家可以自由下载：**

git@github.com:huangdongcong/iamdc.git

只要你将环境部署好，下载之后应该可以直接运行(记得配置数据库信息以及同步数据库)

代码写的不是很整洁或者有一些问题，后续会改进优化，欢迎大家一起来交流 ，可以加QQ群：553052774

## 工单说明

工单系统，用于本部门对其他部门提供服务的接口，主要针对其他部门与运维的一些工作流（申请项目，申请机器，申请域名，申请权限，排查问题等等）

## 工单处理逻辑

1、首先管理员定义好工单的类型，每种工单类型都有各自的审核流程

2、然后用户可以在新建工单中选择工单类型并提交工单任务（工单任务会按照之前工单类型里面定义的审核流程及处理人流转）

3、工单流程到审核人，审核人可以审核通过或者不通过，也可以转发给其他同事审核，

4、接着工单流转到类型处理人，也就是说这类工单任务的负责人进行相关处理，也可以转发或者打回去

5、最后最后等执行人完成之后，再流转回工单创建人进行确认，确认是否符合工单创建者的要求 ，这时可以关闭工单，或者对结果不满意，可以让执行人重新操作执行

## 工单系统模型设计

1、工单类型表(Ticket\_Type)：包含字段有工单类型名称、工单处理人（外键 — 多对一）、工单审核人、工单状态、创建时间，修改时间

2、工单任务表(Ticket\_Tickets)：包含字段工单ID 、工单标题、工单类型（外键 — 多对一）、工单创建者（外键 — 多对一）、工单需求、工单创建时间、工单结果、工单执行者（外键 — 多对一），工单审核人名单（外键 — 多对多）、工单状态

3、工单审核流程表（Tickets\_Users）：包含字段 ID、工单任务表（外键 — 多对一）、用户（外键 — 多对一）、状态

4、工单操作表（ticket\_operating）：包含字段 工单任务表（外键 — 多对一）、操作人、操作类型、操作时间、内容

5、工单回复表（ticket\_reply）：包含字段 工单任务表（外键 — 多对一）、回复人、回复时间、回复内容

## **工单系统效果图**

[![iamdc_t20160521175431](/assets/wp-content/uploads/2016/05/iamdc_t20160521175431.png)](/assets/wp-content/uploads/2016/05/iamdc_t20160521175431.png)

[![iamdc_t20160521175432](/assets/wp-content/uploads/2016/05/iamdc_t20160521175432.png)](/assets/wp-content/uploads/2016/05/iamdc_t20160521175432.png)

[![iamdc_t20160521175433](/assets/wp-content/uploads/2016/05/iamdc_t20160521175433.png)](/assets/wp-content/uploads/2016/05/iamdc_t20160521175433.png)

[![iamdc_t20160521175434](/assets/wp-content/uploads/2016/05/iamdc_t20160521175434.png)](/assets/wp-content/uploads/2016/05/iamdc_t20160521175434.png)

[![iamdc_t20160521175435](/assets/wp-content/uploads/2016/05/iamdc_t20160521175435.png)](/assets/wp-content/uploads/2016/05/iamdc_t20160521175435.png)

[![iamdc_t20160521175436](/assets/wp-content/uploads/2016/05/iamdc_t20160521175436.png)](/assets/wp-content/uploads/2016/05/iamdc_t20160521175436.png)

[![iamdc_t20160521175437](/assets/wp-content/uploads/2016/05/iamdc_t20160521175437.png)](/assets/wp-content/uploads/2016/05/iamdc_t20160521175437.png)

转载请注明：[Huangdc](https://www.huangdc.com) » [运维自动化平台iamdc项目二之工单系统](https://www.huangdc.com/370)