---
layout: post
title: 安装Redis 2.8.18时报错
tags:
  - 运维
  - redis
categories: redis
published: true
---  

### 安装Redis 2.8.18时报错：
```code
zmalloc.h:50:31: error: jemalloc/jemalloc.h: No such file or directory
zmalloc.h:55:2: error: #error "Newer version of jemalloc required"
make[1]: *** [adlist.o] Error 1
make[1]: Leaving directory `/data0/src/redis-2.6.2/src'
make: *** [all] Error 2
```

### 原因分析

在README 有这个一段话。
>### Allocator  
>Selecting a non-default memory allocator when building Redis is done by setting  
>the `MALLOC` environment variable. Redis is compiled and linked against libc  
>malloc by default, with the exception of jemalloc being the default on Linux  
>systems. This default was picked because jemalloc has proven to have fewer  
>fragmentation problems than libc malloc.  
> 
>To force compiling against libc malloc, use:  
> 
>    % make MALLOC=libc  
> 
>To compile against jemalloc on Mac OS X systems, use:  
> 
>    % make MALLOC=jemalloc

<!--more-->

说关于分配器allocator，如果有MALLOC 这个 环境变量， 会有用这个环境变量的 去建立Redis。
而且libc 并不是默认的 分配器， 默认的是 jemalloc, 因为 jemalloc 被证明 有更少的 fragmentation problems 比libc。
但是如果你又没有jemalloc 而只有 libc 当然 make 出错。 所以加这么一个参数。

## 解决办法

make MALLOC=libc