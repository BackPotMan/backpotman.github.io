---
id: 66
title: 'SaltStack Job 管理 及 saltutil.signal_job 模块的问题'
date: '2015-11-02T17:56:52+08:00'
author: huangdc
layout: post
guid: 'http://www.huangdc.com/?p=66'
permalink: /66
views:
    - '7265'
duoshuo_thread_id:
    - '6214583362649064194'
bigfa_ding:
    - '7'
categories:
    - SaltStack
tags:
    - SaltStack
---

## <span style="padding: 0px; margin: 0px; font-size: 18px;">**什么是 Job**</span>

 在salt 中，每次执行一次salt命令就会产生一个Job ，Salt 实时管理的任务都是作为Job来执行的；在maste执行一次salt 命令，minion 就会产生一个唯一的 job id ，job id 可以在minion 机器 /var/cache/salt/minion/proc/ 查看 ，我们可以通过 job id 获取到job 的执行状态等信息

例如，测试环境如下：

 master ： 192.168.202.72

 minion ： 192.168.201.37

我们在master 执行一个命令： salt ‘192.168.201.37’ cmd.run “sh /root/dc.sh”

```
## minion 的脚本：（为方便测试及查看，dc.sh 脚本做了 sleep 50s）
[root@localhost ~]# cat /root/dc.sh 
#!/bin/sh
i=1
while (($i<50)) ; do
    echo "$i"
    let ++i 
    sleep 1
done
 
## master 执行：
[root@local200-72 ~]# salt '192.168.201.37' cmd.run "sh /root/dc.sh"
 
## minion 查看：
[root@localhost ~]# ll /var/cache/salt/minion/proc/
total 8
-rw-r--r-- 1 root root 113 Aug  8 15:11 20150808231316960508
```

## <span style="font-size: 18px;">**<span style="padding: 0px; margin: 0px;">Job 常用管理命令 </span>**</span>

（1）saltutil 模块中的 job管理⽅法 <https://docs.saltstack.com/en/develop/ref/modules/all/salt.modules.saltutil.html>

 saltutil.running #查看minion当前正在运⾏的jobs

 saltutil.find\_job &lt;jid&gt; #查看指定jid的job(minion正在运⾏的jobs)

 saltutil.signal\_job &lt;jid&gt; &lt;single&gt; #给指定的jid进程发送信号

 saltutil.term\_job &lt;jid&gt; #终⽌指定的jid进程(信号为15)

 saltutil.kill\_job &lt;jid&gt; #终⽌指定的jid进程(信号为9)

 ## salt ‘\*’ saltutil.signal\_job &lt;job id&gt; 15

（2）salt runner 中的job管理⽅法 <https://docs.saltstack.com/en/develop/topics/jobs/index.html>

 salt-run jobs.active #查看所有minion当前正在运⾏的jobs(在所有minions上运⾏saltutil.running)

 salt-run jobs.lookup\_jid &lt;jid&gt; #从master jobs cache中查询指定jid的运⾏结果

 salt-run jobs.list\_jobs #列出当前master jobs cache中的所有job

（3）runner 模块获取job 状态信息\\

```
import salt.runner
opts = salt.config.master_config('/etc/salt/master')
runner = salt.runner.RunnerClient(opts)
runner.cmd('jobs.lookup_jid', [str(job_id)])
```

## <span style="font-size: 18px;">**<span style="padding: 0px; margin: 0px;">简单实例 </span>**</span>

<span style="padding: 0px; margin: 0px;">（1）在master 执行一个命令： salt ‘192.168.201.37’ cmd.run “sh /root/dc.sh” ; 并查看job 的状态</span>

