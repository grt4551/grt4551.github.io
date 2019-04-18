---
layout: post
title:  "Java 多线程学习(二)：线程池"
date: 2017-11-28 14:21:04
tags: 
    - java
    - Java Thread 
categories: Java 多线程学习 
---

{:toc}

JDK1.5中对于多线程编程来说java引入了新的启动、调度和管理线程的API。**Executor框架**就是其中之一，其内部使用了一个新的概念`Thread Pool(线程池)`。

简单来说，`Thread Pool(线程池)`就是装着一堆线程的仓库。我们可以对一个线程进行操作，也可以直接对整个线程池进行操作。为了防止生成的线程过多导致系统错误或者分配线程过少而浪费服务器性能，我们需要根据任务的不同对线程池进行合理的配置：

	1. 任务优先级
	2. 任务执行时间
	3. 任务消耗资源类型：CPU、IO、多重混合
	4. 任务是否依赖其他服务，例如数据库服务等

<!--more-->
## 线程池的创建 
在Executor框架中，线程池由`ThreadPoolExecutor(线程池类)`实现，其构造方法如下：
{% highlight java linenos%}
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,	
                          long keepAliveTime,	
                          TimeUnit unit,		
                          BlockingQueue<Runnable> workQueue,
						  ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
	...
}
{% endhighlight %}
参数说明：  
> - corePoolSize： 核心线程数量  
> - maximumPoolSize： 最大线程数量  
> - keepAliveTime： 超时时间，当线程数量大于核心线程数时，超过该时间的线程将会被kill  
> - unit： 超时时间单位  
> - workQueue： Runnable工作任务阻塞队列。如果线程池被占满，这个队列将用于存储多余的Runnable工作任务  
> - threadFactory: 用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字。  
> - handler:饱和策略,当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。

在Executor框架中提供了很多种工厂方法，用于创建不同用途的线程池。为了学习线程池的运行原理，我们就不在此处使用框架中提供的工厂方法了，如果有需要的朋友可以看文章最后的图表。

## 线程池的运行 
线程池在构造前（new操作）是初始状态，一旦构造完成线程池就进入了执行状态RUNNING。严格意义上讲线程池构造完成后并没有线程被立即启动，只有进行"预启动"或者接收到任务的时候才会启动线程。

线程池的运行是继承于`AbstractExecutorService`类，而`AbstractExecutorService`实现了`ExecutorService`接口，然后`ExecutorService`接口又是`Executor`接口的子类。所以他们的关系如下：
>`ThreadPoolExecutor` >ex> `AbstractExecutorService` >impl> `ExecutorService` >ex> `Executor`  

其中，`Executor`接口作为顶级接口，它定义了线程执行任务的方法:
{% highlight java linenos%}
void execute(Runnable command);
{% endhighlight %}
`ExecutorService`接口在，`Executor`接口的基础上添加了更多功能包括运行、批量运行和停止等，其中核心的两个运行方法是：
{% highlight java linenos%}
<T> Future<T> submit(Callable<T> task);

Future<?> submit(Runnable task);
{% endhighlight %}
相对于`Executor`接口的`execute()`方法，`ExecutorService`接口的`submit()`的方法会**阻塞当前线程**，并在线程执行完成任务过后将结果返回为`Future`接口对象，其中包含了取消任务、判断当前任务是否被取消、判断当前任务是否完成等功能。
## 线程池的关闭
线程池的关闭方法是在`ExecutorService`接口中定义的：
{% highlight java linenos%}
void shutdown();

List<Runnable> shutdownNow();
{% endhighlight %}
其中：

 - shutdown方法：平缓的关闭线程池。表示停止线程池接收新的任务，同时**等待所有任务执行完成**，包括在阻塞队列中等待的任务。在此方法运行过程中，线程池的状态为SHUTDOWN。
 - shutdownNow方法：立即关闭线程池。表示停止线程池接收新的任务，同时**取消所有执行中的任务**,包括在阻塞队列中等待的任务,并且**返回从未执行过的任务列表**。在此方法执行过程中，线程池的状态为STOP。

## 最后
Executor框架提供的默认线程池工厂方法图表(图片源自博客：[并发新特性—Executor框架与线程池](http://blog.csdn.net/ns_code/article/details/17465497)):
![](https://s1.ax1x.com/2017/12/04/5yu8I.md.png)

## 结束

>**本文部分文本来源于互联网**

>**感谢以下文章提供的灵感和帮助** 
> - [多线程的实现方法](http://wiki.jikexueyuan.com/project/java-concurrency/function.html) 
> - [林炳文Evankaka - Java多线程学习（吐血超详细总结）](http://blog.csdn.net/evankaka/article/details/44153709)  
> - [【Java并发编程】之十九：并发新特性—Executor框架与线程池（含代码）](http://blog.csdn.net/ns_code/article/details/17465497)  
> - [Java并发——线程池Executor框架](http://www.cnblogs.com/shijiaqi1066/p/3412300.html)