---
layout: post
title:  "spring-cloud 学习笔记(5)"
date: 2017-9-7 16:50:18
tags: 
    - java
    - spring-cloud
categories: spring-cloud
---

{:toc}

# STEP 5 : 分布式配置中心 - 进阶与整合

[上一节](https://maxith.github.io/2017/09/06/spring-cloud-study-5/)我们学习了分布式配置中心config server，由于篇幅有限，所以上一节内容比较基础，在实际生产过程中还需要很多功能的集成。这一节我们将学习分布式配置中心的进阶使用和其他功能的整合。

## 一. 安全整合

### 1. 使用spring security进行密码鉴权

##### 在配置中心服务中添加配置：

pom.xml引入`spring security`支持：

{% highlight xml %}  
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
{% endhighlight %}

在application.properties中添加用户名和密码：
{% highlight xml %}  
security.user.name=userName		//用户名默认是user
security.user.password=password		//默认使用随机密码写入到log中
{% endhighlight %}

<!--more-->

##### 客户端配置配置中心的密码：

在客户端的bootstrap.properties中添加配置：
{% highlight xml %}  
spring.cloud.config.username=userName
spring.cloud.config.password=password
{% endhighlight %}

可参考文章：
> - [Spring Cloud Config 中文文档参考手册中文版](https://springcloud.cc/spring-cloud-config.html)
> - [springcloud-config配置中心的安全配置](http://blog.csdn.net/json20080301/article/details/63685831)


### 2. 配置内容的加密、解密

由于配置文件放到git里面难免会发生配置文件中保存的密码泄露这种严重的问题，所以我们将对密码等敏感信息进行加密处理。

#### 安装JCE

>The Java Cryptography Extension (JCE) is an officially released Standard Extension to the Java Platform and part of Java Cryptography Architecture. JCE provides a framework and implementation for encryption, key generation and key agreement, and Message Authentication Code (MAC) algorithms. JCE supplements the Java platform, which already includes interfaces and implementations of message digests and digital signatures. Installation is specific to the version of the Java Platform being used, with downloads available for Java 6, Java 7, and Java 8.  

>Java加密扩展（JCE）是官方发布的Java平台标准扩展和Java加密体系结构的一部分。 JCE提供加密，密钥生成和密钥协商以及消息认证码（MAC）算法的框架和实现。 JCE补充了Java平台，它已经包括消息摘要和数字签名的接口和实现。 安装特定于正在使用的Java Platform版本，下载可用于Java 6，Java 7和Java 8。

**根据不同的jdk版本选择所需的jce**   
jce 8 的下载地址：[http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)

将下载好的文件解压覆盖到：`jre/lib/security`文件夹下.

#### 修改配置文件
修改配置文件，在application.properties中添加密钥()：
>encrypt.key=123123

#### 运行加密命令
然后使用curl命令调用接口就可以对数据进行加密了：
>curl http://127.0.0.1:1001/encrypt -d [需要加密的内容]

#### 保存加密后的数据
将加密后的字符串放到git上需要该数据的配置文件里，并且在加密后的字符串前面加上`{cipher}`字段，这样spring cloud就知道这个数据是加密保存的：
>password={cipher}e49cf250bafcaeb7b67c061d22a4698071d05c13ef34ad23422b87b4394d7269

#### 使用数据
由于这种加密方式只是在配置中心服务器与git之间产生，而客户端获取到的数据是已经解密后的数据，所以客户端不需要对接收到的数据进行处理。

#### 附言
ps：如果使用的是平台未安装curl插件，可以在这个地址下载：[https://curl.haxx.se/download.html](https://curl.haxx.se/download.html)

pps：spring cloud 版本bug：如果使用的spring cloud 版本为：**Dalston.SR2**或者**Dalston.SR3** 在运行加解密命令的时候会遇到如下bug：
>{  
  "description": "No key was installed for encryption service",  
  "status": "NO_KEY"  
}

该bug的地址：[https://github.com/spring-cloud/spring-cloud-config/issues/767](https://github.com/spring-cloud/spring-cloud-config/issues/767)

#### 参考文章
> - [AAorange : Spring Cloud Config 加密](http://www.jianshu.com/p/a8148da4a587)
> - [周立 : Config Server——配置内容的加密与解密](http://www.itmuch.com/spring-cloud/config-server-encrypt-decrypt/)
> - [huangyongxing310 : spring cloud config 加密使用](http://huangyongxing310.iteye.com/blog/2382002)

## 二. 实时更新  

**使用消息队列通知服务更新配置文件**

我们将使用**spring cloud bus**构建消息服务总线，由于篇幅有限，我们暂时只使用其中一些功能，具体使用过程我们将在以后的学习中深入了解。

消息服务器我使用的 **RabbitMQ** ，如果需要使用 **kafka** 作为消息服务器的话，可以参考这篇文章：[SpringCloud 分布式配置](http://blog.spring-cloud.io/blog/sc-config.html)。

### 1. config server 服务端修改

#### pom.xml中引入amqp支持
{% highlight xml %}  
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
{% endhighlight %}

#### 配置rabbitMQ
{% highlight xml %}  
spring.rabbitmq.host=192.168.1.80
spring.rabbitmq.port=5672
spring.rabbitmq.username=amqp
spring.rabbitmq.password=123456

//关闭权限认证
management.security.enabled=false
{% endhighlight %}

### 2. 客户端修改

**pom.xml中引入amqp支持**  
{% highlight xml %}  
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
{% endhighlight %}

**配置rabbitMQ**  
{% highlight xml %}  
spring.rabbitmq.host=192.168.1.80
spring.rabbitmq.port=5672
spring.rabbitmq.username=amqp
spring.rabbitmq.password=123456
{% endhighlight %}

**添加 `@RefreshScope` 注解**

需要给加载变量的类上面加载`@RefreshScope`，在执行更新操作的时候就会更新此类下面的变量值。

{% highlight java %}  
@RestController
@RefreshScope
public class ServerCallController {

    @Autowired
    private DiscoveryClient discoveryClient;

    @Autowired
    private RestTemplate restTemplate;

    @Value("${hello.world}")
    private String hello;

    @RequestMapping("/call/{serverName}")
    public String callServer(@PathVariable("serverName") String serverName) throws InterruptedException {
        List<ServiceInstance> serviceInstances = discoveryClient.getInstances(serverName);
        if (serviceInstances != null && serviceInstances.size() > 0){
            ServiceInstance instance = serviceInstances.get(0);
            return hello + " --- " +restTemplate.getForObject(instance.getUri().toString()+"/say",String.class);
        }
        return "service not found";
    }
}
{% endhighlight %}

### 3. 运行

将我们改造后的spring-cloud-eureka-server、spring-cloud-eureka-client和spring-cloud-server-consumer依次运行。

首先调用`http://127.0.0.1:2001/call/SPRING-CLOUD-EUREKA-CLIENT` 接口，返回如下结果：
>Hello world!We will rock you. --- [SPRING-CLOUD-EUREKA-CLIENT, SPRING-CLOUD-SERVER-CONSUMER]

然后修改git上面的配置，我们将 `app-dev.properties` 中的参数 `hello.world` 的值修改为： `I change the world.`。接着推送到git远程仓库。

接下来使用postman或者curl调用 `http://127.0.0.1:1001/bus/refresh` 接口，进行post请求，更新配置文件。

>**需要注意的是直接调用该接口会更新所有的配置，如果需要更新指定服务的配置需要添加参数**：`destination=spring-cloud-eureka-client:**`，其中**代表的是所有端口

最后我们再次调用`http://127.0.0.1:2001/call/SPRING-CLOUD-EUREKA-CLIENT` 接口，返回如下结果：
>I change the world. --- [SPRING-CLOUD-EUREKA-CLIENT, SPRING-CLOUD-SERVER-CONSUMER]

### 4. 遗留问题 

#### 1. 配置中心安全问题
如果需要使用refresh功能，需要将关闭配置中心的密码验证，即：
>management.security.enabled=false

这个导致了整个配置中心都不再需要进行密码验证，这个问题我查阅了很多资料，暂时没找到什么好的解决方案。

#### 2. 配置中心与客户端使用https链接

这个问题主要是客户端无法调用使用了https加密链接的配置中心，主要报错为：
>Certificate for <> doesn't match any of the subject alternative names: []

同样的问题在google上没有找到相同的例子，暂时无法解决。。

## 结束

>**本文部分文本来源于互联网**

>**感谢以下文章提供的灵感和帮助**  
> - [纯洁的微笑：springcloud(九)：配置中心和消息总线（配置中心终结版）](http://www.ityouknow.com/springcloud/2017/05/26/springcloud-config-eureka-bus.html)  
> - [LOC_Thomas : 配置spring-boot-actuator时候遇到的一些小问题](http://www.jianshu.com/p/b0b40038bb93)  
> - [Spring Cloud Config 中文文档参考手册中文版](https://springcloud.cc/spring-cloud-config.html)  
> - [天高 : 聊聊 Spring Cloud Config](https://blog.coding.net/blog/spring-cloud-config?utm_source=tuicool&utm_medium=referral)
