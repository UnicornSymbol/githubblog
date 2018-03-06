---
title: django-crontab执行定时任务  
date: 2017-12-27 22:45:33  
tags: [python,django]  
categories: django
---
##### 自定义django-admin命令
&ensp;&ensp;用django-admin自定义命令我们可以ORM框架对model进行操作，如：定时更新数据库，检测数据库状态.....  
&ensp;&ensp;Django为项目中每一个应用下的management/commands目录中名字没有以下划线开始的Python模块都注册了一个manage.py命令，例如：  
<!-- more -->  
```
polls/
    __init__.py
    models.py
    management/
        __init__.py
        commands/
            __init__.py
            _private.py
            closepoll.py
    tests.py
    views.py
```
&ensp;&ensp;closepoll.py模块只有一个要求：它必须定义一个Command类并扩展自Basecommand或其子类。  
```
from django.core.management.base import BaseCommand, CommandError
from polls.models import Poll
 
class Command(BaseCommand):
    help = 'Closes the specified poll for voting'
    #必须实现的方法 
    def handle(self, *args, **options):
        for poll_id in options['poll_id']:
            try:
                poll = Poll.objects.get(pk=poll_id)
            except Poll.DoesNotExist:
                raise CommandError('Poll "%s" does not exist' % poll_id)
 
            poll.opened = False
            poll.save()
 
            self.stdout.write('Successfully closed poll "%s"' % poll_id)
```
新的自定义命令可以使用python manage.py closepoll 调用。  
##### django-crontab实现Django定时任务
###### 安装
```
pip install django-crontab
```
###### 配置setting.py
```
INSTALLED_APPS = (
        ...
        'django_crontab',
    )
CRONJOBS = [
    ('0 0 * * *', 'django.core.management.call_command', ['closepoll']，{},'>> /var/run.log'),
]
```
###### 格式
参数1：定时 例如0 0 * * * 表示每天的0时0分执行,遵循crontab语法。  
参数2：方法的python模块路径，如果执行django-admin命令，则写django.core.management.call_command  
参数3：方法的位置参数列表(默认值：[])，如果执行django-admin命令，则填写所需执行的命令，例如我们在polls中已经定义过的closepoll  
参数4：方法的关键字参数的dict(默认值：{})  
参数5：执行log存放位置(即重定向到文件，默认：'')  
###### django-crontab任务加载
django-crontab任务加载比较简单，只需要运行 python manage.py crontab add 即可  
查看已经激活的任务使用 python manage.py crontab show  
删除已经有的任务使用 python manage.py crontab remove  
如果你修改了任务记得一定要使用 python manage.py crontab add 这个会更新定时任务  
##### 参考资料
[django-crontab 定时执行任务方法](http://blog.csdn.net/sinat_21302587/article/details/72831002)  
[Django 定时任务实现(django-crontab+command)](https://www.cnblogs.com/perfe/p/6198213.html)  