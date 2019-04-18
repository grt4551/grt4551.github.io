---
layout: post
title:  "spring-cloud 学习笔记(0)"
date: 2017-8-28 16:45:07
tags : 
    - java
    - spring-cloud
categories: spring-cloud
---

{:toc}

# STEP 0 : hello spring-cloud

## 1.什么是spring-cloud

官方介绍
>Spring Cloud provides tools for developers to quickly build 
some of the common patterns in distributed systems 
(e.g. configuration management, service discovery, 
circuit breakers, intelligent routing, micro-proxy, 
control bus, one-time tokens, global locks, leadership 
election, distributed sessions, cluster state). 
Coordination of distributed systems leads to boiler plate patterns, 
and using Spring Cloud developers can quickly stand up services and 
applications that implement those patterns. 
They will work well in any distributed environment, 
including the developer's own laptop, bare metal data centres, 
and managed platforms such as Cloud Foundry.

GOOGLE翻译: 
>Spring Cloud为开发人员提供快速构建的工具
分布式系统中的一些常见模式
（例如，配置管理，服务发现，
断路器，智能路由，微代理，
控制器，一次性令牌，全局锁，领导
选举，分布式会话，集群状态）。
分布式系统的引导样板模式，
并使用Spring Cloud开发人员可以快速建立服务
实现这些模式的应用程序。
他们将在任何分布式环境中运行良好，
包括开发商自己的笔记本电脑，裸机数据中心，
以及诸如Cloud Foundry之类的管理平台。

<!--more-->

网上大神的理解(大神的博客 : [纯洁的微笑 : springcloud(一)：大话Spring Cloud](http://www.ityouknow.com/springcloud/2017/05/01/simple-springcloud.html)):
>Spring Cloud是**一系列框架的有序集合**。它利用Spring Boot的开发便利性巧妙地**简化了分布式系统基础设施的开发**，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都可以用Spring Boot的开发风格做到一键启动和部署.Spring并没有重复制造轮子，它只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过Spring Boot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了**一套简单易懂、易部署和易维护的分布式系统开发工具包**。


#### 总的来说,spring cloud就是一堆用于构建分布式系统的工具的集合.

## 2.spring-cloud 和 spring-boot 的关系

要理解这两者的关系首先需要知道**什么是SPRING-BOOT**。

>随着功能以及业务逻辑的日益复杂，应用伴随着大量的XML配置文件以及复杂的Bean依赖关系。随着Spring 3.0的发布，Spring IO团队逐渐开始摆脱XML配置文件，并且在开发过程中大量使用“约定优先配置”（convention over configuration）的思想来摆脱Spring框架中各类繁复纷杂的配置（即时是Java Config）。

>Spring Boot正是在这样的一个背景下被抽象出来的**开发框架**，它**本身并不提供Spring框架的核心特性以及扩展功能**，只是用于快速、敏捷地开发新一代基于Spring框架的应用程序。也就是说，它并不是用来替代Spring的解决方案，而是和Spring框架紧密结合用于提升Spring开发者体验的工具。同时它集成了大量常用的第三方库配置（例如Jackson, JDBC, Mongo, Redis, Mail等等），Spring Boot应用中这些第三方库几乎可以零配置的开箱即用（out-of-the-box），大部分的Spring Boot应用都只需要非常少量的配置代码，开发者能够更加专注于业务逻辑。

#### spring - boot 的特点
> - 为所有Spring开发提供一个从根本上更快，且随处可得的入门体验。
> - 开箱即用，但通过不采用默认设置可以快速摆脱这种方式。
> - 提供一系列大型项目常用的非功能性特征，比如：内嵌服务器，安全，指标，健康检测，外部化配置。
> - 绝对没有代码生成，也不需要XML配置。

#### 那么,spring - boot 和 spring - cloud 是什么关系呢?

首先 spring-boot 是
> **开发框架** 且 **并不提供Spring框架的核心特性以及扩展功能**

而 spring-cloud 是
> **一系列框架的有序集合**

且
>Spring boot可以离开Spring Cloud独立使用开发项目，但是Spring Cloud离不开Spring boot，属于依赖的关系

所以两者的关系应该是:
> **spring > spring boot > spring cloud**


## 3.spring-cloud所包含的子项目

### 核心项目:
#### 1.Spring Cloud Config -- 配置中心
>Spring Cloud Config项目是一个解决分布式系统的配置管理方案。它包含了Client和Server两个部分。

#### 2.Spring Cloud Bus -- 事件、消息总线
>事件、消息总线，用于在集群（例如，配置变化事件）中传播状态变化，可与Spring Cloud Config联合实现热部署。

#### 3.Netflix家族

- **Eureka** (spring cloud中最重要的功能组件)
	> 服务中心，云端服务发现，一个基于 REST 的服务，用于定位服务，以实现云端中间层服务发现和故障转移
- **Hystrix**
	> 熔断器，容错管理工具，旨在通过熔断机制控制服务和第三方库的节点,从而对延迟和故障提供更强大的容错能力。
- **Zuul**
	> Zuul 是在云平台上提供动态路由,监控,弹性,安全等边缘服务的框架。
- **Archaius**
	> 配置管理API，包含一系列配置管理API，提供动态类型化属性、线程安全配置操作、轮询框架、回调机制等功能。
- **Ribbon**
	> 提供云端负载均衡，有多种负载均衡策略可供选择，可配合服务发现和断路器使用。

#### 4.Spring Cloud Consul -- 服务发现与配置工具

>封装了Consul操作，consul是一个服务发现与配置工具，与Docker容器可以无缝集成。

#### 5.Spring Cloud for Cloud Foundry -- 通过Oauth2协议绑定服务到CloudFoundry

>通过Oauth2协议绑定服务到CloudFoundry，CloudFoundry是VMware推出的开源PaaS云平台。

#### 6.Spring Cloud Sleuth -- 日志收集工具包

>日志收集工具包，封装了Dapper和log-based追踪以及Zipkin和HTrace操作，为SpringCloud应用实现了一种分布式追踪解决方案。

#### 7.其他工具
>Spring Cloud Security、Spring Cloud Sleuth、Spring Cloud Data Flow、Spring Cloud Stream、Spring Cloud Task、Spring Cloud Zookeeper、Spring Cloud Connectors、Spring Cloud Starters、Spring Cloud CLI 等等


## 结束

>**本文部分文本来源于互联网**

>**感谢以下文章提供的灵感和帮助** 

> - [纯洁的微笑 : springcloud(一)：大话Spring Cloud](http://www.ityouknow.com/springcloud/2017/05/01/simple-springcloud.html)
> - [翟永超 : Spring Cloud构建微服务架构（一）服务注册与发现](http://blog.didispace.com/springcloud1/) 
> - [Spring Boot——开发新一代Spring Java应用](https://www.tianmaying.com/tutorial/spring-boot-overview)