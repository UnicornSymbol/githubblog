---
title: Django之model详解  
date: 2017-04-11 11:20:26  
tags: [python,django]  
categories: django
---

##### 创建表
&ensp;&ensp;Django中遵循 Code Frist 的原则，即：根据代码中定义的类来自动生成数据库表。  
<!-- more -->
###### 基本结构
``` python
from django.db import models

# Create your models here.

class userinfo(models.Model):
    nid = models.AutoField(primary_key=True)
    username = models.CharField(max_length=32)
    email = models.EmailField()
    ip = models.GenericIPAddressField()
    memo = models.TextField()
    img = models.ImageField()
    usertype=models.ForeignKey("usertype",null=True,blank=True)

class usertype(models.Model):
    name = models.CharField(max_length=32)
    def __str__(self):
        return self.name
```
- 常用字段
```
models.AutoField　　自增列 = int(11)
　如果没有的话，默认会生成一个名称为 id 的列，如果要显示的自定义一个自增列，必须将给列设置为主键 primary_key=True。
models.CharField　　字符串字段
　必须 max_length 参数
models.BooleanField　　布尔类型=tinyint(1)
　不能为空，Blank=True
models.ComaSeparatedIntegerField　　用逗号分割的数字=varchar
　继承CharField，所以必须 max_lenght 参数
models.DateField　　日期类型 date
　对于参数，auto_now = True 则每次更新都会更新这个时间；auto_now_add 则只是第一次创建添加，之后的更新不再改变。
models.DateTimeField　　日期类型 datetime
　同DateField的参数
models.Decimal　　十进制小数类型 = decimal
　必须指定整数位max_digits和小数位decimal_places
models.EmailField　　字符串类型（正则表达式邮箱） =varchar
　对字符串进行正则表达式
models.FloatField　　浮点类型 = double
models.IntegerField　　整形
models.BigIntegerField　　长整形
　　integer_field_ranges = {
　　　　'SmallIntegerField': (-32768, 32767),
　　　　'IntegerField': (-2147483648, 2147483647),
　　　　'BigIntegerField': (-9223372036854775808, 9223372036854775807),
　　　　'PositiveSmallIntegerField': (0, 32767),
　　　　'PositiveIntegerField': (0, 2147483647),
　　}
models.IPAddressField　　字符串类型（ip4正则表达式）
models.GenericIPAddressField　　字符串类型（ip4和ip6是可选的）
　参数protocol可以是：both、ipv4、ipv6
　验证时，会根据设置报错
models.NullBooleanField　　允许为空的布尔类型
models.PositiveIntegerFiel　　正Integer
models.PositiveSmallIntegerField　　正smallInteger
models.SlugField　　减号、下划线、字母、数字
models.SmallIntegerField　　数字
　数据库中的字段有：tinyint、smallint、int、bigint
models.TextField　　字符串=longtext
models.TimeField　　时间 HH:MM[:ss[.uuuuuu]]
models.URLField　　字符串，地址正则表达式
models.BinaryField　　二进制
models.ImageField   图片
models.FilePathField 文件
```
- 常用参数
```
1、null=True
　数据库中字段是否可以为空
2、blank=True
　django的Admin中添加数据时是否可允许空值
3、primary_key = False
　主键，对AutoField设置主键后，就会代替原来的自增 id 列
4、auto_now 和 auto_now_add
　auto_now  自动创建---无论添加或修改，都是当前操作的时间
　auto_now_add  自动创建---永远是创建时的时间
5、choices
GENDER_CHOICE = (
        (u'M', u'Male'),
        (u'F', u'Female'),
    )
gender = models.CharField(max_length=2,choices = GENDER_CHOICE)
6、max_length
7、default　　默认值
8、verbose_name　　Admin中字段的显示名称
9、name|db_column　　数据库中的字段名称
10、unique=True　　不允许重复
11、db_index = True　　数据库索引
12、editable=True　　在Admin里是否可编辑
13、error_messages=None　　错误提示
14、auto_created=False　　自动创建
15、help_text　　在Admin中提示帮助信息
16、validators=[]
17、upload-to  上传到哪个位置,更多与image,filepath配合使用
```
###### 连表结构
一对多:models.ForeignKey(其他表)  
多对多:models.ManyToManyField(其他表)  
一对一:models.OneToOneField(其他表)  
应用场景:  
一对多：当一张表中创建一行数据时，有一个单选的下拉框（可以被重复选择）  
多对多：在某表中创建一行数据是，有一个可以多选的下拉框  
一对一：在某表中创建一行数据时，有一个单选的下拉框（下拉框中的内容被用过一次就消失了）  
##### 表操作
###### 基本操作
```
# 增
models.Tb1.objects.create(c1='xx', c2='oo')  增加一条数据，可以接受字典类型数据 **kwargs
或
obj = models.Tb1(c1='xx', c2='oo')
obj.save()

# 查
models.Tb1.objects.get(id=123)  # 获取单条数据，不存在则报错（不建议）
models.Tb1.objects.all()    # 获取全部
models.Tb1.objects.filter(name='seven') # 获取指定条件的数据

# 删
models.Tb1.objects.filter(name='seven').delete() # 删除指定条件的数据

# 改
models.Tb1.objects.filter(name='seven').update(gender='0')  # 将指定条件的数据更新，均支持 **kwargs
obj = models.Tb1.objects.get(id=1)
obj.c1 = '111'
obj.save()      # 修改单条数据
```
###### 进阶操作
```
# 获取个数
models.Tb1.objects.filter(name='seven').count()

# 大于，小于
models.Tb1.objects.filter(id__gt=1)              # 获取id大于1的值
models.Tb1.objects.filter(id__lt=10)             # 获取id小于10的值
models.Tb1.objects.filter(id__lt=10, id__gt=1)   # 获取id大于1 且 小于10的值

# in
models.Tb1.objects.filter(id__in=[11, 22, 33])   # 获取id等于11、22、33的数据
models.Tb1.objects.exclude(id__in=[11, 22, 33])  # not in

# contains
models.Tb1.objects.filter(name__contains="ven")
models.Tb1.objects.filter(name__icontains="ven") # icontains大小写不敏感
models.Tb1.objects.exclude(name__icontains="ven")

# range
models.Tb1.objects.filter(id__range=[1, 2])   # 范围bettwen and

# 其他类似
# startswith，istartswith, endswith, iendswith,

# order by
models.Tb1.objects.filter(name='seven').order_by('id')    # asc
models.Tb1.objects.filter(name='seven').order_by('-id')   # desc

# limit 、offset
models.Tb1.objects.all()[10:20]

# group by  去重 values('id')在annotate前面就是为了根据id生成group by
#需要固定导入,称为聚合函数
from django.db.models import Count, Min, Max, Sum
models.Tb1.objects.filter(c1=1).values('id').annotate(c=Count('num'))
moddels.Tb1.objects.filter(c1=1).values('id').annotate(c=Count('id')).values('id'','name')
# SELECT "app01_tb1"."id", COUNT("app01_tb1"."num") AS "c" FROM "app01_tb1" WHERE "app01_tb1"."c1" = 1 GROUP BY "app01_tb1"."id"
```
##### 参考资料
[Django之model详解](http://www.cnblogs.com/ccorz/p/5845711.html)
