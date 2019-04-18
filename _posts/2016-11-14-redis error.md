---
layout: post
title: 安装Redis 错误You need tcl 8.5 or newer in order to run the Redis test.
tags:
  - 运维
  - redis
categories: redis
published: true
---  
## You need tcl 8.5 or newer in order to run the Redis test.

### 安装Redis，运行make test的时候出错：

>You need tcl 8.5 or newer in order to run the Redis test
make: *** [test] Error 1
 
### 安装tcl就行了：
 
```code
wget http://downloads.sourceforge.net/tcl/tcl8.6.1-src.tar.gzsudotar xzvf tcl8.6.1-src.tar.gz  -C /usr/local/
cd  /usr/local/tcl8.6.1/unix/
sudo ./configure
sudomakesudomakeinstall
```