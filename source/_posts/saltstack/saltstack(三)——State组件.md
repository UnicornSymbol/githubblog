---
title: saltstack(三)——State组件  
date: 2017-05-11 11:28:56  
tags: [saltstack,运维自动化]  
categories: saltstack
---
##### state
&ensp;&ensp;State是salstack最核心的功能，通过预先定制好的sls(salt state file)文件对被控主机进行状态管理，支持包括程序包(pkg)、文件(file)、网络配置(network)、系统服务(service)、系统用户(user)等。  
<!-- more -->  
##### state定义
state定义是通过sls文件进行描述的，支持YAML语法，定义规则如下：  
```
$ID:
  $State
    -	$state: states
```
$ID：定义state名称，通常采用与描述对象保持一致的方法，如apache,nginx等  
$State：管理对象的类型  
$state: states：定制对象的状态  
##### state的使用
&ensp;&ensp;state的入口文件和pillar一样，都是top.sls,但state要求sls文件必须存放在saltstack base定义的目录下，默认为/srv/salt。在top.sls中引用二级配置有2种方式，一种是直接引用，另一种是创建二级目录，再引用目录中的init.sls文件。为了规范起见，建议用第二种。下面是一个根据不同操作系统类型部署Apache的例子。  
###### 定义pillar
```
[root@linux-node1 ~]# cat /srv/pillar/top.sls 
base:
  '*':
    - data
    - apache
[root@linux-node1 ~]# cat /srv/pillar/apache/init.sls 
pkgs:
  {% if grains['os_family'] == 'Debian' %}
    apache: apache2
  {% elif grains['os_family'] == 'RedHat' %}
    apache: httpd
  {% elif grains['os'] == 'Arch' %}
    apache: apache
  {% endif %}
[root@linux-node1 ~]# salt '*' pillar.data pkgs
linux-node2:
    ----------
    pkgs:
        ----------
        apache:
            httpd
linux-node1:
    ----------
    pkgs:
        ----------
        apache:
            httpd
```
###### 定义state
```
[root@linux-node1 ~]# cat /srv/salt/top.sls
base:
  '*':
- apache
[root@linux-node1 ~]# cat /srv/salt/apache/init.sls
apache:
  pkg:
    - installed
    - name: {{ pillar['pkgs']['apahce'] }}
  service.running
    - name: {{ pillar['pkgs']['apache'] }}
    - require:
      - pkg: {{ pillar['pkgs']['apache']}}
```
###### 执行state
```
[root@linux-node1 ~]# salt 'linux-node2' state.sls apache test=True
linux-node2:
----------
          ID: apache-install
    Function: pkg.installed
        Name: httpd
      Result: True
     Comment: Package httpd is already installed.
     Started: 20:57:17.331685
    Duration: 1078.642 ms
     Changes:   
----------
          ID: apache-install
    Function: pkg.installed
        Name: httpd-devel
      Result: True
     Comment: Package httpd-devel is already installed.
     Started: 20:57:18.410806
    Duration: 1.841 ms
     Changes:   
----------
          ID: apache-service
    Function: service.running
        Name: httpd
      Result: True
     Comment: Service httpd is already enabled, and is in the desired state
     Started: 20:57:18.413696
    Duration: 69.99 ms
     Changes:   

Summary
------------
Succeeded: 3
Failed:    0
------------
Total states run:     3
```
**注**：执行state时一般使用salt 'linux-node2' state.sls apache命令，当然也可以执行salt 'linux-node2' state.highstate匹配所有的state.sls模块。test=True指测试安装，并不进行实际操作。  

##### state执行顺序
&ensp;&ensp;state执行时无序，无序是指我们写的sls是无序的，salt为了保证每次执行顺序是一样的，就加入了state order，在这之前先看看高级数据(high data)和低级数据(low data)，高级数据指的就是我们编写sls文件的数据，低级数据就是经过render和parser编译过的数据。  
查看highdata：
```
[root@linux-node1 ~]# salt '*' state.show_highstate
```
查看lowdata：
```
[root@linux-node1 ~]# salt '*' state.show_lowstate
…
      order:
          10000.0001
…
```

通过查看lowstate我们发现里面有有一个字段order，因为salt默认会自动设置order，从10000开始。可以通过设置master配置文件参数state_auto_order: False来关闭。  
Order的设定：  
1). include：被include的文件order靠前，先执行  
2). 手动定义order字段，order的数字越小先执行，从1开始，-1是最后执行，如：  
```
apache:
  pkg:
   - installed
   - order: 1
```
##### 参考资料
[saltstack 全面介绍](http://outofmemory.cn/saltstack/salt)