---
title: Django book学习笔记(三)——模型  
date: 2017-04-13 15:09:10  
tags: [python,django]  
categories: django
---

##### 模型简介
&ensp;&ensp;模型，即数据存取层。该层处理与数据相关的所有事务：如何存取、如何验证有效性、包含哪些行为以及数据之间的关系等。它能有效的避免我们将数据库操作硬编码于代码中，并且方便在不同数据库之间进行迁移。  
<!-- more -->
##### 数据库设置
打开settings.py配置文件，找到数据库配置DATABASES,例如：
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', # 使用哪个数据库引擎
        'NAME': 'djangodb',                      # 数据库名称
        'USER': 'django',                      # 用哪个用户连接数据库
        'PASSWORD': 'django',                  # 用户密码
        'HOST': 'localhost',                      # 数据库服务器监听地址
        'PORT': '3306',                      # 数据库服务器监听端口
    }
}
```
##### 创建应用程序

```
python manage.py startapp books
```
**注**：系统对app有一个约定：如果你使用了Django的数据库层(模型)，你必须创建一个django app，模型必须存放在apps中。
##### 模型定义
第一步是用Python代码来描述它们。打开由startapp命令创建的models.py并输入下面的内容：
``` python
from django.db import models

class Publisher(models.Model):
    name = models.CharField(max_length=30)
    address = models.CharField(max_length=50)
    city = models.CharField(max_length=60)
    state_province = models.CharField(max_length=30)
    country = models.CharField(max_length=50)
    website = models.URLField()

class Author(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=40)
    email = models.EmailField()

class Book(models.Model):
    title = models.CharField(max_length=100)
    authors = models.ManyToManyField(Author)
    publisher = models.ForeignKey(Publisher)
    publication_date = models.DateField()
```
首先要注意的事是每个数据模型都是django.db.models.Model的子类。它的父类Model包含了所有和数据库打交道的方法，并提供了一个简洁漂亮的定义语法。每个模型相当于单个数据库表，每个属性也是这个表中的一个字段。属性名就是字段名，它的类型例如(CharField)相当于数据库的字段类型 (例如 varchar)。  
##### 模型安装
###### 激活模型
要在数据库中创建这些表，第一步是在Django项目中激活这些模型。将app添加到配置文件的已INSTALLED\_APPS列表中即可完成此步骤。INSTALLED\_APPS告诉 Django 项目哪些 app 处于激活状态。例如：
```
INSTALLED_APPS = (
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Uncomment the next line to enable the admin:
    # 'django.contrib.admin',
    # Uncomment the next line to enable admin documentation:
    # 'django.contrib.admindocs',
    'debug_toolbar',
    'books',
)
```
###### 校验模型
第二步用下面的命令校验模型的有效性：
``` python
python manage.py validate
```
###### 生成sql语句
第三步模型确认没问题了，运行下面的命令来生成 CREATE TABLE 语句：
``` python
python manage.py sqlall books
```
**注**：sqlall 命令并没有在数据库中真正创建数据表，只是把SQL语句段打印出来。  
###### 同步模型
第四步运行以下命令同步模型到数据库：
```
python manage.py syncdb
```
##### 基本数据访问
一旦你创建了模型，Django自动为这些模型提供了高级的Python API。运行python manage.py shell可以和数据库进行交互。
```
>>> from books.models import Publisher
>>> p1 = Publisher(name='Apress', address='2855 Telegraph Avenue',
...     city='Berkeley', state_province='CA', country='U.S.A.',
...     website='http://www.apress.com/')
>>> p1.save()
>>> p2 = Publisher(name="O'Reilly", address='10 Fawcett St.',
...     city='Cambridge', state_province='MA', country='U.S.A.',
...     website='http://www.oreilly.com/')
>>> p2.save()
>>> publisher_list = Publisher.objects.all()
>>> publisher_list
[<Publisher: Publisher object>, <Publisher: Publisher object>]
```
**注意**：当你使用Django modle API创建对象时Django并未将对象保存至数据库内，除非你调用save()方法。  
如果需要一步完成对象的创建与存储至数据库，就使用objects.create()方法。
```
>>> p1 = Publisher.objects.create(name='Apress',
...     address='2855 Telegraph Avenue',
...     city='Berkeley', state_province='CA', country='U.S.A.',
...     website='http://www.apress.com/')
```
##### 添加模块的字符串表现
当我们打印整个publisher列表时，我们没有得到想要的有用的信息：
```
>>> publisher_list = Publisher.objects.all()
>>> publisher_list
[<Publisher: Publisher object>, <Publisher: Publisher object>]
```
只需要添加一个方法 \_\_unicode\_\_() 到 Publisher对象。\_\_unicode\_\_()方法告诉Python如何实现对象的unicode表示。
```
from django.db import models

class Publisher(models.Model):
    name = models.CharField(max_length=30)
    address = models.CharField(max_length=50)
    city = models.CharField(max_length=60)
    state_province = models.CharField(max_length=30)
    country = models.CharField(max_length=50)
    website = models.URLField()

    def __unicode__(self):
        return self.name

class Author(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=40)
    email = models.EmailField()

    def __unicode__(self):
        return u'%s %s' % (self.first_name, self.last_name)

class Book(models.Model):
    title = models.CharField(max_length=100)
    authors = models.ManyToManyField(Author)
    publisher = models.ForeignKey(Publisher)
    publication_date = models.DateField()

    def __unicode__(self):
        return self.title
```
为了让我们的修改生效，先退出Python Shell，然后再次运行 python manage.py shell 进入。
```
>>> from books.models import Publisher
>>> publisher_list = Publisher.objects.all()
>>> publisher_list
[<Publisher: Apress>, <Publisher: O'Reilly>]
```
