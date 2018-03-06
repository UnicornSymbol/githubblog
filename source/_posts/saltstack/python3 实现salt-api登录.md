---
title: python3 实现salt-api登录 
date: 2017-06-01 17:01:23  
tags: [python,saltstack]  
categories: saltstack
---
```
import urllib.request, urllib.parse
import ssl
headers = {'X-Auth-Token':''}
params = urllib.parse.urlencode({'username':'saltapi', 'password':'123456', 'eauth':'pam'})
obj = urllib.parse.unquote(params)
req = urllib.request.Request('https://192.168.229.128:8888/login', obj.encode(), headers)
context = ssl._create_unverified_context()
res = urllib.request.urlopen(rq,context=context)
res.read()

b'{"return": [{"perms": [".*"], "start": 1495546657.7008209, "token": "5137075130b267800afd7c2dbfd5868b7e89b786", "expire": 1495589857.7008221, "user": "saltapi", "eauth": "pam"}]}'
```
<!-- more -->   
**注:** 由于salt-api使用的是自签名证书，在请求时需要传入未经验证的的上下文参数，否则会报certificate verify failed错误。
