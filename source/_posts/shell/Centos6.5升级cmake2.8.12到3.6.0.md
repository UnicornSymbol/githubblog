---
title: Centos6.5升级cmake2.8.12到3.6.0  
date: 2017-03-31 16:59:40  
tags: [centos6.5,cmake]  
categories: shell
---

##### 安装依赖
cmake3.6.0依赖于更高版本的gcc，需先升级gcc([Centos6.5升级gcc4.4.7到4.8.2](https://unicornsymbol.github.io/2017/03/30/shell/centos6.5%E5%8D%87%E7%BA%A7gcc4.4.7%E5%88%B04.8.2/))。
``` bash
yum install ncurses-devel
```
<!-- more -->
##### 安装cmake
``` bash
wget https://cmake.org/files/v3.6/cmake-3.6.0.tar.gz
tar xf cmake-3.6.0.tar.gz
cd cmake-3.6.0
./configure
make && make install
```
**注意：** 在configure时如果报以下错误，是由于缺少GLIBCXX__3.4.15版本或更高版本，通过strings命令查看是否包含此版本，如果没有，由于之前升级过gcc，查找到最新的libstdc++将旧的覆盖的：  
![cmake](/img/shell/cmake.png)
``` bash
strings /usr/lib64/libstdc++.so.6 |grep GLIBCXX
 GLIBCXX_3.4
 GLIBCXX_3.4.1
 GLIBCXX_3.4.2
 GLIBCXX_3.4.3
 GLIBCXX_3.4.4
 GLIBCXX_3.4.5
 GLIBCXX_3.4.6
 GLIBCXX_3.4.7
 GLIBCXX_3.4.8
 GLIBCXX_3.4.9
 GLIBCXX_3.4.10
 GLIBCXX_3.4.11
 GLIBCXX_3.4.12
 GLIBCXX_3.4.13
 GLIBCXX_FORCE_NEW
 GLIBCXX_DEBUG_MESSAGE_LENGTH
cp /usr/local/lib64/libstdc++.so.6.0.18 /usr/lib64/
cd /usr/lib64/
cp libstdc++.so.6 libstdc++.so.6.bak
cp libstdc++.so.6.0.18 libstdc++.so.6
strings libstdc++.so.6|grep GLIBCXX
 GLIBCXX_3.4
 GLIBCXX_3.4.1
 GLIBCXX_3.4.2
 GLIBCXX_3.4.3
 GLIBCXX_3.4.4
 GLIBCXX_3.4.5
 GLIBCXX_3.4.6
 GLIBCXX_3.4.7
 GLIBCXX_3.4.8
 GLIBCXX_3.4.9
 GLIBCXX_3.4.10
 GLIBCXX_3.4.11
 GLIBCXX_3.4.12
 GLIBCXX_3.4.13
 GLIBCXX_3.4.14
 GLIBCXX_3.4.15
 GLIBCXX_3.4.16
 GLIBCXX_3.4.17
 GLIBCXX_3.4.18
 GLIBCXX_3.4.19
 GLIBCXX_FORCE_NEW
 GLIBCXX_DEBUG_MESSAGE_LENGTH
```
##### 添加环境变量
``` bash
cat >> /etc/profile << EOF
#cmake tools
PATH=/home/operation/cmake-3.6.0/bin:$PATH
export PATH
EOF
. /etc/profile
```
##### 验证
```
cmake --version
 cmake version 3.6.0
 CMake suite maintained and supported by Kitware (kitware.com/cmake).
```

##### 参考资料
[centos 6.5下cmake工具的安装与配置](http://blog.csdn.net/xushouwei/article/details/51933487)  
[CentOS-6.3安装配置cmake](http://www.cnblogs.com/zhoulf/archive/2013/02/03/2890717.html)  
['GLIBCXX_3.4.15' not found错误](http://www.cnblogs.com/weinyzhou/p/4983306.html)
