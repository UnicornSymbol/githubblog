---
title: ELK(二)——logstash配置和使用  
date: 2017-08-29 16:37:27  
tags: [elk]  
categories: elk
---
##### LogStash的Shipper和Indexer
&ensp;&ensp;LogStash自身没有什么角色，只是根据不同的功能、不同的配置给出不同的称呼而已。  
Shipper主要是安装在需要收集的日志服务器上，其input为实际的日志源，output一般来说都是Redis(你要是不想用redis做缓存也可以用其他的);  
Indexer则是单独部署(可以集群)，其input是redis(shipper的output)，output则是elasticSearch搜索引擎。  
<!-- more -->  
##### LogStash配置组成
&ensp;&ensp;LogStash配置文件主要由 input、output、filter、codec等区域组成，每个区域内可以定义多个插件。  
###### input
input用来定义日志数据的来源，即日志数据从哪里传输到logstash中。其中常见的配置如下：  
- file：从文件系统中读取一个文件，很像UNIX命令 "tail -0a"  
- syslog：监听514端口，按照RFC3164标准解析日志数据  
- redis：从redis服务器读取数据，支持channel(发布订阅)和list模式。redis一般在Logstash消费集群中作为"broker"角色，保存events队列供Logstash消费。  
- lumberjack：使用lumberjack协议来接收数据，目前已经改为filebeat。  
###### filter
fillter在Logstash处理链中担任中间处理组件。他们经常被组合起来实现一些特定的行为，处理匹配特定规则的事件流。常见的filter配置如下：  
- grok：解析无规则的文字并转化为有结构的格式。Grok 是目前最好的方式来将无结构的数据转换为有结构可查询的数据。有120多种匹配规则，会有一种满足你的需要。  
- mutate：mutate filter允许改变输入的文档，你可以在处理事件的过程中重命名，删除，移动或者修改字段。  
- drop：丢弃一部分events不进行处理，例如：debug events。  
- clone：拷贝event，这个过程中也可以添加或移除字段。  
- geoip：添加地理信息(为前台kibana图形化展示使用)  
###### output
output是logstash处理管道的最末端组件。一个event可以在处理过程中经过多重输出，但是一旦所有的outputs都执行结束，这个event也就完成生命周期。一些常用的output配置包括：  
- elasticsearch：如果你计划将高效的保存数据，并且能够方便和简单的进行查询，Elasticsearch是一个好的方式。  
- file：将event数据保存到文件中。  
- graphite：将event数据发送到图形化组件中，一个很流行的开源存储图形化展示的组件。http://graphite.wikidot.com/。  
- statsd：statsd是一个统计服务，比如技术和时间统计，通过udp通讯，聚合一个或者多个后台服务，如果你已经开始使用statsd，该选项对你应该很有用。  
###### codec
codec是基于数据流的过滤器，它可以作为input，output的一部分配置。Codec可以帮助你轻松的分割发送过来已经被序列化的数据。流行的codecs包括：  
- json：使用json格式对数据进行编码/解码
- multiline：将汇多个事件中数据汇总为一个单一的行。比如：java异常信息和堆栈信息

