---
title: Centos6.5升级glibc2.12到2.15  
date: 2017-04-05 09:37:30  
tags: [centos6.5,glibc]  
categories: shell
---

&ensp;&ensp;Centos6.5默认的 glibc版本最高为2.12， 而在进行一些开发时项目所依赖的包往往需要更高版本的glibc库支持，因此在不升级系统的前提下， 需要主动更新系统glibc库。 一般遇到错误libc.so.6: version GLIBC_2.15 not found时表示需要对glibc进行升级了。
<!-- more -->
##### 查看系统glibc支持的版本
``` bash
strings /lib64/libc.so.6 | grep GLIBC
 GLIBC_2.2.5
 GLIBC_2.2.6
 GLIBC_2.3
 GLIBC_2.3.2
 GLIBC_2.3.3
 GLIBC_2.3.4
 GLIBC_2.4
 GLIBC_2.5
 GLIBC_2.6
 GLIBC_2.7
 GLIBC_2.8
 GLIBC_2.9
 GLIBC_2.10
 GLIBC_2.11
 GLIBC_2.12
 GLIBC_PRIVATE
# 查看当前glibc的版本
ll /lib64/libc.so.6   
 lrwxrwxrwx. 1 root root 12 Oct  9  2014 /lib64/libc.so.6 -> libc-2.12.so
```
##### 安装升级glibc
``` bash
cd /usr/local/src/
wget http://mirror.bjtu.edu.cn/gnu/glibc/glibc-2.15.tar.gz
tar xf glibc-2.15.tar.gz
mkdir build
cd build/
../glibc-2.15/configure  --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
make && make install
```
##### 验证
``` bash
strings /lib64/libc.so.6 | grep GLIBC
 GLIBC_2.2.5
 GLIBC_2.2.6
 GLIBC_2.3
 GLIBC_2.3.2
 GLIBC_2.3.3
 GLIBC_2.3.4
 GLIBC_2.4
 GLIBC_2.5
 GLIBC_2.6
 GLIBC_2.7
 GLIBC_2.8
 GLIBC_2.9
 GLIBC_2.10
 GLIBC_2.11
 GLIBC_2.12
 GLIBC_2.13
 GLIBC_2.14
 GLIBC_2.15
 GLIBC_PRIVATE
ll /lib64/libc.so.6 
 lrwxrwxrwx 1 root root 12 4月   4 03:11 /lib64/libc.so.6 -> libc-2.15.so
```
##### 误删libc.so.6解决办法
```
LD_PRELOAD=/lib/libc-2.12.so ln -s /lib/libc-2.12.so lib/libc.so.6
```
##### 参考资料
[CentOS6.5系统"libc.so.6: version 'GLIBC_2.15' not found"解决方法](http://blog.csdn.net/hnhuangyiyang/article/details/50392997)  
[Centos6.5升级glibc过程](http://cnodejs.org/topic/56dc21f1502596633dc2c3dc)
