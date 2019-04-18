---
layout: post
title:  "Java 多线程学习(三)：并发 - 特性与线程安全"
date: 2017-11-29 18:02:52
tags: 
    - java
    - Java Thread 
categories: Java 多线程学习 
---

{:toc}

随着时代的发展，多核CPU已经取代了过去的单核CPU。硬件技术的飞速发展让服务器的多线程的处理能力大大提升，为了**提高应用程序的吞吐量和多系统协同处理能力**，我们往往会同时运行多个线程去处理多个任务，这就是我们所说的--`线程并发`。

	实际上，并发其实是一种解耦合的策略，它帮助我们把做什么（目标）和什么时候做（时机）分开。
比如，在Java Web应用中我们并不需要手动去为每一个用户请求创建线程。因为，在tomcat等服务容器的支持下，我们的Web应用可以以**单实例多线程**的方式运行，服务容器帮助我们对用户的并发请求进行了处理。这样，Web应用程序员就不用在去关心那些恶心的并发问题了(笑)。

<!--more-->

## 并发的特性
在实际生产过程中，我们会发现并发的线程会产生很多问题，例如：

	1. 多个线程同时修改一个资源，如何让这个资源保障多个线程都能正确处理？
	2. 多个线程同时修改一个资源，如何让这个资源在一个线程修改后其他线程能立即看见修改结果？
	3. 由于**线程的实际运行顺序是由处理器决定的**，如何让多个线程按照既定的顺序执行？

这就是多线程并发的三个特性：**原子性**、**可见性**、**顺序性**

### 原子性
在化学学上，原子(Atom)代表了元素(element)能保持其化学性质的最小单位。类似的，在java多线程应用中的`原子性`就代表了该操作或者说这一段代码是最基础的操作，是**一旦运行起来就不会被其他线程所干扰**的。

我在前面的文章[[Java GC机制](https://www.maxith.com/2017/11/21/start-anew-study-java-4/)]中提到过，java方法运行过程中会由主内存read和write数据到工作内存中。因为这个操作是有锁(lock)的，所以这个操作是**“原子性”**的。

java中存在以下几种原子操作：

	1. 基本类型的读取和赋值是原子性的。但在32位系统中long和double由于长度问题，就没有了原子性。
	2. 所有引用reference的赋值操作。
	3. java.concurrent.Atomic.* 包中所有类的一切操作。

而对于线程来说，除了上面的原生的原子操作之外，我们可以使用 `synchronized` 同步代码块对指定的代码做**加锁处理**，这样就可以实现**一段代码的线程原子性**。
### 可见性
**可见性简单来说就是当有多个线程同时访问一个资源的时候，当一个线程修改了这个资源过后其他线程能立即看见被修改后的资源。**

我们知道，当资源被修改过后并不是直接同步到主内存中的。如果在还未同步到主内存的同时有别的线程读取这个资源，那么就会造成其他线程处理结果有差异。

例如：
{% highlight java linenos%}
public class Visibility implements Runnable{

    private Demo demo;

    public Visibility(Demo demo) {
        this.demo = demo;
    }

    @Override
    public void run() {
        demo.v += " so happy ";
        System.out.println("v = " + demo.v);
    }
}
public class Demo {
    public String v;

    public Demo(String v) {
        this.v = v;
    }

    public static void main(String[] args) {
        Demo demo = new Demo("im");
        new Thread(new Visibility(demo)).start();
        new Thread(new Visibility(demo)).start();
    }
}
{% endhighlight %}
上面这段代码运行结果有概率会出现：`v = im so happy v = im so happy` 的结果。这就是在第一个线程修改后，v的值并没有及时更新到主内存中，导致打印线程没有读取到修改后的结果。

我们可以使用 `synchronized` 同步代码块对该Demo改造就能使结果变成我们希望的：
{% highlight java linenos%}
@Override
public void run() {
    synchronized (demo){
        demo.v += " so happy ";
        System.out.println("v = " + demo.v);
    }
}
{% endhighlight %}
### 顺序性
顺序性指的就是线程的执行顺序。我们都知道线程的执行是由处理器决定的，所以我们无法预测线程执行顺序。

例如：
{% highlight java linenos%}
public class MutiThread implements Runnable{

    @Override
    public void run(){
        System.out.println(Thread.currentThread().getName() + " --- 正在运行");
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 5; i++) {
            new Thread(new MutiThread()).start();
        }
        Thread.sleep(1000);
    }
}
{% endhighlight %}
上面的代码每一次运行都可能会产生的不同的打印结果。为了保证执行线程的顺序，我们可以使用以下几种方式实现：

	1. 使用`synchronized`关键字
	2. 使用`Lock`锁
	3. 使用线程的`join()`方法

关于这几种方法，我将在以后的文章中一一介绍。
## 并发与线程安全
在java中，所谓的`线程安全`其实就是在**多线程环境**下：多个线程同时操作同一个对象时，不会产生**数据不一致或者数据污染**的情况。

我们以最常见的`StringBuffer`和`StringBuilder`来举例：
{% highlight java linenos%}
public class Visibility implements Runnable{

    private Demo demo;

    public Visibility(Demo demo) {
        this.demo = demo;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            demo.stringBuilder.append("a");
            demo.stringBuffer.append("b");
        }
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(demo.stringBuffer.length() + " -------- " + demo.stringBuilder.length());
    }
}
public class Demo {
    public StringBuffer stringBuffer;
    public StringBuilder stringBuilder;

