---
layout: post
title:  "从头开始分析JDK - 深入“基层” （一）"
date: 2017-11-9 18:32:48
tags: 
    - java
    - JDK
    - 源码分析
categories: 从头开始分析JDK
---

{:toc}

<blockquote class="blockquote-center">作为一个JAVA程序开发者,见过太多的技术太多的框架,却对原本JDK的内容知之甚少。为了弥补这个遗憾,我将重新审视分析JDK</blockquote>

# JVM - java运行的基础

## 关于JVM

Java虚拟机（英语：Java Virtual Machine，缩写为JVM），作为java平台运行的基础，它承担了运行java字节码的工作，在[维基百科](https://en.wikipedia.org/wiki/Java_virtual_machine "维基百科")上是这样定义的：

>A Java virtual machine (JVM) is an abstract computing machine that enables a computer to run a Java program. There are three notions of the JVM: specification, implementation, and instance. The specification is a document that formally describes what is required of a JVM implementation. Having a single specification ensures all implementations are interoperable. A JVM implementation is a computer program that meets the requirements of the JVM specification. An instance of a JVM is an implementation running in a process that executes a computer program compiled into Java bytecode.

>Java虚拟机（JVM）是使计算机运行Java程序的**抽象计算机器**。 JVM有三种概念：规范，实现和实例。 规范是一个正式描述JVM实现所需要的文档，有一个单一的规范确保所有的实现是可互操作的。 JVM实现是一个满足JVM规范要求的计算机程序。 JVM的一个实例是运行在执行编译为Java字节码的计算机程序的进程中的实现。

<!--more-->

说白了，jvm类似windows平台上的vmware或者VirtualBox等虚拟系统工具一样，他包含了一套完善的硬件架构体系，如处理器、堆栈、寄存器等，还具有相应的指令系统。
当我们的java代码经过了编译器生成字节码后，将在jvm内部执行，只要保证jvm内部环境一致，也就保证了java的跨平台性。

>JVM也是一个软件，不同的平台有不同的版本。jvm就是负责将字节码文件翻译成特定平台下的机器码然后运行。也就是说，只要在不同平台上安装对应的JVM，就可以运行字节码文件，运行我们编写的Java程序。

## JVM 运行过程

![](https://s1.ax1x.com/2017/11/17/cxl9O.png)
<center>(基于Java虚拟机规范Java SE 7版的Java虚拟机（JVM）体系结构概述)</center>

### Zero step : JAVA 命令 (基于hotspot虚拟机)

所有的java应用都需要使用java命令进行运行,而该命令的主要功能就是用于初始化和加载JVM。

简单的以linux为例，分析[openjdk的源码](http://download.java.net/openjdk/jdk8/)，可以发现jvm的启动过程： 
 
1. 应用入口`main()`方法,作为应用入口，主要实现了判断操作系统并执行不同的代码段  

2. 源码位于java.c的`JLI_Launch()`方法，其中包含了加载各类参数并实例化jvm虚拟机的功能  

3. 在`JLI_Launch()`方法中调用的`LoadJavaVM()`方法，主要是针对不同的操作系统，动态调用不同的接口，完成载入虚拟机动态链接库，并初始化 ifn 中的函数指针  

4. 在`JLI_Launch()`方法中调用的`JVMInit()`方法，这个函数会启动一个新的线程，并执行`JavaMain()`函数,需要注意的是，这个方法是动态加载的，所以会根据不同的操作系统加载不同的源文件，例如linux下的这个方法的源码位于：`jdk/src/solaris/bin/java_md_solinux.c`这个文件中  

5. `JavaMain()`函数同样位于java.c文件中，这个函数会初始化虚拟机，加载各种类，**并执行应用程序中的 main 函数**  

以上就是简单的jvm启动过程的源码分析，我们并不需要太过于深入了解其中代码的详细执行过程，大致了解一下即可,如果对详细的启动过程有兴趣的朋友，可以参考以下两篇文章:
> - [txazoc - JVM启动分析](http://www.txazo.com/jvm/jvm-startup.html)  
> - [韦庆明 - Java JVM 运行机制及基本原理](https://zhuanlan.zhihu.com/p/25713880)

### First step： Class Loader Subsystem 类加载器子系统

类加载器承担了装载java字节码(classes)的工作。Class Loader分为`Bootstrap ClassLoader`(启动类装载器)、`ExtClassLoader`(扩展类装载器)和`AppClassLoader`(应用类装载器)。

1. **启动类装载器**是Java类加载层次中最顶层的类加载器，负责加载JDK中的核心类库，如：rt.jar、resources.jar、charsets.jar等，同时，还会加载jre/classes/下面的class字节码文件。实际上我并没有在jdk源码中找到对应的实现代码，根据[网上的说法](https://stackoverflow.com/questions/18214174/how-is-the-java-bootstrap-classloader-loaded)，这个加载器并不是java类，而是**特定于平台的机器指令**，这就类似电脑系统的boot loader。令人不解的是我并没有在网上找到执行这个指令的具体实现地址，所以如果有网友能找到这个指令执行的地址，请告诉我一下.....

2. **扩展类装载器**之所以被称为扩展类装载器，是因为它负责加载java的拓展类库，默认加载$JAVA_HOME中jre/lib/目录下的jar包 或 -Djava.ext.dirs指定目录下的jar包。

3. **应用类装载器**这个加载器负责加载我们自己的应用程序classpath下面的所有jar包和class文件。

4. 除了java自己提供的三个classloader之外，用户可以根据自己需求自定义ClassLoader。

以上的类加载器，除了启动类装载器之外，其余的都必须继承java.lang.ClassLoader类。由于篇幅原因关于ClassLoader类的源码详解，我将在下一篇博文中详细介绍。

### Second Step : Runtime Data Area 运行数据区

运行时数据空间包括了5个部分：
  
1. **方法区（Method Area）**所有的类(class)级数据将被存储在这里，包括**静态变量**。方法区域是每个JVM一个，它是一个**共享资源**。

2. **Java堆(Heap Area)**所有对象及其相应的实例变量和数组将被存储在这里。堆区域也是每个JVM一个，因为方法区域和堆区域共享多个线程的内存，存储的数据**不是线程安全的**

3. **Java栈(Stack Area)**对于每个线程，将创建一个单独的运行时栈。对于每个方法调用，将在栈内存中创建一个名为Stack Frame的条目。所有本地变量将在栈内存中创建。**栈区域是线程安全的**，因为它不是共享资源。

4. **PC寄存器(PC Registers)**每个线程都有独立的PC寄存器，一旦执行指令就保存当前执行指令的地址，PC寄存器将用下一条指令更新

5. **本地方法栈(Native Method stacks)**本地方法栈包含本地方法信息。对于每个线程，都会创建单独的本地方法栈。栈内的数据在超出其作用域后，会被自动释放掉，**它不由JVM GC管理**。在hotspot虚拟机中，本地方法栈和JVM栈合并了。

作为jvm运行的核心之一的运行数据区，绝大部分功能都是基于内存操作完成的，而其实现又是基于C++的，对于我们来说只需要了解其大致功能就行了，没必要去详细解读其具体的实现原理。

### Third Step : JVM执行引擎

作为JVM的核心之一,JVM的执行引擎拥有类似物理机一样的功能。唯一不同的是物理机基于硬件、指令集和操作系统，所以会有很多的限制，而JVM执行引擎是自己实现的，所以程序员可以自行制定指令集和执行引擎的结构体系，因此能够执行那些不被硬件直接支持的指令集格式。JVM执行引擎的主要功能是：执行引擎读取字节码并逐个执行来自数据区的字节码。

在JVM标准中，为执行引擎定义了三个模块，分别是：`Interpreter(解释器)`、`JIT Compiler(编译器)`和`Garbage Collector(垃圾回收机制)`。

其中,`Interpreter(解释器)`和`JIT Compiler(编译器)`是代码的执行方式，有些虚拟机只采用其中一种，而有些虚拟机两种都会采用。不过，对于所有的JVM执行引擎的来说，**输入的是字节码文件、处理过程是等效字节码解析过程、输出的是执行结果**，这三点是一致的。

`Garbage Collector(垃圾回收机制)`是JAVA语言的一大优势之一，相对于C++等语言来说，GC的出现使得开发人员不用再去调试那些糟心的内存问题，对于不同的JVM来说它的GC引擎是衡量它是否优秀的重要因素之一。我将在以后的博文中专门开一篇来详细介绍GC。

### Other : Native Method Interface (JNI) 本地方法接口 

简单来说，JNI就是java与本地方法库，比如windows系统库等本地方法交互的接口。但是一旦使用了JNI接口，java程序就失去了JAVA平台的最大优势“跨平台”。所以，建议大家尽量减少使用JNI接口。  

## 最后 

关于JVM的一些基础的东西，已经给大家介绍完了，在接下来的博文中，我将对JVM中相对比较关键的功能进行深入了解和分析。

## 结束

>**本文部分文本来源于互联网**

>**感谢以下文章提供的灵感和帮助**  
> - [ Wikipedia - Java virtual machine](https://en.wikipedia.org/wiki/Java_virtual_machine)  
> - [JVM Architecture – Understanding JVM Internals](http://www.javainterviewpoint.com/java-virtual-machine-architecture-in-java/)  
> - [minixbeta - Java虚拟机的启动与程序的运行](https://minixbeta.github.io/%E6%8A%80%E6%9C%AF/2014/08/21/jvm-start-prog-run.html)  
> - [txazoc - JVM启动分析](http://www.txazo.com/jvm/jvm-startup.html)  
> - [韦庆明 - Java JVM 运行机制及基本原理](https://zhuanlan.zhihu.com/p/25713880)  
> - [深度分析Java的ClassLoader机制（源码级别）](http://www.hollischuang.com/archives/199)  
> - [xyang0917 - 深入分析Java ClassLoader原理](http://blog.csdn.net/xyang81/article/details/7292380)
