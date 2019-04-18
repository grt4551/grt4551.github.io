---
layout: post
title:  "spring-cloud 学习笔记 - 外传(3)"
date: 2017-9-21 14:51:41
tags: 
    - java
    - spring-cloud
categories: spring-cloud
---

{:toc}

# [外传] 关于 Ribbon 和 Feign 

&emsp;&emsp;在微服务消费的学习过程中，我并没有像别的博主使用Feign和Ribbon。因为这两个东西如果直接放到博文中可能会导致读者的混乱，搞不清楚这两者的关系，所以我单独写一篇来介绍这两者的关系。

## 一. 关于Ribbon 

### 什么是 Ribbon

>Spring Cloud Netflix Ribbon 是由 Netflix 开源的客户端负载均衡的组件。使用 **java** 语言实现。

说白了，ribbon像是nginx中的负载均衡组件一样，用来实现客户端的负载均衡。

<!--more-->

### 为什么使用 Ribbon 

> Ribbon作为**后端负载均衡器**，比Nginx更注重的是**承担并发**而不是请求分发，可以直接感知后台动态变化来指定分发策略。  
> <p style="text-align:right">---&emsp;<a href="http://blog.csdn.net/rickiyeat/article/details/64918756">Lovnx : Ribbon负载均衡策略配置</a></p>

从上面这段话我们可以看出，和nginx的定位不一样，nginx是**被动分发请求**，而Ribbon是**主动选择不同的服务提供者**，Ribbon的服务提供者又来至于服务注册中心eureka或者consul等。

### Ribbon有什么特点 

> - 和Eureka完美整合  
> - 支持多种协议-HTTP,TCP,UDP  
> - caching/batching  
> - built in failure resiliency  
> <p style="text-align:right">---&emsp;<a href="https://segmentfault.com/a/1190000006162832">木木彬 : Spring Cloud实战(三)-Spring Cloud Netflix Ribbon</a></p>

说白了，Ribbon的最大的优势就是背靠Spring Cloud Netflix这座大山，作为Netflix全家桶中的一员，Ribbon拥有天生的整合优势。

### 如何使用 Ribbon 

如果使用的是`RestTemplate`框架的话，只需要在RestTemplate初始化的方法上面加一个`@LoadBalanced`就可以了。  
{% highlight java linenos %}
@Bean
@LoadBalanced
RestTemplate restTemplate() {
    return new RestTemplate();
}
{% endhighlight %}

### 小坑注意！

详情请参考我的另一篇博文：[spring-cloud ribbon 和 restTemplate 的小坑](http://maxith.com/2017/09/21/spring-cloud-error/)

## 二. 关于Feign

### 什么是Feign

>Feign是一个声明式Web Service客户端。使用Feign能让编写`Web Service`客户端更加简单, 它的使用方法是定义一个接口，然后在上面添加注解，同时也支持`JAX-RS`标准的注解。Feign也支持可拔插式的编码器和解码器。Spring Cloud对Feign进行了封装，使其支持了Spring MVC标准注解和`HttpMessageConverters`。Feign可以与Eureka和Ribbon组合使用以支持负载均衡。  
><p align="right">---&emsp;<a href="http://spring-cloud.io/reference/feign/#feignhystrix">spring-cloud官方手册中文翻译 : 声明式REST客户端：Feign</a></p>

### 如何使用

首先在pom.xml中引入资源：
{% highlight xml linenos %}
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
{% endhighlight %}

>使用`spring-cloud-starter-feign`会默认引入ribbon的包，也就是说feign是默认集成了ribbon的

修改`Application`主类,添加`@EnableFeignClients`注解，以启用Feign:  
{% highlight java linenos %}
@SpringBootApplication
@EnableAutoConfiguration
@EnableDiscoveryClient
@EnableFeignClients
public class SpringCloudServerConsumerApplication {
	public static void main(String[] args) {
		SpringApplication.run(SpringCloudServerConsumerApplication.class, args);
	}
}
{% endhighlight %}

创建远程调用接口类`HelloClient.java`：
{% highlight java linenos %}  
@FeignClient(name = "SPRING-CLOUD-EUREKA-CLIENT")
public interface HelloClient {

    @RequestMapping(method = RequestMethod.POST,value = "/say",consumes = "application/json")
    String say();
}
{% endhighlight %}

>在@FeignClient中使用一个字符串（例子中的"SPRING-CLOUD-EUREKA-CLIENT"）来指定客户端名字,这个名字也将被用于一个Ribbon负载均衡器。 还可以配置url属性来指定URL。 在Spring容器中会使用全限定名作为这个Bean的名称。同样，可以通过name属性进行自定义。 在上例中，通过@Qualifier("helloClient")就可以引用到这个Bean。 如果需要改变默认引用名称，可以在@FeignClient中配置qualifier属性。

使用接口调用实现：  
{% highlight java linenos %}   
@RestController
public class ServerCallController {

    @Autowired
    private HelloClient helloClient;

    @RequestMapping("/callServer")
    public String callServer(){
        return helloClient.say();
    }
}
{% endhighlight %}

### 参考文章  

上述的Feign使用是非常基础的实现，如果需要实现更多的内容，可以参考以下文章：

> - [声明式REST客户端：Feign](http://spring-cloud.io/reference/feign/#feignhystrix)  
> - [Lin_XJ : Spring Cloud 微服务(六) 服务消费Feign](http://www.jianshu.com/p/07303bc4b015)  
> - [秋雨霏霏 : [Spring Cloud] 4.6 Declarative REST Client：Feign](https://my.oschina.net/roccn/blog/803731)  
> - [铁汤 : Feign正确的使用姿势和性能优化注意事项](http://www.jianshu.com/p/191d45210d16)

## 结束

>**本文部分文本来源于互联网**

>**感谢以下文章提供的灵感和帮助**  
>
> - [杨波 : 实施微服务，我们需要哪些基础框架？](http://www.infoq.com/cn/articles/basis-frameworkto-implement-micro-service)  
> - [木木彬 : Spring Cloud实战(三)-Spring Cloud Netflix Ribbon](https://segmentfault.com/a/1190000006162832)