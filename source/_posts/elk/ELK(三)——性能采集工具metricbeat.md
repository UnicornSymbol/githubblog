---
title: ELK(三)——性能采集工具metricbeat  
date: 2017-08-31 18:02:44  
tags: [elk]  
categories: elk
---
##### Beats简介
&ensp;&ensp;Beats是一个代理，负责采集数据发送给elasticsearch或者logstash，Beats分为四个模块可以对系统进行监控，分别为FileBeat、MetricBeat、PacketBeat、WinlogBeat，分为监控文件、系统性能、网络流量、windows系统日志。  
<!-- more -->
##### metricbeat介绍
&ensp;&ensp;metricbeat用来统计并展示系统的信息 cpu， 内存等，也可以用来采集某些服务的信息，例如MySQL、nginx、redis等。  

##### metricbeat安装配置
###### 安装
```
# cd /root/elk
# curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-5.5.2-x86_64.rpm
# rpm -vi metricbeat-5.5.2-x86_64.rpm
```
###### 配置
```
# cat /etc/metricbeat/metricbeat.yml | grep -Ev "^#|^ +#|^$"
metricbeat.modules:  #要监控的模块，有nginx、MySQL、docker等
- module: system
  metricsets:  #监控指标，要保证是模块能够提供的
    - cpu
    - load
    - filesystem
    - fsstat
    - memory
    - network
    - process
  enabled: true  #是否使能对该模块的监控，默认true
  period: 10s  #每次执行取数据的时间间隔，如果系统无效，则每个周期会返回一个错误信息
  processes: ['.*']  #定义一个正则表达式过滤程序
output.elasticsearch:
  hosts: ["172.26.1.65:9200"]  #elasticsearch地址
  username: "elastic"  #x-pack用户名
  password: "changeme"  #x-pack密码
  template.name: "metricbeat"  #模板名称
  template.path: "metricbeat.template.json"  #模板路径
  template.overwrite: false  #是否覆盖已经存在的template

```
###### 手动加载es模板
```
# curl -u elastic:changeme -H 'Content-Type: application/json' -XPUT 'http://172.26.1.65:9200/_template/metricbeat' -d@/etc/metricbeat/metricbeat.template.json
```
###### 启动metricbeat
```
# /etc/init.d/metricbeat start
```
###### 加载kibana图形
```
# /usr/share/metricbeat/scripts/import_dashboards -es http://172.26.1.65:9200 -user elastic -pass changeme  #指定es和x-pack用户名密码
```
###### 展示
![image](/img/elk/metricbeat1.png)  
![image](/img/elk/metricbeat2.png)  
##### 参考资料
[Metricbeat Reference](https://www.elastic.co/guide/en/beats/metricbeat/5.5/index.html)  
[metricbeat实现容器监控](http://blog.csdn.net/tech_salon/article/details/66969521?utm_source=itdadao&utm_medium=referral)