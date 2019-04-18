---
layout: post
title:  "Java 多线程学习(一)：线程"
date: 2017-11-27 15:22:16
tags: 
    - java
    - Java Thread 
categories: Java 多线程学习 
---

{:toc}

线程，作为CPU调度的最基本单位以及任务的执行者，其存在的数量决定了程序的执行吞吐量。

作为虚拟机，JVM也对物理计算机的多线程技术进行了实现，即在一个JVM虚拟机内可以执行多个Java线程。而对于物理机而言，绝大多数操作系统针对JVM是把内核线程（kernel thread）与 JVM线程进行一一对应的。

上面这些理解起来可能会比较绕，简单来说：**物理计算机中一个JVM进程拥有多个内核线程（kernel thread）,一个内核线程对应JVM中的Java线程(java.lang.Thread)。**如下图：  
![](https://s1.ax1x.com/2017/11/23/R74yR.jpg)

所以，对于JVM而言其部署的服务器的可开启的最大线程数既是JVM可开启的最大线程数。

<!--more-->

## 线程的创建与运行
使用Thread类创建多线程应用有两种方式：

	1. 继承`java.lang.Thread`基础线程对象，并覆写其中的 `run()` 方法。  
	2. 实现`java.lang.Runnable`接口的`run()`方法，并new Thread对象传入Runnable接口实现对象。  

相对于继承Thread类而实现多线程的方式，个人更推荐使用实现Runnable接口的方式实现多线程，主要的优势有：
  
	1. 避免java单继承特性带来的局限性
	2. 增强程序的健壮性，代码能够被多个线程共享，代码与数据是独立的
	3. 适合多个相同程序代码的线程区处理同一资源的情况  

最后，完成线程对象的创建过后，使用 `start()` 方法启动线程。

**对于Thread类来说，使用start()方法并不是立即执行多线程代码，而是将该线程变成可运行态(Runnable)，实际运行时间是由操作系统决定的。**

## 线程状态
在线程创建成功到结束任务的过程中线程都会拥有一个状态。这个状态在Thread的内部枚举类State中定义了：
{% highlight java linenos%}
public enum State {
	/**
     * 尚未开始状态
     * 
     * 新建且未执行start()方法的线程的状态
     */
    NEW,

    /**
     * 等待执行状态
     * 
     * 已执行start()方法正在等待CPU执行的线程状态
     */
    RUNNABLE,

	/**
     * 阻塞状态
     * 
     * 该线程所需的资源被其他线程占用(blocked)，线程进入等待资源释放的状态
     */
    BLOCKED,

	/**
     * 无限等待状态
     * 
     * 等待其余线程调用该线程的notify()或notifyAll()方法才能解除这个状态
     */
    WAITING,

	/**
     * 有时间限制的等待状态
     * 
     * 等待时间限制结束解除等待状态，
     * 或调用该线程的notify()或notifyAll()方法亦能解除该状态
     */
    TIMED_WAITING,

	/**
     * 线程中止的状态，这个线程已经完整地执行了它的任务
     */
    TERMINATED;
}
{% endhighlight %}
上面的代码注释中已经简单对每一个状态进行了简单的说明，其中阻塞状态和两种等待状态我将在下一篇博文中对同步与线程锁进行详细的介绍。

## 线程的停止
### 自动停止
正常情况下，线程任务完成过后就会自动结束生命周期，并设置状态为`TERMINATED`。
{% highlight java linenos%}
public static void main(String[] args) throws ExecutionException, InterruptedException {
    Thread threadA = new Thread();
    threadA.start();
    Thread.sleep(1000);
    System.out.println(threadA.getState().name());	//结果为：TERMINATED
}
{% endhighlight %}

当线程出现错误的时候，会立即结束生命周期，并设置状态为`TERMINATED`。
{% highlight java linenos%}
public class ThrowExceptionRunnable implements Runnable {
    @Override
    public void run() {
        try {
            throw new Exception("123");
        } catch (Exception e) {
        }
    }
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new ThrowExceptionRunnable());
        thread.start();
        Thread.sleep(1000);
        System.out.println(thread.getState().name());	//结果为：TERMINATED
    }
}
{% endhighlight %}

### 手动停止
手动停止线程的方法有如下四种：

	1. 异常停止：利用线程错误自动停止的机制，抛出异常使线程停止。
	2. 暴力停止：调用线程的`stop()`方法，但是这个方法已经设置为过时。因为会导致某些收尾程序未完成，对象"解锁"，导致数据不同步。
	3. 使用return停止线程：当方法执行的代码遇到了return时，这个方法就会结束。如果在`run()`方法中使用return也就代表了线程的执行结束。
	4. 使用`interrupt()`方法使中断状态的线程结束运行

由于篇幅原因，最后的一个停止线程的方法就无法详细分析了，大家可以参考以下两篇文章：
> - [Java多线程系列--“基础篇”09之 interrupt()和线程终止方式](http://www.cnblogs.com/skywang12345/p/3479949.html)
> - [如何停止一个正在运行的java线程](http://ibruce.info/2013/12/19/how-to-stop-a-java-thread/)

## 结束

>**本文部分文本来源于互联网**

>**感谢以下文章提供的灵感和帮助**  
> - [JVM 内部运行线程介绍](http://ifeve.com/jvm-thread/)   
> - [深入理解JVM之高效并发 - Java内存模型与线程、线程安全与锁优化](http://wanglizhi.github.io/2016/07/16/JVM-JMM-And-Thread/#%E7%BA%BF%E7%A8%8B%E7%9A%84%E5%AE%9E%E7%8E%B0)  
> - [Java线程与Linux内核线程的映射关系](http://blog.sina.com.cn/s/blog_605f5b4f010198b5.html)
> - [聊聊JVM（五）从JVM角度理解线程](http://blog.csdn.net/iter_zc/article/details/41843595)  
> - [Java 编程要点 - 进程（Processes ）和线程（Threads）](https://waylau.gitbooks.io/essential-java/docs/concurrency-Processes%20and%20Threads.html)  
