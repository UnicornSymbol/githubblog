---
title: Centos6.5升级gcc4.4.7到4.8.2  
date: 2017-03-30 18:22:29  
tags: [centos6.5,gcc]  
categories: shell
---

由于centos6.5自带的gcc4.4.7不能支持c++11的特性，所以希望升级到4.8.2。  
##### 下载源码安装包
``` bash
wget http://ftp.gnu.org/gnu/gcc/gcc-4.8.2/gcc-4.8.2.tar.bz2
tar -jxvf gcc-4.8.2.tar.bz2
```
<!-- more -->
##### 下载编译所需依赖
``` bash
cd gcc-4.8.2
./contrib/download_prerequisites
```
##### 建立目录供编译输出
``` bash
mkdir gcc-build-4.8.2
cd gcc-build-4.8.2
```
##### 生成Makefile文件
``` bash
../configure -enable-checking=release -enable-languages=c,c++ -disable-multilib
```
##### 编译安装
```
make && make install
```
##### 验证
```
[root@localhost gcc-build-4.8.2]# gcc -v
使用内建 specs。
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/local/libexec/gcc/x86_64-unknown-linux-gnu/4.8.2/lto-wrapper
目标：x86_64-unknown-linux-gnu
配置为：../configure -enable-checking=release -enable-languages=c,c++ -disable-multilib
线程模型：posix
gcc 版本 4.8.2 (GCC) 
```
##### 参考资料
[CentOS6.5手动升级gcc4.8.2](http://www.centoscn.com/image-text/config/2015/0206/4643.html)  
[CentOS6.6升级gcc4.8教程](http://www.centoscn.com/image-text/config/2015/0823/6041.html)
