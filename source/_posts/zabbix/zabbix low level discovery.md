---
title: zabbix low level discovery  
date: 2017-09-20 12:43:12  
tags: [zabbix]  
categories: zabbix
---
##### lld概述
&ensp;&ensp;lld用于发现item、trigger、graph等等。我们最常用如：filesystem（如/、/home、/proc、C:、D:等），network（eth0，eth1等），例如众多服务器，难免系统以及分区会有所不同。一般存在linux和windows两种系统，linux下分区有/、/data、/proc等等，windows有C: D: E:等，A服务器有/data分区，B服务器可能有/site分区。他有什么分区，我便监控什么分区，这就是low-level discovery的功能。  
<!-- more -->  
##### lld配置
###### 创建模板
省略  
###### 创建自动发现规则
&ensp;&ensp;步骤为：Configuration ----> Templates ------> 选中之前创建好的模板名----->Create discovery rule  
![image](/img/zabbix/discovery-rule.png)  
###### 创建item原型
&ensp;&ensp;创建好auto discovery rule后，可以通过Configuration ----> Templates ----> Template Redis Auto Discovery（假设模板名为该名称）----> Discovery rules ----> Item prototypes ----> Create item prototype 创建新的item 。  
![image](/img/zabbix/itemprototype.png)  
###### 客户端配置
1. 端口收集json化：
```
# cat redis_port.py
#!/usr/bin/env python
import os
import json
t=os.popen("""netstat -natp|awk -F: '/redis-server/&&/LISTEN/{print $2}'|awk '{print $1}' """)
ports = []
for port in  t.readlines():
        r = os.path.basename(port.strip())
        ports += [{'{#REDISPORT}':r}]
print json.dumps({'data':ports},sort_keys=True,indent=4,separators=(',',':'))
```
   运行结果：
```
python redis_port.py
{
    "data":[
        {
            "{#REDISPORT}":"6376"
        },
        {
            "{#REDISPORT}":"6378"
        },
        {
            "{#REDISPORT}":"6379"
        }
    ]
}
```
2. zabbix_agentd.conf配置
```
UserParameter=redis.discovery,/etc/zabbix/redis_port.py
UserParameter=redis_stats[*],redis-cli -h 127.0.0.1 -p $1 info|grep $2|cut -d : -f2
```
###### 服务端验证
```
# zabbix_get -s 127.0.0.1 -p 10050 -k redis.discovery
{
    "data":[
        {
            "{#REDISPORT}":"6376"
        },
        {
            "{#REDISPORT}":"6378"
        },
        {
            "{#REDISPORT}":"6379"
        }
    ]
}
```
##### 参考资料
[Low-level discovery](https://www.zabbix.com/documentation/3.2/manual/discovery/low_level_discovery)
[zabbix小结（六）low level discovery](http://825536458.blog.51cto.com/4417836/1827735)