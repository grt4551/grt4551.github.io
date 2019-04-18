---
layout: post
title:  "注解@EnableDiscoveryClient，@EnableEurekaClient的区别[转]"
date: 2017-8-30 16:24:22
tags : 
    - java
    - spring-cloud
categories: spring-cloud
---

## 注解@EnableDiscoveryClient，@EnableEurekaClient的区别[转]

SpringCloud中的`Discovery Service`有多种实现，比如：eureka, consul, zookeeper。

{% highlight xml %}
1.`@EnableDiscoveryClient`注解是基于`spring-cloud-commons`依赖，并且在classpath中实现；
2，`@EnableEurekaClient`注解是基于`spring-cloud-netflix`依赖，只能为eureka作用；
{% endhighlight %}

如果你的classpath中添加了eureka，则它们的作用是一样的。

#### 来源:
[夜闯秋名山吃瓜皮 : 注解@EnableDiscoveryClient，@EnableEurekaClient的区别](http://blog.csdn.net/ezreal_king/article/details/72594535)