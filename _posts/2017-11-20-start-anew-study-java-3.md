---
layout: post
title:  "从头开始分析JDK - 深入“基层” （三）"
date: 2017-11-21 17:21:50
tags: 
    - java
    - JDK
    - JVM
    - Java (JVM) Memory Model
categories: 从头开始分析JDK 
---

{:toc}

# 数据仓库：堆、栈 - 关于JVM的内存模型
---
![](https://s1.ax1x.com/2017/11/21/2aGPH.png)
<center>JVM 运行时数据区</center>

在JVM的运行时内存数据区中包含了：`Method Area(方法区)`、`Heap Area(堆区)`、`Stack Area(栈区)`、`PC Registers(程序计数器)`和`Native Method Stack(本地方法栈)`。  

<!--more-->

> 1. `Method Area(方法区)`中存储了所有被虚拟机加载的**类信息**、**常量**、**静态变量**、**即时编译后的代码**等数据。该区域使用`永久代`实现，并且该区域的数据是**所有线程共享使用**的。
> 2. `Heap Area(堆区)`是JVM中所占内存最大的一块，它唯一的目的是存储对象的实例，几乎所有的对象实例都在这个区域中被分配内存。该区域使用`新生代`和`老年代`实现，同时该区域的数据也是**所有线程共享使用**的。
> 3. `Stack Area(栈区)`由线程创建，其中存储了每一个**方法实例**私有的局部变量表、操作数栈、动态链接、方法出口等信息，每一个方法实例都会创建一个**私有**的`栈帧(Stack Frame)`并放入这个区域。当线程关闭的时候，该线程的栈区内存也将会释放。
> 4. `PC Registers(程序计数器)`全称为：Program Counter Registers,每一个线程都会创建一个程序计数器。线程执行过程中，它会存储正在执行的方法在`Method Area(方法区)`中存储的位置。
> 5. `Native Method Stack(本地方法栈)`是用Java以外的语言编写的本机代码栈，通过JNI（Java Native Interface）调用。 由于它是一个“本地”堆栈，这个堆栈的行为完全依赖于底层操作系统。

完整的JVM内存模型如下图：
![](https://s1.ax1x.com/2017/11/22/2OLcQ.png)

接下来我们将对最重要的三块：`Method Area(方法区)`、`Heap Area(堆区)`和`Stack Area(栈区)`进行详细分析。

## Method Area 方法区 - 编译后的代码仓库

方法区是一个所有**线程共享**的区域，由虚拟机启动的时候创建，对于GC来说它位于`永生代`①中。方法区存储了所有被虚拟机加载的**类信息**、**常量**②、**静态变量**、**即时编译后的代码**等数据。

虽然Java 虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做`Non-Heap`（非堆），目的应该是与Java 堆区分开来。但是，和堆区一样它并不需要连续的内存和可选择固定大小或可扩展，除此之外他和堆区不一样的是它并不需要完全的GC。针对该区域的内存回收，主要是**常量池**的回收和对**类型的卸载**。

### Runtime constant pool 运行时常量池
运行时常量池是方法区的子区域，存放着在编译时已知的各种字面量和符号引用。除此之外，我们还可以以使用类似String类中的intern()方法添加新的常量。

1. 字面量②：如文本字符串、final的常量值  
2. 符号引用：编译语言层面的概念，包括以下3类：  
	- 类和接口的全限定名  
	- 字段的名称和描述符  
	- 方法的名称和描述符  

①：在JDK8中，永生代PermGen已经被彻底移除，取而代之的是metaspace数据区，使用native内存，申请和释放由虚拟机负责管理。  
②：在JDK7中，**字符串常量**已经从永久代移除。

## Heap 堆 - 对象实例的存储仓库和GC的主战场

作为JVM内存中占比最大的一块，堆区存储了所有使用`new`运算符实例化过后的`类实例`和`数组`。堆区的创建时间是在虚拟机启动的时候创建的，而且改区域内的内容是所有线程共享的。
> JVM默认的堆区内存分配如下：  
> 初始分配的内存由 `-Xms` 指定，默认是物理内存的1/64  
> 最大分配的内存由 `-Xmx` 指定，默认是物理内存的1/4  

> 默认空余堆内存 `小于40%` 时，JVM 就会增大堆直到 `-Xmx` 的最大限制，可以由 `-XX:MinHeapFreeRatio` 指定  
> 默认空余堆内存 `大于70%` 时，JVM 会减少堆直到 `-Xms` 的最小限制，可以由 `-XX:MaxHeapFreeRatio` 指定  

堆区分为两个大块：`Young Generation(新生代)`和`Old Generation(老年代)`。之所以会分为两个区域，是为了JAVA中最重要的特性之一的`Java GC（Garbage Collection，垃圾收集，垃圾回收）机制`。  
![](https://s1.ax1x.com/2017/11/22/2Oq1g.png)

### Young Generation(新生代)
1. 按照JVM的规则，当一个新的对象产生后将会被分配到新生代中的`Eden区`中。需要注意的是，如果产生的新对象过大，比如超大型数组等，该对象将会被直接放入到`Old Gen老生代`中。
2. 如果`Eden区域`内存不够用的时候，就会触发GC机制。这个时候，GC会将`Eden`中无用的对象回收，如果该对象依然在使用那么它将会被`copy`到其中一个`Survivor`中。
3. `From Survivor`和`To Survivor`是相对的，GC选中其中一个为`From Survivor`，而另一个就为`To Survivor`，在`From Survivor`接收对象之前，`From Survivor`中已存在的对象会被`copy`到`To Survivor`中，并且清空`From Survivor`。
4. 当`Survivor`中的对象经历了15次（默认，可通过参数调整）GC过后还存在，该对象会被移动到`Old Gen(老生代)`中。
5. 每一次GC会使`Eden区域`清空。
### Old Generation(老年代)
1. 在`Old Gen`中，主要存放应用程序中生命周期长的内存对象。
2. 当`Old Gen`空间足够时，To Survivor区的对象会被移到`Old Gen`，否则会被保留在To Survivor区。
3. 当`Old Gen`空间不够时，会触发仅在`Old Gen`进行的GC。
4. 当`Old Gen`和位于方法区中的`Permanet Generation(永生代)`都表示空间不足或者用户显式调用System.GC, RMI等时，将会对

ps.关于JVM的GC是非常重要的一块内容，我将在以后的博文中进行深入分析。

## VM Stack 虚拟机栈 - 方法执行的内存模型

1. 虚拟机栈是Java方法执行的内存模型，每一个线程都拥有一个虚拟机栈，其生命周期与线程的生命周期一致。
2. 每一个方法在执行的时候都会创建一个栈帧，栈帧中存放局部变量表、操作数栈、动态链接、方法出口等信息。
3. 线程在执行方法时候实际上就是**方法栈帧入栈出栈的过程**。
4. 虚拟机栈**大小是默认固定的**，对于现代JVM来说虚拟机栈又**拥有动态扩展的能力**。如果线程需要创建一个比固定栈大小还大的虚拟机栈的话，就会抛出StackOverflowError。

![](https://s1.ax1x.com/2017/11/23/R6KnH.png)
<center>VM Stack 虚拟机栈</center>

## 其他：关于G1回收机制
G1回收机制是从JDK7开始引入的新一代垃圾回收机制，并在JDK8中基本稳定。

[Oracle官方对于G1机制的描述](http://www.oracle.com/technetwork/tutorials/tutorials-1876574.html):
>The heap is partitioned into a set of equal-sized heap regions, each a contiguous range of virtual memory. Certain region sets are assigned the same roles (eden, survivor, old) as in the older collectors, but there is not a fixed size for them. This provides greater flexibility in memory usage.  
>
>相对于传统的GC来说，G1最大的特点是先将内存分为一组相等大小的堆区域，每个区域都是连续的虚拟内存范围。由虚拟机对不同的区域赋予不同的角色(Eden、Survivor、Old)，但是他们都没有固定的大小，这样提供了更大的内存使用灵活性。  
![](https://s1.ax1x.com/2017/11/23/R4x9s.png)


## 结束

>**本文部分文本来源于互联网**

>**感谢以下文章提供的灵感和帮助**  
> - [Memory Architecture Of JVM(Runtime Data Areas)](https://hackthejava.wordpress.com/2015/01/09/memory-architecture-by-jvmruntime-data-areas/)  
> - [ 纳达丶无忌：JVM内存模型你只要看这一篇就够了 ](http://www.jianshu.com/p/c9ac99b87d56)  
> - [ 阳光岛主：Java 内存模型及GC原理 ](http://blog.csdn.net/ithomer/article/details/6252552)  
> - [ 奔跑的小邵：JVM内存模型及内存分配过程](http://www.cnblogs.com/windlaughing/archive/2013/05/27/3101650.html)
> - [JVM系列二:GC策略&内存申请、对象衰老](http://www.cnblogs.com/redcreen/archive/2011/05/04/2037056.html)