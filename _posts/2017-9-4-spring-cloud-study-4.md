---
layout: post
title:  "spring-cloud 学习笔记(3)"
date: 2017-9-4 10:57:51
tags: 
    - java
    - spring-cloud
categories: spring-cloud
---

{:toc}

# STEP 3 : 熔断器 

## 1. 什么是熔断器

### 熔断器(CircuitBreaker)
>**"熔断器(CircuitBreaker)"**本身是一种开关装置，用于在电路上保护线路过载，当线路中有电器发生短路时，**“熔断器”**能够及时的切断故障电路，防止发生过载、发热、甚至起火等严重后果。

在现代分布式应用的日常的生产过程中，我们的应用服务总是会出现各种各样的问题比如网络连接缓慢、资源繁忙，暂时不可用，服务脱机等导致服务稳定受到影响，一旦其中一个关联了子服务的服务受到影响后，子服务也会受到影响，然后一级一级的服务就会接连崩溃，最后导致整个系统的**雪崩**.这样的情况，我相信没有人希望出现。

而**熔断器模式**解决了大型分布式系统因为服务稳定性异常导致系统雪崩的问题，它能在服务阻塞(BLOCK)的时候通过断路器的故障监控（类似熔断保险丝)，向调用方返回一个错误响应，而不是长时间的等待，防止服务雪崩的发生。

<!--more-->

### Netflix Hystrix

在spring cloud 中提供了`Hystrix`来实现熔断器模式。

>Hystrix is a latency and fault tolerance library designed to isolate points of access to remote systems, services and 3rd party libraries, stop cascading failure and enable resilience in complex distributed systems where failure is inevitable.

>Hystrix是一个延迟和容错库，旨在隔离对远程系统，服务和第三方库的访问点，停止级联故障，并在复杂的分布式系统中启用恢复能力，故障是不可避免的。

## 2. 使用Hystrix的@HystrixCommand注解 

由于原生Hystrix的command模式需要设置较多的配置,比较麻烦,所以此处我们使用由Netflix开源社区提供的netflix contrib库 `javanica` 进行注解支持.

如果需要使用原生的Hystrix的command模式可以参考下面这几篇文章:  
> - [ybak : 防雪崩利器：熔断器 Hystrix 的原理与使用](https://segmentfault.com/a/1190000005988895)
> - [shangjiuliu : Hystrix使用](https://segmentfault.com/a/1190000011003059)
> - [star24 : Hystrix使用入门手册（中文）](http://www.jianshu.com/p/b9af028efebb)
> - [hot66hot : Hystrix 使用与分析](http://hot66hot.iteye.com/blog/2155036)

由于篇幅有限,该demo在前三章已经搭建好的服务消费者 `spring-cloud-server-consumer` 基础上进行修改,添加Hystrix.

#### 1. 在pom文件中添加依赖文件 
{% highlight xml %}  
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
{% endhighlight %}

#### 2. 开启熔断器

在spring-cloud中开启熔断器非常简单,只需要在Application类加入注解`@EnableCircuitBreaker`,这个注解由spring-cloud框架提供,并不是来源于**Hystrix**,如果需要指定使用Hystrix的话,也可以使用`@EnableHystrix`注解.

{% highlight java %}  
@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreaker		//开启熔断器的注解
public class SpringCloudServerConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCloudServerConsumerApplication.class, args);
	}

    @Bean
    public SimpleClientHttpRequestFactory simpleClientHttpRequestFactory(){
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        factory.setReadTimeout(5000); //ms
        factory.setConnectTimeout(15000);//ms
        return factory;
    }

    @Bean
    public RestTemplate restTemplate(@Qualifier("simpleClientHttpRequestFactory") ClientHttpRequestFactory factory){
        return new RestTemplate(factory);
    }
}
{% endhighlight %}

#### 3. 在需要熔断的方法上添加注解 `@HystrixCommand`

{% highlight java %}  
@RestController
public class ServerCallController {

    @Autowired
    private DiscoveryClient discoveryClient;

    @Autowired
    RestTemplate restTemplate;

    @RequestMapping("/call/{serverName}")
    @HystrixCommand(fallbackMethod = "callServerFallBack",commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "1000")
    })
    public String callServer(@PathVariable("serverName") String serverName) throws InterruptedException {
        //设置随机睡眠时间,模仿系统延迟
        Thread.sleep(new Random(System.currentTimeMillis()).nextInt(2000));

        List<ServiceInstance> serviceInstances = discoveryClient.getInstances(serverName);
        if (serviceInstances != null && serviceInstances.size() > 0){
            ServiceInstance instance = serviceInstances.get(0);
            return restTemplate.getForObject(instance.getUri().toString()+"/say",String.class);
        }
        return "service not found";
    }

    public String callServerFallBack(String serverName){
        return "service " + serverName + " error ";
    }
}  
{% endhighlight %}

>注解`@HystrixCommand`的参数`fallbackMethod`表示如果这个方法调用失败，会切换到备用方法findOrderFallback;参数`commandProperties`是一组命令参数的集合,我们这里设置了执行的超时时间为1秒钟,这意味着如果该服务的响应时间超过1秒钟就会切换到备用方法上.

#### 4. 运行结果

启动服务后访问`http://127.0.0.1:2001/call/SPRING-CLOUD-EUREKA-CLIENT`这个地址就能看到结果,在随机值的影响下,连续访问会返回不同的结果 `[SPRING-CLOUD-EUREKA-CLIENT]` 和 `service SPRING-CLOUD-EUREKA-CLIENT error`.

## 结束  

>**本文部分文本来源于互联网**

>**感谢以下文章提供的灵感和帮助**  
> - [菩提树下的杨过.Net : spring cloud 学习(4) - hystrix 服务熔断处理](http://www.cnblogs.com/yjmyzz/p/spring-cloud-hystrix-tutorial.html)  
> - [翟永超 : Spring Cloud构建微服务架构（三）断路器](http://blog.didispace.com/springcloud3/)  
> - [skaka : 微服务框架Spring Cloud介绍 Part5: 在微服务系统中使用Hystrix, Hystrix Dashboard与Turbine](http://skaka.me/blog/2016/09/04/springcloud5/)