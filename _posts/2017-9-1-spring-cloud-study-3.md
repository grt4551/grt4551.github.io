---
layout: post
title:  "spring-cloud 学习笔记(2)"
date: 2017-9-1 10:24:22
tags: 
    - java
    - spring-cloud
categories: spring-cloud
---

{:toc}

# STEP 2 : 微服务的消费

在上一个笔记中我们学习了服务的注册与发现,那么已经有了服务提供方,接下来会有服务的消费方,这次笔记的主要内容就是 如何消费微服务.

## 基本配置

### 1.pom 设置
{% highlight xml %}  
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka</artifactId>
	</dependency>
</dependencies>
{% endhighlight %}

**pom配置需要注意的一个问题:**

>调用的客户端框架需要和服务端所使用的框架一致,比如说我们使用eureka框架作为服务端,那么客户端也需要使用eureka框架.

<!--more-->

### 2.配置文件 

application.properties配置如下:

{% highlight xml %}  
spring.application.name=spring-cloud-consumer  
server.port=2001  
eureka.client.serviceUrl.defaultZone=http://127.0.0.1:9999/eureka/  
{% endhighlight %}

spring-boot项目配置如下:

{% highlight java %}
@SpringBootApplication
@EnableDiscoveryClient			//启用服务注册与发现
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

	//spring 提供的更加简洁的url访问框架
    @Bean
    public RestTemplate restTemplate(@Qualifier("simpleClientHttpRequestFactory") ClientHttpRequestFactory factory){
        return new RestTemplate(factory);
    }
}
{% endhighlight %}

## 具体业务代码  

下面这段代码实现了调用指定微服务,并返回结果的功能.

{% highlight java %}
@RestController
public class ServerCallController {

    @Autowired
    private DiscoveryClient discoveryClient;

    @Autowired
    RestTemplate restTemplate;
	
    @RequestMapping("/call/{serverName}")
    public String callServer(@PathVariable("serverName") String serverName){
        List<ServiceInstance> serviceInstances = discoveryClient.getInstances(serverName);
        if (serviceInstances != null && serviceInstances.size() > 0){
            ServiceInstance instance = serviceInstances.get(0);
            return restTemplate.getForObject(instance.getUri().toString()+"/say",String.class);
        }
        return "service not found";
    }
}
{% endhighlight %}

## 运行

配置完成后,运行spring-cloud-eureka-consumer工程,访问 `http://127.0.0.1:9999/`,可以在注册中心的已注册服务列表中发现我们绑定的微服务.

![](/assets/images/post-images/2017-9-1-spring-cloud-study-3/1.png)

然后访问`http://127.0.0.1:2001/call/spring-cloud-eureka-client`就可以看见输出的结果了:

>[SPRING-CLOUD-EUREKA-CLIENT, SPRING-CLOUD-SERVER-CONSUMER]

微服务`SPRING-CLOUD-SERVER-CONSUMER`是我们刚才实现的微服务消费者,而`SPRING-CLOUD-EUREKA-CLIENT`是微服务提供者,我们可以发现,在spring-cloud中,将消费者也作为一个微服务提供者进行了注册.

如果不想将服务消费者写入注册中心,那么需要在`application.properties`中添加一个配置:

>eureka.client.register-with-eureka = false

有了这个配置,服务消费者将不会出现在注册中心的列表中,但是一样可以对服务进行消费.

## 结束

>**本文部分文本来源于互联网**

>**感谢以下文章提供的灵感和帮助**  
> - [纯洁的微笑 : springcloud(三)：服务提供与调用](http://www.ityouknow.com/springcloud/2017/05/12/eureka-provider-constomer.html)  
> - [翟永超 : Spring Cloud构建微服务架构：服务消费（基础）【Dalston版】](http://blog.didispace.com/spring-cloud-starter-dalston-2-1/)  
> - [lzhou666 : 综合使用spring cloud技术实现微服务应用](http://www.cnblogs.com/skyblog/p/5133796.html)