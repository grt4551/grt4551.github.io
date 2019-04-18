---
layout: post
title:  "从头开始分析JDK : 从'源头'开始"
date: 2017-11-9 18:32:48
tags: 
    - java
    - JDK
    - 源码分析
categories: 从头开始分析JDK
---

<blockquote class="blockquote-center">作为一个JAVA程序开发者,见过太多的技术太多的框架,却对原本JDK的内容知之甚少。为了弥补这个遗憾,我将重新审视分析JDK</blockquote>

# JDK里都有啥？

以jdk1.8为例，下图为jdk1.8的整体技术架构图：
![](https://s1.ax1x.com/2017/11/10/DCrSH.png)

<!--more-->

由上至下，分别是

- Java Language
>Java™编程语言是一种通用的，并发的，强类型的，基于类的面向对象语言。它通常编译为Java虚拟机规范中定义的字节码指令集和二进制格式。

- JDK Tools & Tool APIs （JDK工具和实用程序集合）
>JDK工具和实用程序集合，包含了基础工具集、安全工具集、国际化工具集、远程方法调用（RMI）工具集等多种实用工具，其中基础工具集包含了java应用启动器、java编译器javac等工具，是JDK的基础。

- Deployment （客户端程序部署工具）
>Java Web Start 是基于 Java 技术的应用程序的一种部署解决方案。Applet或Java小应用程序是一种在Web环境下，运行于客户端的Java程序组件。

- User Interface Toolkits（用户界面工具包）
>User Interface Toolkits包含了多种用户交互的接口

- Integration Libraries (外部应用集成库)
>包含JDBC（java数据库连接）、RMI（远程方法调用）、JNDI（Java命名和目录接口）等多种外部应用集成接口工具。

- Other Base Libraries(其他基础包)
>包含了java中的其他重要的功能，例如javaBean、I\O输入输出、Serialization序列化、网络功能等。

- lang and util Base Libraries（基础对象包）
>提供基本的Object和Class 类，基本类型的包装类，基本的数学类等等。

- Java Virtual Machine（java虚拟机）
>java运行的基础环境，java跨平台特性的实际实现者。

从下一篇博文开始，我将从java 虚拟机开始对jdk进行源码分析。

**ps.看着这么多的功能，着实让人头疼，但是其中很多功能或是已经被淘汰，或者是被很多第三方库的功能取代，所以讨论的价值并不大。**

**pps.在今年9月发布的java 9中实现了java 20年以来最重要的功能：模块化（Module），它把JDK中的一大坨玩意儿分离成了各种不同功能的模块，把原本几十上百M的运行环境一下就变得小巧精致了。如果可以的话，建议在尽早熟悉使用java 9。**

## 结束

>**本文部分文本来源于互联网**

>**感谢以下文章提供的灵感和帮助**  
>
> - [Java Platform Standard Edition 8 Documentation](https://docs.oracle.com/javase/8/docs/)  
