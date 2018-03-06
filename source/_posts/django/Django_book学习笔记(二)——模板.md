---
title: Django book学习笔记(二)——模板
date: 2017-03-28 13:40:36
tags: [python,django]
categories: django
---

##### 模板系统的基本知识
模板是一个文本，用于分离文档的表现形式和内容。 模板定义了占位符以及各种用于规范文档该如何显示的各部分基本逻辑（模板标签）。 模板通常用于产生HTML，但是Django的模板也能产生任何基于文本格式的文档。  
<!-- more -->
##### 如何使用模板系统
在Python代码中使用Django模板的最基本方式如下：  
1).可以用原始的模板代码字符串创建一个 Template 对象， Django同样支持用指定模板文件路径的方式来创建 Template 对象;  
2).调用模板对象的render方法，并且传入一套变量context。它将返回一个基于模板的展现字符串，模板中的变量和标签会被context值替换。  
例：  

``` python
>>> from django import template
>>> t = template.Template('My name is {{ name }}.')
>>> c = template.Context({'name': 'Adrian'})
>>> print t.render(c)
My name is Adrian.
>>> c = template.Context({'name': 'Fred'})
>>> print t.render(c)
My name is Fred.
```
##### 深度变量的查找
在 Django 模板中遍历复杂数据结构的关键是句点字符 (.)。  
比如，假设你要向模板传递一个 Python 字典。 要通过字典键访问该字典的值，可使用一个句点：  

``` python
>>> from django.template import Template, Context
>>> person = {'name': 'Sally', 'age': '43'}
>>> t = Template('{{ person.name }} is {{ person.age }} years old.')
>>> c = Context({'person': person})
>>> t.render(c)
u'Sally is 43 years old.'
```
同样，也可以通过句点来访问对象的属性。点语法也可以用来引用对象的方法。最后，句点也可用于访问列表索引。
句点查找规则可概括为： 当模板系统在变量名中遇到点时，按照以下顺序尝试进行查找：  
- 字典类型查找 （比如 foo["bar"] )
- 属性查找 (比如 foo.bar )
- 方法调用 （比如 foo.bar() )
- 列表类型索引查找 (比如 foo[bar] )  

系统使用所找到的第一个有效类型。 这是一种短路逻辑。  
**注意**：
- 调用方法时不需要使用圆括号，而且也无法给该方法传递参数；你只能调用不需参数的方法。
- 在方法查找过程中，如果某方法抛出一个异常，如果该异常有一个 silent_variable_failure 属性并且值为 True，则模板里的指定变量会被置为空字符串。
- 如果模板文件里包含了 {{ account.delete }} ，对象又具有 delete()方法，而且delete() 有alters_data=True这个属性，那么在模板载入时， delete()方法将不会被执行。 它将静静地错误退出。 
 
##### 基本的模板标签和过滤器
1).if/else  
{% raw %}
{% if %} 标签检查(evaluate)一个变量，如果这个变量为真(即，变量存在，非空，不是布尔值假)，系统会显示在 {% if %} 和 {% endif %} 之间的任何内容，例如：  
{% endraw %}

``` template
{% if today_is_weekend %}
    <p>Welcome to the weekend!</p>
{% else %}
    <p>Get back to work.</p>
{% endif %}
```
{% raw %}
{% if %} 标签接受 and ， or 或者 not 关键字来对多个变量做判断 ，或者对变量取反（ not )，但是{% if %} 标签不允许在同一个标签中同时使用 and 和 or ，因为逻辑上可能模糊的。而且没有 {% elif %} 标签， 请使用嵌套的{% if %}标签来达成同样的效果。一定要用 {% endif %} 关闭每一个 {% if %} 标签。  
{% endraw %}
2).for  
{% raw %}
{% for %} 允许我们在一个序列上迭代。每一次循环中，模板系统会渲染在 {% for %} 和 {% endfor %} 之间的所有内容。  
{% endraw %}
给标签增加一个reversed使得该列表被反向迭代。

``` template
{% for athlete in athlete_list reversed %}
...
{% endfor %}
```
{% raw %}
for标签支持一个可选的{% empty %}分句，通过它我们可以定义当列表为空时的输出内容。
{% endraw %}

```
{% for athlete in athlete_list %}
    <p>{{ athlete.name }}</p>
{% empty %}
    <p>There are no athletes. Only computer programmers.</p>
{% endfor %}
```
{% raw %}
在每个{% for %}循环里有一个称为forloop的模板变量。这个变量有一些提示循环进度信息的属性。  
{% endraw %}
- forloop.counter 总是一个表示当前循环的执行次数的整数计数器。 这个计数器是从1开始的，所以在第一次循环时 forloop.counter 将会被设置为1。

