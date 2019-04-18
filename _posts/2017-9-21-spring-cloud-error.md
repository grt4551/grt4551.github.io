---
layout: post
title:  "spring-cloud ribbon 和 restTemplate 的小坑"
date: 2017-9-21 11:50:47
tags : 
    - java
    - spring-cloud
categories: spring-cloud
---

今天在学习使用`Ribbon`和`RestTemplate`集成的时候遇到一个小坑:

使用`@LoadBalanced`注解后的`RestTemplate`运行报错：
{% highlight java %}
java.lang.IllegalStateException: No instances available for Maxith
	at org.springframework.cloud.netflix.ribbon.RibbonLoadBalancerClient.execute(RibbonLoadBalancerClient.java:90) ~[spring-cloud-netflix-core-1.3.4.RELEASE.jar:1.3.4.RELEASE]
	at org.springframework.cloud.client.loadbalancer.RetryLoadBalancerInterceptor$1.doWithRetry(RetryLoadBalancerInterceptor.java:103) ~[spring-cloud-commons-1.2.3.RELEASE.jar:1.2.3.RELEASE]
	at org.springframework.cloud.client.loadbalancer.RetryLoadBalancerInterceptor$1.doWithRetry(RetryLoadBalancerInterceptor.java:91) ~[spring-cloud-commons-1.2.3.RELEASE.jar:1.2.3.RELEASE]
	at org.springframework.retry.support.RetryTemplate.doExecute(RetryTemplate.java:287) ~[spring-retry-1.2.1.RELEASE.jar:na]
	at org.springframework.retry.support.RetryTemplate.execute(RetryTemplate.java:164) ~[spring-retry-1.2.1.RELEASE.jar:na]
	at org.springframework.cloud.client.loadbalancer.RetryLoadBalancerInterceptor.intercept(RetryLoadBalancerInterceptor.java:91) ~[spring-cloud-commons-1.2.3.RELEASE.jar:1.2.3.RELEASE]
	at org.springframework.http.client.InterceptingClientHttpRequest$InterceptingRequestExecution.execute(InterceptingClientHttpRequest.java:86) ~[spring-web-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.cloud.netflix.metrics.MetricsClientHttpRequestInterceptor.intercept(MetricsClientHttpRequestInterceptor.java:64) ~[spring-cloud-netflix-core-1.3.4.RELEASE.jar:1.3.4.RELEASE]
	at org.springframework.http.client.InterceptingClientHttpRequest$InterceptingRequestExecution.execute(InterceptingClientHttpRequest.java:86) ~[spring-web-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.http.client.InterceptingClientHttpRequest.executeInternal(InterceptingClientHttpRequest.java:70) ~[spring-web-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.http.client.AbstractBufferingClientHttpRequest.executeInternal(AbstractBufferingClientHttpRequest.java:48) ~[spring-web-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.http.client.AbstractClientHttpRequest.execute(AbstractClientHttpRequest.java:53) ~[spring-web-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.web.client.RestTemplate.doExecute(RestTemplate.java:652) ~[spring-web-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.web.client.RestTemplate.execute(RestTemplate.java:613) ~[spring-web-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.web.client.RestTemplate.getForObject(RestTemplate.java:287) ~[spring-web-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.web.client.RestTemplate$$FastClassBySpringCGLIB$$aa4e9ed0.invoke(<generated>) ~[spring-web-4.3.10.RELEASE.jar:4.3.10.RELEASE]
...
{% endhighlight %}

如果将`@LoadBalanced`注解去掉，则返回正确的结果，由于初学spring cloud，所以不是很明白为什么会出现这个问题，于是在网上找了半天，终于在stackoverflow找到了答案。

>The `RestTemplate` you autowired is already connected to Ribbon. So you do a lookup by hand and then `RestTemplate` is trying to lookup the hostname passed in to ribbon. You have two options: 1) Don't use the netflix `DiscoveryClient` and pass the serviceId as a logical hostname to ribbon (`http://TEST/myservice`), 2) Don't use the autowired `RestTemplate`, create a new one for your class. My choice would be #1.  
> <p align="right"> ---- spencergibb</p>

虽然这为朋友说的是对的，但是不是很好理解，其实说起来原因非常简单：

**去掉`DiscoveryClient` ! 因为之前我们需要`DiscoveryClient`来获取一个服务的实例并执行，但是加入了`@LoadBalanced`注解过后就不需要寻找实例了，`Ribbon`会主动帮我们寻找适合的实例并执行，所以我们只需要传递一个服务地址给`Ribbon`就行了。**

{% highlight java %}
@RequestMapping("/call/{serverName}")
public String callServer(@PathVariable("serverName") String serverName) throws InterruptedException {
    return restTemplate.getForObject("http://" +serverName+"/say",String.class);
}
{% endhighlight %}

问题链接：[Ribbon with Spring Cloud and Eureka: java.lang.IllegalStateException: No instances available for Samarths-MacBook-Pro.local](https://stackoverflow.com/questions/31574131/ribbon-with-spring-cloud-and-eureka-java-lang-illegalstateexception-no-instanc)