```
## master 窗口1执行：
[root@local200-72 ~]# salt '192.168.201.37' cmd.run "sh /root/dc.sh"
 
## master 窗口2查看：
[root@local200-72 ~]# salt-run jobs.active   
20150808232959218377:
    ----------
    Arguments:
        - sh /root/dc.sh
    Function:
        cmd.run
    Returned:
    Running:
        |_
          ----------
          192.168.201.37:
              15767
    Target:
        192.168.201.37
    Target-type:
        glob
    User:
        root
[root@local200-72 ~]# salt-run jobs.lookup_jid 20150808232959218377
192.168.201.37:
    1
    2
    3
   ...(省略)
 
## minion：
[root@localhost ~]# ll /var/cache/salt/minion/proc/
total 8
-rw-r--r-- 1 root root 113 Aug  8 15:26 20150808232959218377
```

<span style="color: #333333; font-family: 'Microsoft YaHei', Verdana, sans-serif, 宋体; font-size: 12.5px; letter-spacing: 0.5px; line-height: 22.5px; background-color: #ffffff;">（2）在master 执行一个命令： salt ‘192.168.201.37’ cmd.run “sh /root/dc.sh” ; 并在中途通过job id kill 掉这个 job </span>

```
## master 窗口1执行：
[root@local200-72 ~]# salt '192.168.201.37' cmd.run "sh /root/dc.sh"
 
 
## minion：查看 job id
[root@localhost ~]# ll /var/cache/salt/minion/proc/
total 8
-rw-r--r-- 1 root root 113 Aug  8 15:44 20150808234707827791
[root@localhost ~]# ps -ef |grep dc.sh
root     16986 16985  0 15:44 ?        00:00:00 sh /root/dc.sh
[root@localhost ~]# ps -ef |grep 16985
root     16985     1  0 15:44 ?        00:00:00 /usr/bin/python2.6 /usr/bin/salt-minion -d
root     16986 16985  0 15:44 ?        00:00:00 sh /root/dc.sh
 
## master 查看执行 kill
[root@local200-72 ~]# salt '192.168.201.37' saltutil.signal_job 20150808234707827791 15
192.168.201.37:
    Signal 15 sent to job 20150808234707827791 at pid 16985
     
     
## minion 继续查看 进程及job id
[root@localhost ~]# ps -ef |grep dc.sh
root     16986 1 0 15:44 ?        00:00:00 sh /root/dc.sh
[root@localhost ~]# ll /var/cache/salt/minion/proc/
total 0
 
## 细心的朋友会发现，当我们在master 执行命令调用其他脚本的时候， saltutil.signal_job 把 job id kill 掉了，但是之前此 job 调用的脚本，并没有被kill 掉 （为什么呢）
```

**<span style="padding: 0px; margin: 0px; font-size: 14px;"> </span>**

## <span style="font-size: 18px;">**<span style="padding: 0px; margin: 0px;">Job 管理中的 signal\_job 问题（如上问题）</span>**</span>

<span style="padding: 0px; margin: 0px;"> 当我们在master 执行命令调用其他脚本的时候， saltutil.signal\_job 把 job id kill 掉了，但是之前此 job 调用的脚本，并没有被kill 掉 。</span>

<span style="padding: 0px; margin: 0px;"> （1）linux 中，子进程及父进程的问题 ， 我们可以来测试一下 linux 中是如何kill 掉进程及父进程的  
</span>

<span style="padding: 0px; margin: 0px;"> 通过a.sh 脚本调用 dc.sh (dc.sh 如上不变) ， a.sh 如下：</span>

<span style="padding: 0px; margin: 0px;"> 测试1 ， kill 命令</span>

```
[root@localhost ~]# cat a.sh 
#!/bin/sh
 
sh /root/dc.sh
 
echo "xx"
[root@localhost ~]# sh a.sh &
[root@localhost ~]# ps -ef |grep a.sh 
root     17559 14095  0 15:52 pts/2    00:00:00 sh a.sh
[root@localhost ~]# ps -ef |grep 17559
root     17559 14095  0 15:52 pts/2    00:00:00 sh a.sh
root     17560 17559  0 15:52 pts/2    00:00:00 sh /root/dc.sh
[root@localhost ~]# kill 17559
[root@localhost ~]# ps -ef |grep 17559
root     17633 17569  0 15:52 pts/3    00:00:00 grep 17559
[root@localhost ~]# ps -ef |grep dc.sh
root     17560     1  0 15:52 pts/2    00:00:00 sh /root/dc.sh
 
## 同样，当kill 掉 父进程的id 17559 后， 子进程 dc.sh 仍然在运行  (发现父进程是被kill掉了，但是子进程还活着，而且PPID已经换成为1，也就是init(1)进程)
```