    public Demo(StringBuffer stringBuffer, StringBuilder stringBuilder) {
        this.stringBuffer = stringBuffer;
        this.stringBuilder = stringBuilder;
    }

    public static void main(String[] args) {
        Demo demo = new Demo(new StringBuffer(),new StringBuilder());
        for (int i = 0; i < 100; i++) {
            new Thread(new Visibility(demo)).start();
        }
    }
}
{% endhighlight %}
上面的代码的结果中，`StringBuilder`的结果总是会出现偏差。这就很尴尬了，如果这要是放到高并发的生产环境中，那不是要爆炸？于是，我们把`StringBuilder`称之为：**非线程安全的**。

### 为什么会这样呢？
我们来简单分析一下`StringBuffer`和`StringBuilder`的`append()`方法源码：

StringBuffer:
{% highlight java linenos%}
@Override
public synchronized StringBuffer append(String str) {
    toStringCache = null;
    super.append(str);
    return this;
}
{% endhighlight %}

StringBuilder：
{% highlight java linenos%}
@Override
public StringBuilder append(String str) {
    super.append(str);
    return this;
}
{% endhighlight %}

AbstractStringBuilder:
{% highlight java linenos%}
public AbstractStringBuilder append(String str) {
    if (str == null)
        return appendNull();
    int len = str.length();
    ensureCapacityInternal(count + len);
    str.getChars(0, len, value, count);
    count += len;
    return this;
}
{% endhighlight %}

从上面的源码我们可以发现，`StringBuffer`和`StringBuilder`都是调用的父类`AbstractStringBuilder`的`append()`方法，唯一的区别是StringBuffer在append()方法上添加了synchronized关键字，而StringBuilder没有。而且，父类AbstractStringBuilder中也没有对append()方法使用`synchronized`关键字。

这样就很明显了，synchronized关键字对线程的安全有一定的影响，那么synchronized关键字到底有什么用呢？在下一篇文章我们将进行详细的分析。

### jdk中常用的线程安全的对象
1. 使用`synchronized`关键字保证线程安全

	Timer，TimerTask，Vector，Stack，HashTable，StringBuffer
2. java.util.concurrent.atomic包下的原子类

	常见的是AtomicInteger、AtomicLong等基础对象的实现操作原子性的包装类，原理是使用Unsafe类的本地方法实现线程安全。
3. ConcurrentHashMap等开头为Concurrent的工具类
	
	ConcurrentHashMap为了实现最大程度的线程共享，使用了**分段锁**实现线程的同步。
4. 线程池ThreadPoolExecutor使用了ReentrantLock保证了线程的同步

5. Collections中的synchronizedCollection(Collection c)方法可将一个集合变为线程安全，其内部通过synchronized关键字加锁同步
## 结束

>**本文部分文本来源于互联网**

>**感谢以下文章提供的灵感和帮助**  
> - [Java并发编程的总结与思考](http://www.jianshu.com/p/053943a425c3)
> - [多线程并发相关的3个特性](http://www.jianshu.com/p/1263102348ec) 
> - [java并发特性，原子性、有序性、可见性](http://longload.iteye.com/blog/2307621)  
> - [Java并发-线程安全性](https://my.oschina.net/bendaxia/blog/1511368)
> - [线程安全的问题](https://gavinzhang1.gitbooks.io/java-concurrence-practise/content/xian_cheng_an_quan_de_wen_ti.html)