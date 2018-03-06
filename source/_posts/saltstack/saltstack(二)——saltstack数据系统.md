---
title: saltstack(二)——saltstack数据系统  
date: 2017-05-09 21:52:17  
tags: [saltstack,运维自动化]  
categories: saltstack
---
##### grains组件
&ensp;&ensp;grains组件是saltstack最重要的组件之一，grains的作用是收集被控端主机的基本信息，这些信息通常都是一些静态类的数据，包括CPU、内核、操作系统、虚拟化等，在服务器端可以根据这些信息进行灵活定制，管理员利用这些信息对不同业务进行个性化配置。  
<!-- more -->  
###### grains常用操作命令
获取所有主机的grains项信息：
```
[root@linux-node1 ~]# salt "*" grains.ls
```
查看grains所有信息：
```
[root@linux-node1 ~]# salt "*" grains.items
```
查看某个grains项信息：
```
[root@linux-node1 ~]# salt "*" grains.item os
```
###### 自定义grains数据
&ensp;&ensp;自定义grains数据有2中方法，一种为在被控主机定制配置文件，另一种通过主控端扩展模块API实现，区别是模块更灵活，可以通过Python编程动态定义，而配置文件只适合相对固定的键与值。  
1. 被控端主机定制grains数据  
配置文件定制的路径为/etc/salt/minion，参数为default_include: minion.d/*.conf。
```
[root@linux-node2 ~]# cat /etc/salt/minion.d/hostinfo.conf
grains:
  roles:
    - webserver
    - memcache
  deployment: datacenter4
  cabinet: 13
[root@linux-node2 ~]# service salt-minion restart    #重启生效
[root@linux-node1 ~]# salt "linux-node2" grains.item roles deployment cabinet
linux-node2:
    ----------
    cabinet:
        13
    deployment:
        datacenter4
    roles:
        - webserver
        - memcache

```
2. 主控端扩展模块定制grains数据  
首先在主控端编写Python代码，然后将该Python文件同步到被控主机，最后刷新生效。Python代码存放的位置为base目录(/etc/salt/master配置文件的file_roots项)。
```
[root@linux-node1 ~]# install -d /srv/salt/_grains
[root@linux-node1 ~]# cat /srv/salt/_grains/sysprocess.py
import os,sys,commands
def Grains_openfile():
	grains = {}
	# init default value
	_open_file=65536

	try:
		getulimit=commands.getstatusoutput('source /etc/profile;ulimit -n')
	except Exception,e:
		pass
	if getulimit[0]==0:
		_open_file=int(getulimit[1])
	grains['max_open_file'] = _open_file
	return grains

[root@linux-node1 ~]# salt 'linux-node2' saltutil.sync_all    #同步模块
linux-node2:
    ----------
    beacons:
    grains:
        - grains.sysprocess
    modules:
    output:
    renderers:
    returners:
    sdb:
    states:
utils:
# 同步后将在被控端生成以下2个文件
[root@linux-node2 ~]# ls /var/cache/salt/minion/extmods/grains/sysprocess.py
/var/cache/salt/minion/extmods/grains/sysprocess.py  #扩展模块文件最终存放位置
[root@linux-node2 ~]# ls /var/cache/salt/minion/files/base/_grains/sysprocess.py 
/var/cache/salt/minion/files/base/_grains/sysprocess.py  #临时存放位置

[root@linux-node1 ~]# salt 'linux-node2' sys.reload_modules    #刷新模块
linux-node2:
True
# 刷新后将在被控端生成一个编译后的字节码文件
[root@linux-node2 ~]# ls /var/cache/salt/minion/extmods/grains/sysprocess.pyc
/var/cache/salt/minion/extmods/grains/sysprocess.pyc

[root@linux-node1 ~]# salt 'linux-node2' grains.item max_open_file    #测试
linux-node2:
    ----------
    max_open_file:
        65535
```
##### pillar组件
&ensp;&ensp;pillar组件也是saltstack最重要的组件之一，其作用是定义与被控主机相关的任何数据，定义好的数据可以被其他组件使用，如模板、state、API等。在pillar中定义的数据与不同业务特性的被控主机相关联，这样不同被控主机只能看到自己匹配的数据，因此pillar安全性很高，适用于一些较敏感的数据。  
###### pillar的定义
1. Saltstack默认将主控端配置文件中的所有数据都定义到pillar中，而且对所有被控主机开放，可以通过修改/etc/salt/amster配置中的pillar_opts: True或False来定义是否开启此功能，修改后执行salt ‘*’ pillar.data来观察效果。  
```
[root@linux-node1 ~]# salt "*" pillar.data
linux-node2:
    ----------
    master:
        ----------
        __role:
            master
        auth_mode:
            1
        auto_accept:
            True
        cache_sreqs:
            True
        cachedir:
            /var/cache/salt/master
        cli_summary:
            False
        client_acl:
            ----------
        client_acl_blacklist:
            ----------
        cluster_masters:
        cluster_mode:
            paranoid
        con_cache:
            False
        ……
```
2. sls文件定义  
Pillar支持在sls文件中定义数据，格式必须符合YAML规范，与saltstack的state组件十分相似，容易混淆。
```
[root@linux-node1 ~]# cat /etc/salt/master    #修改主配置文件
…
pillar_roots:
  base:
    -	/srv/pillar
…
[root@linux-node1 ~]# install -d /srv/pillar
[root@linux-node1 ~]# cat /srv/pillar/top.sls
base:
  '*':
    - data
[root@linux-node1 ~]# cat /srv/pillar/data.sls
appname: website
flow:
  maxconn: 30000
  maxmem: 6G
[root@linux-node1 ~]# salt 'linux-node2' pillar.data appname flow    #测试
linux-node2:
    ----------
    appname:
        website
    flow:
        ----------
        maxconn:
            30000
        maxmem:
            6G
```
**注**：如果结果不符合预期，可以执行salt '*' saltutil.refresh_pillar刷新被控主机pillar数据。
##### grains和pillar区别
grains：静态数据，minion启动时收集，也可以用saltutil.sync_grains进行刷新。  
Pillar：动态数据，在master端定义指定给对应的minion，可以使用saltutil.refresh_pillar刷新。

##### 参考资料
[saltstack 全面介绍](http://outofmemory.cn/saltstack/salt)