---
title: ELK(一)——Centos7 elk日志分析平台搭建  
date: 2017-08-20 00:38:52  
tags: [elk]  
categories: elk
---
##### 简介
###### 核心组成
ELK由Elasticsearch、Logstash和Kibana三部分组件组成：  
Elasticsearch是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。  
Logstash是一个完全开源的工具，它可以对你的日志进行收集、分析，并将其存储供以后使用。  
kibana 是一个开源和免费的工具，它可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助您汇总、分析和搜索重要数据日志。  
<!-- more -->
###### ELK工作流程
&ensp;&ensp;在需要收集日志的所有服务上部署logstash，作为logstash agent（logstash shipper）用于监控并过滤收集日志，将过滤后的内容发送到Redis，然后logstash indexer将日志收集在一起交给全文搜索服务ElasticSearch，可以用ElasticSearch进行自定义搜索通过Kibana 来结合自定义搜索进行页面展示。  
![image](/img/elk/elk架构.JPG)
##### 架构
Elasticsearch+redis：centos7.3 x86_64 IP:172.26.1.65  
Logstash+Kibana：centos7.3 x86_64 IP:172.26.1.64  
##### Logstash安装
###### 安装JDK
```
# tar xf jdk-8u101-linux-x64.tar.gz -C /usr/local/
# cat /etc/profile.d/java.sh
export JAVA_HOME=/usr/local/jdk1.8.0_101
export PATH=$PATH:$JAVA_HOME/bin
# . /etc/profile.d/java.sh
# java -version
java version "1.8.0_101"
Java(TM) SE Runtime Environment (build 1.8.0_101-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.101-b13, mixed mode)
```
###### 安装logstash
1. 安装
```
# tar xf logstash-5.5.1.tar.gz
# cd logstash-5.5.1
# echo "export PATH=\$PATH: /root/elk/logstash-5.5.1/bin" > /etc/profile.d/logstash.sh
# . /etc/profile.d/logstash.sh
```
2. 常用参数  
```
-e :指定logstash的配置信息，可以用于快速测试;
-f :指定logstash的配置文件；可以用于生产环境;
```
3. 启动logstash  
通过-e参数指定logstash的配置信息，用于快速测试，直接输出到屏幕：  
```
# logstash -e "input {stdin{}} output {stdout{}}"
...  //启动信息省略
hello  //手动输入回车
2017-08-19T08:41:27.265Z wyb-test1 hello
```
以json格式输出到屏幕：  
```
# logstash -e 'input{stdin{}}output{stdout{codec=>rubydebug}}'
...
hello
{
    "@timestamp" => 2017-08-20T09:01:23.138Z,
      "@version" => "1",
          "host" => "wyb-test1",
       "message" => "hello"
}
```
通过配置文件启动：  
```
# cat config/logstash.conf  //配置文件默认是没有的，需要手动创建
input { stdin {} }
output {
   stdout { codec=> rubydebug }
}
# logstash -f config/logstash.conf
...
hello
{
    "@timestamp" => 2017-08-20T09:23:43.307Z,
      "@version" => "1",
          "host" => "wyb-test1",
       "message" => "hello"
}
```
###### logstash输出信息到redis数据库中  
首先确保redis数据库已经安装。  
```
# cat config/logstash_to_redis.conf 
input { stdin { } }
output {
    stdout { codec => rubydebug }
    redis {
        host => '172.26.1.65'
        port => 6379
        data_type => 'list'
        key => 'logstash:redis'
    }
}
$ ./redis-cli monitor     //另开一个终端开启reids动态监控
# logstash -f config/logstash_to_redis.conf
...
world
{
    "@timestamp" => 2017-08-19T09:07:18.831Z,
      "@version" => "1",
          "host" => "wyb-test1",
       "message" => "world"
}
hello
{
    "@timestamp" => 2017-08-19T09:07:44.334Z,
      "@version" => "1",
          "host" => "wyb-test1",
       "message" => "hello"
}
$ redis-cli monitor  //查看redis监控的输出
OK
1503133638.803386 [0 172.26.1.64:44946] "rpush" "logstash:redis" "{\"@timestamp\":\"2017-08-19T09:07:18.831Z\",\"@version\":\"1\",\"host\":\"wyb-test1\",\"message\":\"world\"}"
1503133664.320678 [0 172.26.1.64:44946] "rpush" "logstash:redis" "{\"@timestamp\":\"2017-08-19T09:07:44.334Z\",\"@version\":\"1\",\"host\":\"wyb-test1\",\"message\":\"hello\"}"
```
##### Elasticseatch安装
###### 安装
elasticsearch默认不能以root用户运行，需要先创建一个elasticsearch用户：
```
# tar xf elasticsearch-5.5.1.tar.gz -C /usr/local/
# cd /usr/local/elasticsearch-5.5.1
添加用户
# useradd elasticsearch
# chown -R elasticsearch:elasticsearch /usr/local/elasticsearch-5.5.1
修改系统最大打开文件数和进程数
编辑/etc/security/limits.conf文件，新增以下内容
* soft nofile 65536 
* hard nofile 65536 
* soft nproc 65536 
* hard nproc 65536
修改elasticsearch配置
# vim config/elasticsearch.yml
network.host: 172.26.1.65
# systemctl stop firewalld
# setenforce 0
# su - elasticsearch 
$ cd /usr/local/elasticsearch-5.5.1
$ nohup /home/elasticsearch/elk/elasticsearch-5.5.1/bin/elasticsearch > /home/elasticsearch/log/elasticsearch.log 2>&1 &
测试
$ curl http://172.26.1.65:9200
{
  "name" : "Rh8Ot71",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "aLFj06p0SgWrnK5ulAbZBQ",
  "version" : {
    "number" : "5.5.1",
    "build_hash" : "19c13d0",
    "build_date" : "2017-07-18T20:44:24.823Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.0"
  },
  "tagline" : "You Know, for Search"
}
```
###### elasticsearch和logstash结合
修改logstash配置文件：
```
# vim config/logstash.conf 

input { stdin { } }
output {
    stdout { codec => rubydebug }
    elasticsearch { hosts => '172.26.1.65' }
    redis {
        host => '172.26.1.65'
        port => 6379
        data_type => 'list'
        key => 'logstash:redis'
    }
}
```
###### 启动logstash
```
# logstash -f config/logstash.conf
...
hello  //输入
{
    "@timestamp" => 2017-08-21T08:32:13.362Z,
      "@version" => "1",
          "host" => "wyb-test1",
       "message" => "hello"
}
```
###### 查看elasticsearch是否收到数据
```
# curl http://172.26.1.65:9200/_search?pretty
{
  "took" : 6,
  "timed_out" : false,
  "_shards" : {
    "total" : 6,
    "successful" : 6,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : ".kibana",
        "_type" : "config",
        "_id" : "5.5.1",
        "_score" : 1.0,
        "_source" : {
          "buildNum" : 15405
        }
      },
      {
        "_index" : "logstash-2017.08.21",
        "_type" : "logs",
        "_id" : "AV4D67lm2zeDiqIzWtWm",
        "_score" : 1.0,
        "_source" : {
          "@timestamp" : "2017-08-21T08:32:13.362Z",
          "@version" : "1",
          "host" : "wyb-test1",
          "message" : "hello"
        }
      }
    ]
  }
}
```
##### Kibana安装
###### 安装
```
# ar xf kibana-5.5.1-linux-x86_64.tar.gz -C /usr/local
# cd /usr/local/kibana-5.5.1-linux-x86_64
修改配置
# vim config/kibana.yml
elasticsearch.url: "http://172.26.1.65:9200"
# ./bin/kibana
  log   [08:23:59.702] [info][status][plugin:kibana@5.5.1] Status changed from uninitialized to green - Ready
  log   [08:23:59.811] [info][status][plugin:elasticsearch@5.5.1] Status changed from uninitialized to yellow - Waiting for Elasticsearch
  log   [08:23:59.864] [info][status][plugin:console@5.5.1] Status changed from uninitialized to green - Ready
  log   [08:23:59.907] [info][status][plugin:metrics@5.5.1] Status changed from uninitialized to green - Ready
  log   [08:23:59.944] [info][status][plugin:elasticsearch@5.5.1] Status changed from yellow to green - Kibana index ready
  log   [08:24:00.179] [info][status][plugin:timelion@5.5.1] Status changed from uninitialized to green - Ready
  log   [08:24:00.190] [info][listening] Server running at http://localhost:5601
  log   [08:24:00.192] [info][status][ui settings] Status changed from uninitialized to green - Ready
# netstat -tlunp | grep 5601
tcp        0      0 127.0.0.1:5601          0.0.0.0:*               LISTEN      20395/./bin/../node
```
###### 使用nginx代理kibana并设置身份验证
```
# rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
# yum install nginx -y
# yum install http-tools
# vim /etc/nginx/conf.d/kibana.conf
server {
    listen 80;
    server_name 172.26.1.64;    #当前主机名
    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/htpasswd.users;      #登录验证
    location / {
        proxy_pass http://127.0.0.1:5601;     #转发到kibana
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
# htpasswd -bc /etc/nginx/htpasswd.users admin admin
Adding password for user admin
# cat /etc/nginx/htpasswd.users 
admin:$apr1$CB/m9/Uy$/E5B0oGx6zE03XQCP4mVR/
# systemctl start nginx.service
# systemctl enable nginx.service
```
###### 测试
浏览器访问kibana能正常显示表示安装成功。  

##### 参考资料
[Elastic Stack and Product Documentation](https://www.elastic.co/guide/index.html)  
[ELK日志分析系统](http://467754239.blog.51cto.com/4878013/1700828/)  
[Centos7_ELK5.4.1配置部署](http://www.mamicode.com/info-detail-1857697.html)  