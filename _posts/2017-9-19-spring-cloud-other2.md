---
layout: post
title:  "spring-cloud 学习笔记 - 外传(2)"
date: 2017-9-19 11:27:23
tags: 
    - java
    - spring-cloud
categories: spring-cloud
---

{:toc}

## [外传] 关于服务发现框架Consul 、Zookeeper 、Eureka

作为分布式应用的基础，服务注册与发现是至关重要的，而Consul 、Zookeeper 、Eureka作为时下最火的三个框架自然也是需要拿出来比一比的。

三者的特点如下：
<table>
	<tr>
		<th></th>
		<th>Consul</th>
		<th>Zookeeper</th>
		<th>Eureka</th>
	</tr>
	<tr>
		<td>服务健康检查</td>
		<td>服务状态，内存，硬盘等</td>
		<td>(弱)长连接，keepalive	</td>
		<td>可配支持</td>
	</tr>
	<tr>
		<td>多数据中心</td>
		<td>支持</td>
		<td>-	</td>
		<td>-</td>
	</tr>
	<tr>
		<td>key-value存储服务</td>
		<td>支持</td>
		<td>支持	</td>
		<td>-</td>
	</tr>
	<tr>
		<td>一致性</td>
		<td>raft</td>
		<td>paxos</td>
		<td>-</td>
	</tr>
	<tr>
		<td>CAP</td>
		<td>ca</td>
		<td>cp</td>
		<td>ap</td>
	</tr>
	<tr>
		<td>使用接口(多语言能力)</td>
		<td>支持http和dns</td>
		<td>客户端	</td>
		<td>http（sidecar）</td>
	</tr>
	<tr>
		<td>watch支持</td>
		<td>全量/支持long polling</td>
		<td>支持</td>
		<td>http（sidecar）</td>
	</tr>
	<tr>
		<td>使用接口(多语言能力)</td>
		<td>支持http和dns</td>
		<td>支持</td>
		<td>支持 long polling/大部分增量</td>
	</tr>
	<tr>
		<td>自身监控</td>
		<td>metrics</td>
		<td>-</td>
		<td>metrics</td>
	</tr>
	<tr>
		<td>安全</td>
		<td>acl /https</td>
		<td>acl</td>
		<td>-</td>
	</tr>
	<tr>
		<td>spring cloud集成</td>
		<td>已支持</td>
		<td>已支持</td>
		<td>已支持</td>
	</tr>
