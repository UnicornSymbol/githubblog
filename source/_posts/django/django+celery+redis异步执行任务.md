---
title: django+celery+redis异步执行任务  
date: 2018-01-16 23:24:53  
tags: [python,django]  
categories: django
---
##### celery简介  
&ensp;&ensp;celery是一个异步任务队列/基于分布式消息传递的作业队列。Celery通过消息（message）进行通信，使用代理（broker）在客户端和工作执行者之间进行交互。当开始一个任务时，客户端发送消息到队列并由代理将其发往响应的工作执行者处。  
<!-- more -->
##### 环境
Django (1.9.5)  
celery (3.1.25)  
redis (2.10.6)  
##### 安装
```
pip install celery
pip install redis
```
##### 配置celery
###### 在project/project目录下(和settings.py同级)，添加如下celery.py文件：
```
#!/usr/bin/env python 
# -*- coding: utf-8 -*-

from __future__ import absolute_import

import os
from celery import Celery
from django.conf import settings

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'hxoms.settings')

app = Celery('assets')

app.config_from_object('django.conf:settings')
app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)


@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))
```
###### 在project/project/__init__.py中添加如下内容：
```
from __future__ import absolute_import, unicode_literals

# This will make sure the app is always imported when
# Django starts so that task will use this app.
from .celery import app as celery_app

__all__ = ['celery_app']

```
###### 在项目配置文件中，settings.py中添加：
```
# Celery settings
import djcelery
djcelery.setup_loader()

# 使用redis作为broker(确保redis服务已经运行并连通)
BROKER_URL = 'redis://127.0.0.1:6379/0'
# 使用redis作为结果存储
CELERY_RESULT_BACKEND = 'redis://127.0.0.1:6379/0'
# 这里我注意到的不多，我只知道返回（包括错误）是json格式的，输入貌似也得是
CELERY_ACCEPT_CONTENT = ['application/json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERYBEAT_SCHEDULER = 'djcelery.schedulers.DatabaseScheduler'
CELERY_TIMEZONE = 'Asia/Shanghai

# Application definition
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'account',
    'assets',
    'rest_framework',
    'django_crontab',
    'djcelery',
]
```
##### 执行异步任务
###### 在app目录下添加task.py文件
```
import time
#from celery import Celery,platforms
from celery import task

#app=Celery('tasks')
#platforms.C_FORCE_ROOT = True

@task()
def add(x, y):
    return x + y

@task
def sendmail(mail):
    print "++++++++++++++++++++++++++++++++++++"
    print('sending mail to %s...' % mail['to'])
    time.sleep(2.0)
    print('mail sent.')
    print "------------------------------------"
    return mail['to']
```
###### 启动worker
```
python manage.py celery worker --loglevel=info
```
**注意：** 如果需要通过celery -A app.tasks worker --loglevel=info命令启动，则需要在task.py中添加以下内容：
```
from celery import Celery,platforms
app=Celery('tasks'，broker='redis://localhost:6379/0')
```
否则会报以下错误：
```
AttributeError: 'module' object has no attribute 'celery'
```
###### 测试
```
# python manage.py shell
In [1]: from assets.tasks import *

In [2]: t = add.delay(1,3)

In [3]: t.get()
Out[3]: 4

In [4]: sendmail.delay(dict(to='asd@as.com'))
Out[4]: <AsyncResult: 3c65dc5f-6aec-4656-b694-9396ccd5f189>
```
同时worker终端会输出以下内容：
```
[2018-01-17 18:34:15,405: WARNING/MainProcess] celery@localhost.localdomain ready.
[2018-01-17 18:36:22,423: INFO/MainProcess] Received task: assets.tasks.add[fcc397ad-696f-4b51-90a3-cd2cd276c420]
[2018-01-17 18:36:22,516: INFO/MainProcess] Task assets.tasks.add[fcc397ad-696f-4b51-90a3-cd2cd276c420] succeeded in 0.0525121779647s: 4
[2018-01-17 18:37:13,551: INFO/MainProcess] Received task: assets.tasks.sendmail[3c65dc5f-6aec-4656-b694-9396ccd5f189]
[2018-01-17 18:37:13,554: WARNING/Worker-1] ++++++++++++++++++++++++++++++++++++
[2018-01-17 18:37:13,556: WARNING/Worker-1] sending mail to asd@as.com...
[2018-01-17 18:37:15,560: WARNING/Worker-1] mail sent.
[2018-01-17 18:37:15,560: WARNING/Worker-1] ------------------------------------
[2018-01-17 18:37:15,562: INFO/MainProcess] Task assets.tasks.sendmail[3c65dc5f-6aec-4656-b694-9396ccd5f189] succeeded in 2.00837284001s: u'asd@as.com'
```
##### 执行计划任务
###### Django后台定义任务
进入Django后台添加contrabs/intervals —> 添加periodic tasks ——> 为periodic task指定contrab/interval以及计划执行的任务(在tasks.py中定义)。  
###### 执行计划任务
```
启动worker
# python manage.py celery worker --loglevel=info
启动beat
# python manage.py celery beat
```
worker终端输出(每分钟任务)：
```
[2018-01-16 17:04:14,668: WARNING/MainProcess] celery@localhost.localdomain ready.
[2018-01-16 17:20:00,058: INFO/MainProcess] Received task: assets.tasks.add[81c39106-7753-4242-bec4-d778a78a9e03]
[2018-01-16 17:20:00,067: INFO/MainProcess] Task assets.tasks.add[81c39106-7753-4242-bec4-d778a78a9e03] succeeded in 0.00645293178968s: 3
[2018-01-16 17:21:00,007: INFO/MainProcess] Received task: assets.tasks.add[223fa2fe-b9da-4058-a1f6-c94f6ac177c8]
[2018-01-16 17:21:00,014: INFO/MainProcess] Task assets.tasks.add[223fa2fe-b9da-4058-a1f6-c94f6ac177c8] succeeded in 0.00481811491773s: 3
[2018-01-16 17:22:00,008: INFO/MainProcess] Received task: assets.tasks.add[a0ba36b5-7456-4bd8-9f95-13bc8a4a7784]
[2018-01-16 17:22:00,014: INFO/MainProcess] Task assets.tasks.add[a0ba36b5-7456-4bd8-9f95-13bc8a4a7784] succeeded in 0.00473075709306s: 3
[2018-01-16 17:23:00,007: INFO/MainProcess] Received task: assets.tasks.add[42d1f7ce-33c6-4d83-8d11-0c0b85a55996]
[2018-01-16 17:23:00,019: INFO/MainProcess] Task assets.tasks.add[42d1f7ce-33c6-4d83-8d11-0c0b85a55996] succeeded in 0.00799032091163s: 3
```
##### 参考资料
[Celery - Distributed Task Queue](http://docs.celeryproject.org/en/latest/index.html)  
[Django Celery Redis 异步执行任务demo实例](https://www.cnblogs.com/guanfuchang/p/6561034.html)  
[Celery+django+redis异步执行任务](http://blog.csdn.net/apple9005/article/details/54236212)  
[django+celery+redis实现运行定时任务](https://www.cnblogs.com/raxxar1024/p/6744501.html)  
[Django中如何使用django-celery完成异步任务 (1)](http://www.weiguda.com/blog/73/)  
[django+celery+djcelery 最简配置](http://blog.csdn.net/tmpbook/article/details/51648596)
