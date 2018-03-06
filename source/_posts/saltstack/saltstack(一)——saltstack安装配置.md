---
title: saltstack(一)——saltstack安装配置  
date: 2017-05-08 12:12:53  
tags: [saltstack,运维自动化]  
categories: saltstack
---
##### Saltstack简介  
&ensp;&ensp;SaltStack是一个服务器基础架构集中化管理平台，具备配置管理、远程执行、监控等功能，一般可以理解为简化版的puppet和加强版的func。SaltStack基于Python语言实现，结合轻量级消息队列（ZeroMQ）与Python第三方模块（Pyzmq、PyCrypto、Pyjinjia2、python-msgpack和PyYAML等）构建。  
<!-- more -->  
&ensp;&ensp;通过部署SaltStack环境，我们可以在成千上万台服务器上做到批量执行命令，根据不同业务特性进行配置集中化管理、分发文件、采集服务器数据、操作系统基础及软件包管理等，SaltStack是运维人员提高工作效率、规范业务配置与操作的利器。

##### Saltstack安装
###### 测试环境
Master：192.168.229.128(linux-node1)
Minion：192.168.229.128(linux-node1)，192.168.229.129(linux-node2)
###### 安装epel源
```
[root@linux-node1 ~]# rpm -ivh http://mirrors.aliyun.com/epel/epel-releae-latest-6.noarch.rpm
```
###### Master安装
```
[root@linux-node1 ~]# yum install salt-master -y
[root@linux-node1 ~]# chkconfig salt-master on
[root@linux-node1 ~]# service salt-master start
```
###### Minion安装
```
[root@linux-node1 ~]# yum install salt-minion –y
[root@linux-node1 ~]# chkconfig salt-minion on
[root@linux-node1 ~]# service salt-minion start
```
###### 配置saltstack
```
[root@linux-node1 ~]# vim /etc/salt/master
# 绑定master IP
Interface: 192.168.229.128
# 自动认证，避免手动运行salt-key来确认证书信任
auto_accept: True
# 指定saltstack文件根目录
file_roots:
  base:
    -	/srv/salt
[root@linux-node1 ~]# mkdir /srv/salt
[root@linux-node1 ~]# service salt-master restart
[root@linux-node1 ~]# vim /etc/salt/minion
# 指定master主机地址
master: 192.168.229.128
# 指定被控主机识别ID，建议使用操作系统主机名来配置，默认通过Python方法socket.getfqdn()去获取fqdn名。
id: linux-node1
[root@linux-node1 ~]# service salt-minion restart
```
##### 参考资料
[Saltstack自动化操作记录（1）-环境部署](http://www.cnblogs.com/kevingrace/p/5570290.html)