``` template
{% for item in todo_list %}
    <p>{{ forloop.counter }}: {{ item }}</p>
{% endfor %}
```
- forloop.counter0 类似于 forloop.counter ，但是它是从0计数的。 第一次执行循环时这个变量会被设置为0。
- forloop.revcounter 是表示循环中剩余项的整型变量。 在循环初次执行时 forloop.revcounter 将被设置为序列中项的总数。 最后一次循环执行中，这个变量将被置1。
- forloop.revcounter0 类似于 forloop.revcounter ，但它以0做为结束索引。在第一次执行循环时，该变量会被置为序列的项的个数减1。
- forloop.first 是一个布尔值。 在第一次执行循环时该变量为True，在下面的情形中这个变量是很有用的。

``` template
{% for object in objects %}
    {% if forloop.first %}<li class="first">{% else %}<li>{% endif %}
    {{ object }}
    </li>
{% endfor %}
```
- forloop.last 是一个布尔值；在最后一次执行循环时被置为True。 一个常见的用法是在一系列的链接之间放置管道符(|)

``` template
{% for link in links %}{{ link }}{% if not forloop.last %} | {% endif %}{% endfor %}
```
- forloop.parentloop 是一个指向当前循环的上一级循环的 forloop 对象的引用（在嵌套循环的情况下）。  

3).ifequal/ifnotequal  
{% raw %}
{% ifequal %} 标签比较两个值，当他们相同时，显示在 {% ifequal %} 和 {% endifequal %} 之中所有的值。和 {% if %} 类似， {% ifequal %} 支持可选的 {% else%} 标签。
{% endraw %}