##### 实例
###### 配置解析
```
input {
    file {  # 使用file 作为输入源
        path => [ "/var/log/nginx/access.log" ]  # 日志的路径，支持/var/log*.log，及[ "/var/log/messages", "/var/log/*.log" ] 格式
        start_position => "beginning"  # 从文件的开始读取事件。另外还有end参数
        ignore_older => 0  #  忽略早于24小时（默认值86400）的日志，设为0，即关闭该功能，以防止文件中的事件由于是早期的被logstash所忽略。
    }
}

filter {
    grok {  #  数据结构化转换工具
        patterns_dir => ["/opt/logstash/patterns"]  # 指定gork表达式文件路径
        match => { "message" => "%{NGINXACCESS}" }  # 匹配条件格式，将nginx日志作为message变量，并应用grok条件NGINXACCESS进行转换
    }
    geoip {  # 该过滤器从geoip中匹配ip字段，显示该ip的地理位置
      source => "clientip"  # ip来源字段
      target => "geoip"  # 指定插入的logstash字断目标存储为geoip
      database => "/opt/logstash/GeoLite2-City_20170801/GeoLite2-City.mmdb"  # geoip数据库的存放路径
      add_field => [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]  # 增加的字段，坐标经度
      add_field => [ "[geoip][coordinates]", "%{[geoip][latitude]}" ]  # 增加的字段，坐标纬度
    }

    mutate {  # 数据的修改、删除、类型转换
      convert => [ "[geoip][coordinates]", "float" ]  # 将坐标转为float类型
      convert => [ "response","integer" ]  # http的响应代码字段转换成 int
      convert => [ "bytes","integer" ]  #  http的传输字节转换成int
      replace => { "type" => "nginx_access" }  # 替换一个字段
      remove_field => "message"  # 移除message 的内容，因为数据已经过滤了一份，这里不必在用到该字段了。不然会相当于存两份
    }

    date {  # 时间处理，该插件很实用，主要是用你日志文件中事件的事件来对timestamp进行转换
      match => [ "timestamp","dd/MMM/yyyy:HH:mm:ss Z"]  # 匹配到timestamp字段后，修改格式为dd/MMM/yyyy:HH:mm:ss Z

    }
    #mutate {
    #  remove_field => "timestamp"  # 移除timestamp字段。
    #}


}
output {
    elasticsearch {  # 输出到es中
        hosts => ["172.26.1.65:9200"]  # es的主机ip＋端口或者es 的FQDN＋端口
        index => "logstash-nginx-access-%{+YYYY.MM.dd}"  # 为日志创建索引logstash-nginx-access-＊，这里也就是kibana那里添加索引时的名称
        user => "elastic"  # x-pack插件用户名
        password => "changeme"  # x-pack插件密码
    }
    stdout {codec => rubydebug}  # json格式输出
}
```
###### pattern配置
gork使用的是ruby正则表达式，可以参考：[Ruby 正则表达式](http://www.runoob.com/ruby/ruby-regular-expressions.html)。  
```
# mkdir -pv /opt/logstash/patterns
]# cat /opt/logstash/patterns/nginx 
NGUSERNAME [a-zA-Z\.\@\-\+_%]+
NGUSER %{NGUSERNAME}
NGINXACCESS %{IPORHOST:clientip} - %{NOTSPACE:remote_user} \[%{HTTPDATE:timestamp}\] \"(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:httpversion})?|%{DATA:rawrequest})\" %{NUMBER:response} (?:%{NUMBER:bytes}|-) %{QS:referrer} %{QS:agent} \"(?: %{IPV4:http_x_forwarded_for}|-)\"
```
###### geoip配置
logstash5依赖于geolite2数据库。  
```
# wget http://geolite.maxmind.com/download/geoip/database/GeoLite2-City.tar.gz
# mv GeoLite2-City.tar.gz /opt/logstash/
# cd /opt/logstash/
# tar xf GeoLite2-City.tar.gz
```
**注：**geoip插件的“source”字段可以是任一处理后的字段，比如“clientip”，但是字段内容却需要小心！GeoIp库内只存有公共网络上的IP信息，查询不到结果的，会直接返回null，而Logstash的GeoIp插件对null结果的处理是：“不生成对应的geoip.字段”。所以读者在测试时，如果使用了诸如127.0.0.1、172.16.0.1、182.168.0.1、10.0.0.1等内网地址，会发现没有对应输出！  
##### 参考资料
[Logstash Reference](https://www.elastic.co/guide/en/logstash/current/index.html)
[利用 ELK系统分析Nginx日志并对数据进行可视化展示](http://www.cnblogs.com/hanyifeng/p/5857875.html)
[llogstash快速入门](http://blog.csdn.net/wp500/article/details/41040213)
[Logstash笔记（三）----Filter插件及grok的正则表达式来解析日志](http://liqingbiao.blog.51cto.com/3044896/1928653)