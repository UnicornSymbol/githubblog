---
title: Centos6.5 salt-api安装和配置  
date: 2017-05-31 22:08:17  
tags: [saltstack]  
categories: saltstack
---
##### 安装
&ensp;&ensp;一般情况下，salt-api会使用HTTPS，首次配置成功后，使用用户名和密码登陆，获得Token，Token创建后，默认有效期是12小时，在有效期之内，使用该Token可以代替使用用户名和密码来访问API(该有效时间可在salt-master配置文件中修改)。
```
rpm -ivh http://mirrors.aliyun.com/epel/epel-releae-latest-6.noarch.rpm
yum install salt-api -y
```
<!-- more -->
##### 配置
###### 生成证书
```
cd /etc/pki/tls/certs
make testcert

#Enter pass phrase: 键入加密短语
#Verifying - Enter pass phrase: 确认加密短语
#/usr/bin/openssl req -utf8 -new -key /etc/pki/tls/private/localhost.key -x509 -days 365 -out /etc/pki/tls/certs/localhost.crt -set_serial 0
#Enter pass phrase for /etc/pki/tls/private/localhost.key: 再次输入相同的加密短语
#Country Name (2 letter code) [XX]:CN
#State or Province Name (full name) []:Guangdong
#Locality Name (eg, city) [Default City]:Shenzhen
#Organization Name (eg, company) [Default Company Ltd]:
#Organizational Unit Name (eg, section) []:
#Common Name (eg, your name or your server's hostname) []:
#Email Address []:

cd ../private/
openssl rsa -in localhost.key -out localhost_nopass.key

#Enter pass phrase for localhost.key: 输入之前的加密短语
```
也可以借助salt-call命令来生成:
```
salt-call --local tls.create_self_signed_cert
```

###### 添加用户
```
useradd -M -s /sbin/nologin saltapi
passwd saltapi
```
###### 配置salt-api
```
mkdir -p /etc/salt/master.d/ 
cd /etc/salt/master.d/ 
touch eauth.conf
touch api.conf

#vi eauth.conf
external_auth:
  pam:
    saltapi:   #用户
      - .*     #该配置文件给予saltapi用户所有模块使用权限，出于安全考虑一般只给予特定模块使用权限

#vi api.conf
rest_cherrypy:
  port: 8888
  ssl_crt: /etc/pki/tls/certs/localhost.crt
  ssl_key: /etc/pki/tls/private/localhost_nopass.key
```
###### 重启服务
```
service salt-master restart
service salt-api restart
```
###### 验证
```
#获取token
curl -k https://192.168.181.15:8888/login -H "Accept: application/x-yaml" -d username='saltapi' -d password='123456' -d eauth='pam'

return:
- eauth: pam
  expire: 1495586320.09008
  perms:
  - .*
  start: 1495543120.0900791
  token: 0fd22dfdc3062ae0ca6d708a4803a40a83e5b6b0
  user: saltapi
```

##### 参考资料
[Saltstack学习笔记——安装Salt-API](http://www.jianshu.com/p/6c9b34cfe3f9)  
[SaltStack RESTful API的调用](http://sanwen.net/a/ecbegbo.html)  