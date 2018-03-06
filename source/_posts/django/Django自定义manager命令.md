---
title: Django自定义manager命令  
date: 2017-12-27 22:24:54  
tags: [python,django]  
categories: django
---
##### 自定义命令
&ensp;&ensp;在Django之中可以自定义管理员命令，通常是有两种做法的，一种是对单个app定义manage命令，对于该app所做的一些处理。一种是对整个工程所作用的manage命令，用来启动某些service。  
<!-- more -->  
&ensp;&ensp;注册manage命令只需要在app中创建一个management/commands路径，Django会自动将该路径下不以’_’开头的文件注册为manage.py的命令。  
如下路径：
```bash
# tree assets/
assets/
├── admin.py
├── apps.py
├── forms.py
├── __init__.py
├── management
│   ├── commands
│   │   ├── cron.py
│   │   ├── __init__.py
│   ├── __init__.py
├── models.py
├── templates
├── tests.py
├── urls.py
├── views.py
```
&ensp;&ensp;对于注册的manage.py命令closepoll仅仅需要满足将一个条件：其必须定义一个继承自BaseCommand或者其子类的Command类。  
```
from django.core.management.base import BaseCommand, CommandError
from assets.models import Server,Service,Requisition
import datetime
from django.core.exceptions import ObjectDoesNotExist

class Command(BaseCommand):
    def handle(self, *args, **options):
        now = datetime.datetime.now()
        expire = datetime.timedelta(days=7)
        reqs = Requisition.objects.filter(payment_status=2)
        print(now)
        for req in reqs:
            try:
                ser = Server.objects.get(ip=req.asset)
            except ObjectDoesNotExist:
                ser = Service.objects.get(name=req.asset)
            if ser.status == 2:
                continue
            end_date = datetime.datetime.strptime(req.end_date.strftime("%Y/%m/%d"),"%Y/%m/%d")
            if end_date >= (now-datetime.timedelta(days=1)) and now >= (end_date-expire):
                req.payment_status = 3
                req.save()
```
执行cron命令：
```
# python manage.py cron
2017-12-27 22:35:56.561382
```
##### 增加可选参数
&ensp;&ensp;Command提供了一个方法用来增加可选的参数——add_arguments()。  
```
class Command(BaseCommand):
    def add_arguments(self, parser):
        # Positional arguments
        parser.add_argument('id', nargs='+', type=int)

        # Named (optional) arguments
        parser.add_argument('--delete',
            action='store_true',
            dest='delete',
            default=False,
            help='Delete instead of closing it')

    def handle(self, *args, **options):
        # ...
        if options['delete']:
            # ...
```
这种设计就是传入参数如-–delete,所有的manager都有默认的选项–verbosity和–traceback。  
##### 参考资料
[Django 自定义管理员命令](http://blog.csdn.net/sicofield/article/details/49434253)  
[扩展Django:实现自己的manage命令](https://www.cnblogs.com/holbrook/archive/2012/03/09/2387679.html)  