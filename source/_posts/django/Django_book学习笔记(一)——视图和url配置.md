---
title: Django book学习笔记(一)——视图和url配置
date: 2017-03-24 03:09:35
tags: [python,django]
categories: django
---

##### 视图
一个视图就是Python的一个函数，每个视图函数至少要有一个参数，通常被叫作request。 这是一个触发这个视图、包含当前Web请求信息的对象，是类django.http.HttpRequest的一个实例。它返回一个HttpResponse实例。为了使一个Python的函数成为一个Django可识别的视图，它必须满足这两个条件。(也有例外)  
<!-- more -->
例：
``` python
from django.http import HttpResponse
 
def hello(request):
        return HttpResponse("Hello World")
```
##### URL配置
1).URLconf  
URLconf就像是 Django 所支撑网站的目录。它的本质是 URL 模式以及要为该 URL 模式调用的视图函数之间的映射表。你就是以这种方式告诉 Django，对于这个 URL 调用这段代码，对于那个 URL 调用那段代码。
``` python
from django.conf.urls import patterns, include, url
from mysite.views import hello
 
# Uncomment the next two lines to enable the admin:
# from django.contrib import admin
# admin.autodiscover()
 
urlpatterns = patterns('',
    # Examples:
    # url(r'^$', 'mysite.views.home', name='home'),
    # url(r'^mysite/', include('mysite.foo.urls')),
 
    # Uncomment the admin/doc line below to enable admin documentation:
    # url(r'^admin/doc/', include('django.contrib.admindocs.urls')),
 
    # Uncomment the next line to enable the admin:
    # url(r'^admin/', include(admin.site.urls)),
    (r'^hello/$',hello),
)
```
2).URL配置和松耦合
在Django的应用程序中，URL的定义和视图函数之间是松耦合的，换句话说，决定URL返回哪个视图函数和实现这个视图函数是在两个不同的地方。这使得 开发人员可以修改一块而不会影响另一块。  
3).动态URL
我们使用圆括号把参数在URL模式里标识出来。在这个例子中，我们想要把这些数字作为参数，用圆括号把 \d{1,2} 包围起来：
``` python
(r'^time/plus/(\d{1,2})/$', hours_ahead),
```
对应视图函数：
``` python
from django.http import Http404, HttpResponse
import datetime
 
def hours_ahead(request, offset):
    try:
        offset = int(offset)
    except ValueError:
        raise Http404()
    dt = datetime.datetime.now() + datetime.timedelta(hours=offset)
    html = "<html><body>In %s hour(s), it will be %s.</body></html>" % (offset, dt)
    return HttpResponse(html)

```
解释：offset 是从匹配的URL里提取出来的。 例如：如果请求URL是/time/plus/3/，那么offset将会是3；如果请求URL是/time/plus/21/，那么offset将会是21。请注意：捕获值永远都是字符串（string）类型，而不会是整数（integer）类型，即使这个字符串全由数字构成（如：“21”）。  
4).URL中常用的正则表达式
![正则表达式](/img/django/url正则表达式.png)
##### Django处理请求过程
1).进来的请求转入/hello/.  
2).Django通过在ROOT_URLCONF配置来决定根URLconf.  
3).Django在URLconf中的所有URL模式中，查找第一个匹配/hello/的条目。  
4).如果找到匹配，将调用相应的视图函数  
5).视图函数返回一个HttpResponse  
6.)Django转换HttpResponse为一个适合的HTTP response， 以Web page显示出来  

