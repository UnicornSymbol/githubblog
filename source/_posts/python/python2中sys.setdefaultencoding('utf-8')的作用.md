---
title: python2中sys.setdefaultencoding('utf-8')的作用  
date: 2017-11-26 14:12:35  
tags: [python]  
categories: python
---
在python中，编码解码其实是不同编码系统间的转换，默认情况下，转换目标是Unicode，即编码unicode→str，解码str→unicode，其中str指的是字节流，而str.decode是将字节流str按给定的解码方式解码，并转换成utf-8形式，u.encode是将unicode类按给定的编码方式转换成字节流str。注意调用encode方法的是unicode对象，生成的是字节流；调用decode方法的是str对象（字节流），生成的是unicode对象。若str对象调用encode会默认先按系统默认编码方式decode成unicode对象再encode，忽视了中间默认的decode往往导致报错。  
<!-- more -->  
比如有如下代码：
```
#! /usr/bin/env python 
# -*- coding: utf-8 -*- 
s = '中文字符'  # 这里的 str 是 str 类型的，而不是 unicode 
s.encode('gb2312') 
```
这句代码将 s 重新编码为 gb2312 的格式，即进行 unicode -> str的转换。因为s本身就是str类型的，因此Python 会自动的先将s解码为unicode，然后再编码成gb2312。因为解码是python自动进行的，我们没有指明解码方式，python 就会使用 sys.defaultencoding 指明的方式来解码。很多情况下sys.defaultencoding为ANSCII，如果 s 不是这个类型就会出现以下错误：  
```
UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-3: ordinal not in range(128)
```
对于这种情况，我们有两种方法来改正错误：  
1. 明确指出s的编码方式
```
#! /usr/bin/env python 
# -*- coding: utf-8 -*- 
s = '中文字符' 
s.decode('utf-8').encode('gb2312') 
```
2. 更改 sys.defaultencoding 为文件的编码方式  
```
#! /usr/bin/env python 
# -*- coding: utf-8 -*- 
import sys 
reload(sys) #Python2.5初始化后删除了sys.setdefaultencoding 方法，我们需要重新载入 
sys.setdefaultencoding('utf-8') 

str = '中文字符' 
str.encode('gb2312')
```
参考资料：  
[python中sys.setdefaultencoding('utf-8')的作用](https://www.cnblogs.com/guosq/p/6378639.html)