``` template
{% ifequal section 'sitenews' %}
    <h1>Site News</h1>
{% else %}
    <h1>No News Here</h1>
{% endifequal %}
```
{% raw %}
只有模板变量，字符串，整数和小数可以作为 {% ifequal %} 标签的参数。
{% endraw %}
``` template
{% ifequal variable 1 %}
{% ifequal variable 1.23 %}
{% ifequal variable 'foo' %}
{% ifequal variable "foo" %}
```
{% raw %}
其他的一些类型，例如Python的字典类型、列表类型、布尔类型，不能用在 {% ifequal %} 中。下面是些错误的例子：
{% endraw %}
``` template
{% ifequal variable True %}
{% ifequal variable [1, 2, 3] %}
{% ifequal variable {'key': 'value'} %}
```
##### 注释
Django模板语言同样提供代码注释。 注释使用 {# #}，但这种语法的注释不能跨越多行：
``` template
{# This is a comment #}
```
{% raw %}
如果要实现多行注释，可以使用{% comment %}模板标签，就像这样：
{% endraw %}
``` template
{% comment %}
This is a
multi-line comment.
{% endcomment %}
```
##### 过滤器
模板过滤器是在变量被显示前修改它的值的一个简单方法。 过滤器使用管道字符，如下所示：
``` template
{{ name|lower }}
```
有些过滤器有参数。 过滤器的参数跟随冒号之后并且总是以双引号包含。 例如：
``` template
{{ bio|truncatewords:"30" }}
```
以下几个是最为重要的过滤器的一小部分。  
- addslashes : 添加反斜杠到任何反斜杠、单引号或者双引号前面。
- date : 按指定的格式字符串参数格式化 date 或者 datetime 对象。
- length : 返回变量的长度。 对于列表，这个参数将返回列表元素的个数。 对于字符串，这个参数将返回字符串中字符的个数。 你可以对列表或者字符串，或者任何知道怎么测定长度的Python 对象使用这个方法（也就是说，有 __len__() 方法的对象）。

##### 在视图中使用模板
1).模板加载  
打开settings.py配置文件，找到TEMPLATE_DIRS添加一个目录用于存放模板文件，例如：
```
TEMPLATE_DIRS = (
    '/home/django/mysite/templates',
)
```
完成 TEMPLATE_DIRS 设置后，下一步就是修改视图代码，让它使用 Django 模板加载功能而不是对模板路径硬编码。例：
``` python
from django.template.loader import get_template
from django.template import Context
from django.http import HttpResponse
import datetime

def current_datetime(request):
    now = datetime.datetime.now()
    t = get_template('current_datetime.html')
    html = t.render(Context({'current_date': now}))
    return HttpResponse(html)
```
get_template()方法会自动为你连接已经设置的TEMPLATE_DIRS目录和你传入该法的模板名称参数，在文件系统中找出模块的位置，打开文件并返回一个编译好的 Template 对象。  
2).render_to_response()  
Django提供了一个捷径，让你一次性地载入某个模板文件，渲染它，然后将此作为HttpResponse返回。该捷径就是位于django.shortcuts模块中名为render_to_response() 的函数，大多数情况下，你会使用render_to_response()而不是手动加载模板并创建Context和HttpResponse对象。  
render_to_response() 的第一个参数必须是要使用的模板名称。 如果要给定第二个参数，那么该参数必须是为该模板创建 Context 时所使用的字典。 如果不提供第二个参数， render_to_response() 使用一个空字典。
``` python
from django.shortcuts import render_to_response
import datetime

def current_datetime(request):
    now = datetime.datetime.now()
    return render_to_response('current_datetime.html', {'current_date': now})
```
##### include模板标签
{% raw %}
该标签允许在（模板中）包含其它的模板的内容。 标签的参数是所要包含的模板名称，可以是一个变量，也可以是用单/双引号硬编码的字符串。 每当在多个模板中出现相同的代码时，就应该考虑是否要使用 {% include %} 来减少重复。  
{% endraw %}
``` template
# mypage.html

<html>
<body>
{% include "includes/nav.html" %}
<h1>{{ title }}</h1>
</body>
</html>

# includes/nav.html

<div id="nav">
    You are in: {{ current_section }}
</div>
```
##### 模板继承
模板继承就是先构造一个基础框架模板，而后在其子模板中对它所包含站点公用部分和定义块进行重载。
``` template
#base.html

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
<html lang="en">
<head>
    <title>{% block title %}{% endblock %}</title>
</head>
<body>
    <h1>My helpful timestamp site</h1>
    {% block content %}{% endblock %}
    {% block footer %}
    <hr>
    <p>Thanks for visiting my site.</p>
    {% endblock %}
</body>
</html>
```
{% raw %}
我们使用一个以前已经见过的模板标签： {% block %} 。 所有的 {% block %} 标签告诉模板引擎，子模板可以重载这些部分。每个{% block %}标签所要做的是告诉模板引擎，该模板下的这一块内容将有可能被子模板覆盖。
{% endraw %}
``` template
#current_datetime.html 

{% extends "base.html" %}

{% block title %}The current time{% endblock %}

a% block content %}
<p>It is now {{ current_date }}.</p>
{% endblock %}
```
在加载 current_datetime.html 模板时，模板引擎发现了 {% raw %}{% extends %}{% endraw %} 标签， 注意到该模板是一个子模板。模板引擎立即装载其父模板，即base.html 。  
此时，模板引擎注意到 base.html 中的三个 {% raw %}{% block %} 标签，并用子模板的内容替换这些 block 。因此，引擎将会使用我们在 { block title %} 中定义的标题，对 {% block content %} 也是如此。 所以，网页标题一块将由 {% block title %}替换，同样地，网页的内容一块将由 {% block content %}{% endraw %}替换。
以下是使用模板继承的一些诀窍：
- {% raw %}如果在模板中使用 {% extends %} ，必须保证其为模板中的第一个模板标记。 否则，模板继承将不起作用。{% endraw %}
- {% raw %}一般来说，基础模板中的 {% block %} 标签越多越好。记住，子模板不必定义父模板中所有的代码块，因此你可以用合理的缺省值对一些代码块进行填充，然后只对子模板所需的代码块进行（重）定义。 俗话说，钩子越多越好。{% endraw %}
- {% raw %}如果发觉自己在多个模板之间拷贝代码，你应该考虑将该代码段放置到父模板的某个 {% block %} 中。{% endraw %}
- {% raw %}如果你需要访问父模板中的块的内容，使用 {{ block.super }}这个标签吧，这一个魔法变量将会表现出父模板中的内容。 如果只想在上级代码块基础上添加内容，而不是全部重载，该变量就显得非常有用了。{% endraw %}
- {% raw %}不可同一个模板中定义多个同名的 {% block %} 。存在这样的限制是因为block 标签的工作方式是双向的。也就是说，block 标签不仅挖了一个要填的坑，也定义了在父模板中这个坑所填充的内容。 如果模板中出现了两个相同名称的 {% block %} 标签，父模板将无从得知要使用哪个块的内容。{% endraw %}
- {% raw %}{% extends %} 对所传入模板名称使用的加载方法和 get_template() 相同。 也就是说，会将模板名称被添加到 TEMPLATE_DIRS 设置之后。{% endraw %}
- {% raw %}多数情况下，{% extends %} 的参数应该是字符串，但是如果直到运行时方能确定父模板名，这个参数也可以是个变量。 这使得你能够实现一些很酷的动态功能。{% endraw %}
