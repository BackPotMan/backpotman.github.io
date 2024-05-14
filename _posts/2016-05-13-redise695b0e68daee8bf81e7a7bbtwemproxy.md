---
id: 361
title: redis数据迁移(twemproxy)
date: '2016-05-13T21:04:35+08:00'
author: huangdc
layout: post
guid: 'http://www.huangdc.com/?p=361'
permalink: /361
duoshuo_thread_id:
    - '6284158598654198529'
views:
    - '6952'
bigfa_ding:
    - '5'
categories:
    - Linux
tags:
    - redis
    - twemproxy
    - 数据迁移
---

redis 数据迁移至 twemproxy redis 集群

在之前的文章讲到实现redis集群高可用，可以查看链接：[twemproxy + redis + sentinel 实现redis集群高可用](http://www.huangdc.com/254)

但是要从旧的redis中将数据迁移到redis集群确实还是要想想办法，因为集群的前端是 twemproxy .  
反正想来想去，实在没有办法了，了解到了redis的这三个命令 dump 、 pttl 、restore  
那就用最原始的方法，将redis的每个key的数据从旧redis直接dump出来，然后通过restore命令导入到 twemproxy redis 集群中。

先来了解一下redis的这三个命令 dump 、 pttl 、restore

## Redis DUMP 命令

Redis DUMP 命令用于序列化给定 key ，并返回被序列化的值

命令基本语法：  
redis 127.0.0.1:4500 &gt; DUMP KEY\_NAME  
如果 key 不存在，那么返回 nil 。 否则，返回序列化之后的值

```
[root@localhost ~]# redis-cli -h 192.168.1.101 -p 4503
192.168.1.101:4503> SET huangdc "hello,welcome huangdc.com!"
OK
192.168.1.101:4503> GET huangdc
"hello,welcome huangdc.com!"
192.168.1.101:4503> 
192.168.1.101:4503> DUMP huangdc
"\x00\x1ahello,welcome huangdc.com!\x06\x00\xfe\xe1:\xea_)7\xe0"
192.168.1.101:4503> 
```

## Redis PTTL 命令

Redis PTTL 命令以毫秒为单位返回 key 的剩余过期时间。  
命令基本语法如下：  
redis 127.0.0.1:6379&gt; PTTL KEY\_NAME  
当 key 不存在时，返回 -2 。 当 key 存在但没有设置剩余生存时间时，返回 -1 。 否则，以毫秒为单位，返回 key 的剩余生存时间。  
注意：在 Redis 2.8 以前，当 key 不存在，或者 key 没有设置剩余生存时间时，命令都返回 -1 。

```
192.168.1.101:4503> GET huangdc
"hello,welcome huangdc.com!"
192.168.1.101:4503> PTTL huangdc
(integer) -1
192.168.1.101:4503> 
192.168.1.101:4503> PEXPIRE huangdc 204800
(integer) 1
192.168.1.101:4503> 
192.168.1.101:4503> PTTL huangdc
(integer) 201868
192.168.1.101:4503>
```

## Redis RESTORE 命令

反序列化给定的序列化值，并将它和给定的 key 关联

命令基本语法：  
redis 127.0.0.1:6379&gt; RESTORE KEY\_NAME ttl serialized-value \[REPLACE\]

参数 ttl 以毫秒为单位为 key 设置生存时间；如果 ttl 为 0 ，那么不设置生存时间。（ttl 的值需要通过 PTTL 命令来回去）  
RESTORE 在执行反序列化之前会先对序列化值的 RDB 版本和数据校验和进行检查，如果 RDB 版本不相同或者数据不完整的话，那么 RESTORE 会拒绝进行反序列化，并返回一个错误。  
如果键 key 已经存在， 并且给定了 REPLACE 选项， 那么使用反序列化得出的值来代替键 key 原有的值；  
相反地， 如果键 key 已经存在， 但是没有给定 REPLACE 选项， 那么命令返回一个错误。

```
192.168.1.101:4503>
192.168.1.101:4503> GET huangdc
"hello,welcome huangdc.com!"
192.168.1.101:4503> DUMP huangdc
"\x00\x1ahello,welcome huangdc.com!\x06\x00\xfe\xe1:\xea_)7\xe0"
192.168.1.101:4503> PEXPIRE huangdc 2048000
(integer) 1
192.168.1.101:4503> PTTL huangdc
(integer) 1971895
192.168.1.101:4503> GET huangdc2
(nil)
192.168.1.101:4503> RESTORE huangdc2 1971895 "\x00\x1ahello,welcome huangdc.com!\x06\x00\xfe\xe1:\xea_)7\xe0"
OK
192.168.1.101:4503> GET huangdc2
"hello,welcome huangdc.com!"
192.168.1.101:4503> 
```

好了，大家知道了，就是将 dump 出来的序列化的值重新RESTORE 到新的key去即可 ，即完成了数据的导入导出

## python多线程脚本实现迁移

最后，贡献一个多线程小脚本来实现：（python脚本，大家提前需要安装redis模块，pip install redis 即可安装）

```
# -*- coding: utf-8 -*-

"""
info: migrate redis cache （redist to twemproxy）
date: 2016-05-12
author: DcHuang
"""

import redis
import os
import sys
import time
from multiprocessing import Pool
from multiprocessing.dummy import Pool as ThreadPool


def usage():
    print "Usage: %s src['1.1.1.1:4500'] dst['2.2.2.2:14500']" % sys.argv[0]


try:
    src=str(sys.argv[1])
    dst=str(sys.argv[2])

    src_ip = src.split(':')[0]
    src_port = src.split(':')[1]

    dst_ip = dst.split(':')[0]
    dst_port = dst.split(':')[1]

except Exception,e:
    usage()
    sys.exit()

now_time = int(time.time())

"""connect redis"""
try:
    src_pool = redis.ConnectionPool(host=str(src_ip), port=int(src_port))  
    src_redis = redis.Redis(connection_pool=src_pool)
except:
    print  "src redis %s connection fail" % src
    sys.exit()

try:
    dst_pool = redis.ConnectionPool(host=str(dst_ip), port=int(dst_port))  
    dst_redis = redis.Redis(connection_pool=dst_pool)
except:
    print  "dst redis %s connection fail" % dst
    sys.exit()


"""导入导出"""
def dump_restore(key):

    """获取pttl"""
    s_pttl = src_redis.pttl(key)
    if s_pttl == -2 :
        return "1"
    elif s_pttl == -1 :
        s_pttl = 0
    else:
        s_pttl = 0

    """获取key的dump,并且restore 导入到新的redis中"""
    try:
        s_dump = src_redis.dump(key)
        if s_dump:
            d_status = dst_redis.restore(key,int(s_pttl),s_dump)
            #print "key:",str(key)," type:",str(src_redis.type(key))," pttl:",str(s_pttl) ##," d_status:",str(d_status)
    except:
        return "1"
    return "0"


"""创建线程执行导出导入函数"""
pool = ThreadPool(10)
results = pool.map(dump_restore,src_keys_all)
pool.close() 
pool.join()

new_time = int(time.time())

################
print "use time(s):",str(new_time - now_time)


```

实例(保存为 mig.py 脚本)：

redis 源： 192.168.1.101:4672  
twemproxy redis 集群： 192.168.1.6:4503  
redis源数据大概有150M ，可以看到数据迁移的用时是108秒，稍微有点慢。

```
[root@localhist DcHuang]# /usr/local/python27/bin/python mig.py '192.168.1.101:4672' '192.168.1.6:4503'
use time(s): 108
```

大家可以自己试试，或者有其他更好的数据迁移方法，希望分享一下

转载请注明：[Huangdc](https://www.huangdc.com) » [redis数据迁移(twemproxy)](https://www.huangdc.com/361)