</table>
<center>(表格数据源自：[服务发现比较:Consul vs Zookeeper vs Etcd vs Eureka](https://luyiisme.github.io/2017/04/22/spring-cloud-service-discovery-products/))</center>

<!--more-->

从上面的表格我们可以看出，这三个框架有自己不同的长处和短处，其中三者区别最大的是CAP属性：

> **CAP定理**:  

> 在理论计算机科学中，CAP定理（CAP theorem），又被称作布鲁尔定理（Brewer's theorem），它指出对于一个分布式计算系统来说，不可能同时满足以下三点： 
 
> - 一致性（Consistence) （等同于所有节点访问同一份最新的数据副本）  
> - 可用性（Availability）（每次请求都能获取到非错的响应——但是不保证获取的数据为最新数据）  
> - 分区容错性（Network partitioning）（以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。） 
 
> 根据定理，分布式系统只能满足三项中的两项而不可能满足全部三项。理解CAP理论的最简单方式是想象两个节点分处分区两侧。允许至少一个节点更新状态会导致数据不一致，即丧失了C性质。如果为了保证数据一致性，将分区一侧的节点设置为不可用，那么又丧失了A性质。除非两个节点可以互相通信，才能既保证C又保证A，这又会导致丧失P性质。

> -- [维基百科：CAP定理](https://zh.wikipedia.org/wiki/CAP%E5%AE%9A%E7%90%86)

**由此可见，三个框架的应用场景是不相同的。**

### Consul

> - Consul通过长轮询的方式来实现变化的感知
> - Consul是强一致性的数据存储，使用gossip形成动态集群。
> - Consul为多种数据中心提供了开箱即用的原生支持，其中的gossip系统不仅可以工作在同一集群内部的各个节点，而且还可以跨数据中心工作。

由于Consul是提供了多数据中心的高可用高一致性的解决方案，根据CAP定理，这种解决方案如果多个数据中心不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在一致性和高可用之间做出选择。

### Zookeeper

事实上我**并不推荐**使用Zookeeper作为服务发现框架，虽然Zookeeper是著名Hadoop的一个子项目，旨在解决大规模分布式应用场景下，服务协调同步（Coordinate Service）的问题。

为什么呢？

由上面图表我们可以看见Zookeeper的CAP属性是CP，这意味着：

>&nbsp;&nbsp;&nbsp;&nbsp;**任何时刻对ZooKeeper的访问请求能得到一致的数据结果，同时系统对网络分割具备容错性；但是它不能保证每次服务请求的可用性（注：也就是在极端环境下，ZooKeeper可能会丢弃一些请求，消费者程序需要重新请求才能获得结果）**。  
>&nbsp;&nbsp;&nbsp;&nbsp;但是别忘了，ZooKeeper是分布式协调服务，它的职责是保证数据（注：配置数据，状态数据）在其管辖下的所有服务之间保持同步、一致；所以就不难理解为什么ZooKeeper被设计成CP而不是AP特性的了。  
>
>\----[Knewton : 为什么不应该使用ZooKeeper做服务发现](http://dockone.io/article/78)

所以说，Zookeeper从设计的时候就没有考虑过高可用，作为一个分布式协同服务，ZooKeeper非常好，但是对于服务发现服务来说就不合适了；因为对于服务发现服务来说就算是返回了包含不实的信息的结果也比什么都不返回要好。

### Eureka

相比其他两个框架，Eureka则是舍去了对于分布式服务发现较为不那么敏感的一个方面，就是数据一致性。

> - Eureka处理网络问题导致分区。如果Eureka服务节点在短时间里丢失了大量的心跳连接（注：可能发生了网络故障），那么这个Eureka节点会进入“自我保护模式”，同时保留那些“心跳死亡”的服务注册信息不过期。此时，这个Eureka节点对于新的服务还能提供注册服务，对于“死亡”的仍然保留，以防还有客户端向其发起请求。当网络故障恢复后，这个Eureka节点会退出“自我保护模式”。**Eureka的哲学是，同时保留“好数据”与“坏数据”总比丢掉任何“好数据”要更好，所以这种模式在实践中非常有效**。 
> 
> - Eureka就是为发现服务所设计的，它有独立的客户端程序库，同时提供心跳服务、服务健康监测、自动发布服务与自动刷新缓存的功能。但是，如果使用ZooKeeper你必须自己来实现这些功能。  
> 
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\----&nbsp;[xixicat : 服务发现方案梳理及NetflixEureka简介](https://segmentfault.com/a/1190000004944218)

对于Eureka来说，它的专注并不是判断服务的“好坏”，它专注的是能否拿到用户所需要的服务，这一点在分布式服务发现中是最重要的。

## 结束

>**本文部分文本来源于互联网**

>**感谢以下文章提供的灵感和帮助**  
>
> - [服务发现比较:Consul vs Zookeeper vs Etcd vs Eureka](https://luyiisme.github.io/2017/04/22/spring-cloud-service-discovery-products/)  
> - [xixicat : 服务发现方案梳理及NetflixEureka简介](https://segmentfault.com/a/1190000004944218)  
> - [Knewton : 为什么不应该使用ZooKeeper做服务发现](http://dockone.io/article/78)  
> - [Sunddenly(⊙_⊙) : ZooKeeper和CAP理论及一致性原则](http://www.cnblogs.com/sunddenly/articles/4072987.html)