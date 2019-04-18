---
layout: post
title:  "spring-cloud 学习笔记(6)"
date: 2017-9-20 17:00:26
tags: 
    - java
    - spring-cloud
categories: spring-cloud
---

{:toc}

# STEP 6 : API Gateway 之 Netflix Zuul

## 什么是API Gateway

>API Gateway是一个服务器，也可以说是进入系统的唯一节点。这跟面向对象设计模式中的Facade模式很像。API Gateway封装内部系统的架构，并且提供API给各个客户端。它还可能有其他功能，如授权、监控、负载均衡、缓存、请求分片和管理、静态响应处理等。

>API Gateway负责请求转发、合成和协议转换。所有来自客户端的请求都要先经过API Gateway，然后路由这些请求到对应的微服务。API Gateway将经常通过调用多个微服务来处理一个请求以及聚合多个服务的结果。它可以在web协议与内部使用的非Web友好型协议间进行转换，如HTTP协议、WebSocket协议。

>API Gateway可以提供给客户端一个定制化的API。它暴露一个粗粒度API给移动客户端。以产品最终页这个使用场景为例。API Gateway提供一个服务提供点（/productdetails?productid=xxx）使得移动客户端可以在一个请求中检索到产品最终页的全部数据。API Gateway通过调用多个服务来处理这一个请求并返回结果，涉及产品信息、推荐、评论等。

简单的来说，API Gateway功能有点儿像nginx，它是整个系统服务的总入口（即网关），它可以将请求分发到不同的微服务上。
> - 相比nginx，API Gateway是一个简化的轻量级的网关，它简化了内部服务相互调用的复杂度。  
> - 相比nginx，API Gateway能动态修改服务的地址，以及服务的启用和关闭，而不需要复杂的配置文件。

<!--more-->

## Zuul  

作为Netflix全家桶之一，zuul承担了API Gateway的功能，它拥有以下功能：

> - 认证鉴权  
> - 审查  
> - 压力测试  
> - 金丝雀测试  
> - 动态路由  
> - 服务迁移  
> - 负载剪裁  
> - 安全  
> - 静态应答处理  

同时，Zuul允许使用任何支持JVM的语言来建立规则和过滤器，并且内置了对Java和Groovy的支持。

### 简单使用

我们将使用前面几节搭建的微服务框架进行改造，加入简单的zuul服务路由。

#### 1. 搭建项目

新建spring boot项目，如果使用spring initializr，请勾选zuul，如果直接创建，需要引入依赖：  
{% highlight xml %}  
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>  
<dependency>  
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>  
{% endhighlight %}  

**引入eureka的目的是为了后续与eureka做整合，使用serviceId做路由。**

修改`SpringCloudZuulServerApplication` 启动类,添加`@EnableZuulProxy`注解：  
{% highlight java linenos %}  
@SpringBootApplication  
@EnableZuulProxy  
public class SpringCloudZuulServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCloudZuulServerApplication.class, args);
	}
}  
{% endhighlight %}

修改配置文件`application.properties`:
{% highlight java linenos %}
spring.application.name=spring-cloud-zuul-server
server.port=3001

eureka.client.service-url.defaultZone=http://192.168.1.110:9999/eureka/
{% endhighlight %}

#### 2.运行项目  

分别启动项目：
> - spring-cloud-eureka-server 服务发现服务端  
> - spring-cloud-eureka-client 提供远程服务调用接口
> - spring-cloud-server-consumer 服务消费者，使用Feign调用远程服务，并提供服务callServer返回远程调用结果
> - spring-cloud-zuul-server zuul路由服务  

zuul的默认规则：
>默认情况下，即无任何zuul配置的时，zuul会默认代理所有注册到Eureka Server的微服务，并且Zuul的路由规则如下：`http://ZUUL_HOST:ZUUL_PORT/微服务在Eureka上的serviceId/**`会被转发到serviceId对应的微服务。

调用spring-cloud-server-consumer的接口`http://192.168.1.110:2001/callServer`，返回结果：
>[SPRING-CLOUD-EUREKA-CLIENT, SPRING-CLOUD-SERVER-CONSUMER, SPRING-CLOUD-ZUUL-SERVER]

调用spring-cloud-zuul-server的接口`http://192.168.1.110:3001/spring-cloud-server-consumer/callServer`，返回结果：
>[SPRING-CLOUD-EUREKA-CLIENT, SPRING-CLOUD-SERVER-CONSUMER, SPRING-CLOUD-ZUUL-SERVER]

可以看见，两种调用方法结果一致，说明zuul正确的对我们的服务进行的分发。

### 进阶使用  

#### 1. 忽略、指定路由

忽略指定服务,使用`，`号隔开，支持正则匹配，直接填写`*`则全部忽略：  
{% highlight java%}  
zuul.ignored-services=spring-cloud-eureka-client  
{% endhighlight %}  

指定路由,规则是 `zuul.routes.服务名称=指定的服务地址` ，被指定的服务如果存在于忽略服务列表中，将**不会被忽略**：
{% highlight java%}
zuul.routes.spring-cloud-server-consumer=/server/**
{% endhighlight %}

#### 2. 动态路由配置

使用动态配置加载保存在git或其他配置文件中的配置信息，参考文章：

> [scipio : zuul动态路由加载配置](http://www.jianshu.com/p/a9332b111002)

## 结束

>**本文部分文本来源于互联网**

>**感谢以下文章提供的灵感和帮助**  
>
> - [2.6 API Gateway](http://book.itmuch.com/2%20Spring%20Cloud/2.6%20API%20Gateway.html)  
> - [纯洁的微笑 : springcloud(十)：服务网关zuul初级篇](http://www.ityouknow.com/springcloud/2017/06/01/gateway-service-zuul.html)  
> - [Software_King : Spring Cloud Zuul](http://xujin.org/sc/docs/sc-zuul/)
> - [Software_King : API GateWay(网关)那些儿事](http://xujin.org/sc/sc-zuul/)
> - [Spring Cloud技术分析（4）- spring cloud zuul](http://tech.lede.com/2017/05/16/rd/server/SpringCloudZuul/)