<span style="padding: 0px; margin: 0px;">测试2 ， kill -9 命令 （测试结果同上）</span>

**说明：** kill 一般只能杀掉单个进程，同样，我们也可以发送信号给进程组

 好，测试3，**kill — -&lt;gpid&gt;** (kill 掉进程组) **（ 注意 两个横线&lt;空格&gt;一个横线&lt;gpid&gt;）**

 **查看组进程ID ： ps efo pid,pgid,ppid,comm**

![010632_vPag_588586.png](/ueditor/php/upload/image/20151103/1446562089275163.png "1446562089275163.png")

**<span style="padding: 0px; margin: 0px;">好了，终于把 父进程和 子进程都kill 掉了，，，问题又来了，，，为什么 salt <span style="padding: 0px; margin: 0px;">saltutil.signal\_job 并没有把 子进程也kill 掉呢？</span></span>**

**<span style="padding: 0px; margin: 0px;"><span style="padding: 0px; margin: 0px;"> （2）<span style="padding: 0px; margin: 0px;">修改 minion 的 </span><span style="padding: 0px; margin: 0px;">/usr/lib/python2.6/site-packages/salt/modules/saltutil.py 模块代码</span></span></span>**

我们来看一下 salt saltutil 模块的代码：/usr/lib/python2.6/site-packages/salt/modules/saltutil.py

```
[root@local200-72 modules]# vim /usr/lib/python2.6/site-packages/salt/modules/saltutil.py
 
... (前面省略)
## 我们直接来看一下  signal_job 函数的代码，下面我加了注释
 
def signal_job(jid, sig):
    '''
    Sends a signal to the named salt job's process
 
    CLI Example:
 
    .. code-block:: bash
 
        salt '*' saltutil.signal_job <job id> 15
    '''
    for data in running():
        if data['jid'] == jid:
            try:
                os.kill(int(data['pid']), sig)
                ## 这个地方，你会发现 saltutil 只是使用kill 命令，所以无法kill 掉子进程 ； 
                ## 但你往下看代码，你会发现 saltutil 是有去kill child_pids ，我测试后发现，data['child_pids'] 的值是为空的； 有就是说 salt并没有获取到 child_pids 。 所以 无法kill 掉子进程。
                ## 好了，原因我们知道了，我们应该如何修改呢，当然可以修改salt的saltutil 模块代码
                ## 解决方法 使用 os.killpg(gpid,sig) 
                 
                ## 将上面的 os.kill(int(data['pid']), sig) 修改为 os.killpg(os.getpgid(int(data['pid'])),sig)  即可
                 
                 
                if 'child_pids' in data:
                    for pid in data['child_pids']:
                        os.kill(int(pid), sig)
                return 'Signal {0} sent to job {1} at pid {2}'.format(
                        int(sig),
                        jid,
                        data['pid']
                        )
            except OSError:
                path = os.path.join(__opts__['cachedir'], 'proc', str(jid))
                if os.path.isfile(path):
                    os.remove(path)
                return ('Job {0} was not running and job data has been '
                        ' cleaned up').format(jid)
    return ''
     
... (后面省略)
```

好了，问题终于解决了，，，

总结，这里简单描述一下 salt job 的管理及遇到saltutil.signal\_job 的问题

转载请注明：[Huangdc](https://www.huangdc.com) » [SaltStack Job 管理 及 saltutil.signal\_job 模块的问题](https://www.huangdc.com/66)