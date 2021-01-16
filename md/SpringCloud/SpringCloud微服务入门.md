# Spring Cloud 微服务入门

Spring Cloud 集成相关优质项目

## Spring Cloud Config

Spring 配置管理工具包，让你可以把配置放到远程服务器，集中化管理集群配置，目前支持本地存储、Git以及Subversion。

------

## Spring Cloud Bus

Spring 事件、消息总线，用于在集群（例如，配置变化事件）中传播状态变化，可与Spring Cloud Config联合实现热部署。

------

## [Eureka](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2FNetflix%2Feureka)

Netflix 云端服务发现，一个基于 REST 的服务，用于定位服务，以实现云端中间层服务发现和故障转移。

>    Eureka Server会提供服务注册服务，各个服务节点启动后，会在Eureka Server中进行注册，这样Eureka Server中就有了所有服务节点的信息，并且Eureka有监控页面，可以在页面中直观的看到所有注册的服务的情况。 Eureka有心跳机制，当某个节点服务在规定时间内没有发送心跳信号时，Eureka会从服务注册表中把这个服务节点移除。Eureka还提供了客户端缓存的机制，即使所有的Eureka Server都挂掉，客户端仍可以利用缓存中的信息调用服务节点的服务。 Eureka一般配合Ribbon进行使用，Ribbon提供了客户端[负载均衡](https://cloud.tencent.com/product/clb?from=10680)的功能，Ribbon利用从Eureka中读取到的服务信息，在调用服务节点提供的服务时，会合理的进行负载。 Eureka通过心跳检测、健康检查、客户端缓存等机制，保证了系统具有高可用和灵活性。 

------

## [Hystrix](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2FNetflix%2FHystrix)

Netflix 熔断器，容错管理工具，旨在通过熔断机制控制服务和第三方库的节点,从而对延迟和故障提供更强大的容错能力。

>    [使用Hystrix实现自动降级与依赖隔离-微服务](https://www.jianshu.com/p/138f92aa83dc) 

------

## [Zuul](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2FNetflix%2Fzuul)

Netflix Zuul 是在云平台上提供动态路由,监控,弹性,安全等边缘服务的框架。Zuul 相当于是设备和 Netflix 流应用的 Web 网站后端所有请求的前门。

>    [Netflix Zuul与Nginx 性能对比](https://www.jianshu.com/p/a4990fb371d4) 

------

## [Archaius](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2FNetflix%2Farchaius)

Netflix 配置管理API，包含一系列配置管理API，提供动态类型化属性、线程安全配置操作、轮询框架、回调机制等功能。

>    archaius是Netflix公司开源项目之一，基于java的配置管理类库，主要用于多配置存储的动态获取。主要功能是对apache common configuration类库的扩展。在云平台开发中可以将其用作分布式配置管理依赖构件。 《Netflix Archaius 分布式配置管理依赖构件》 

------

## Consul

HashiCorp 封装了Consul操作，consul是一个服务发现与配置工具，与Docker容器可以无缝集成。

------

## Spring Cloud for Cloud Foundry

Pivotal 通过Oauth2协议绑定服务到CloudFoundry，CloudFoundry是VMware推出的开源PaaS云平台。

------

## Spring Cloud Sleuth

Spring 日志收集工具包，封装了Dapper和log-based追踪以及Zipkin和HTrace操作，为SpringCloud应用实现了一种分布式追踪解决方案。

------

## Spring Cloud Data Flow

Pivotal 大数据操作工具，作为Spring XD的替代产品，它是一个混合计算模型，结合了流数据与批量数据的处理方式。

------

## Spring Cloud Security

Spring 基于spring security的安全工具包，为你的应用程序添加安全控制。

------

## Spring Cloud Zookeeper

Spring 操作Zookeeper的工具包，用于使用zookeeper方式的服务发现和配置管理。

------

## Spring Cloud Stream

Spring 数据流操作开发包，封装了与Redis,Rabbit、Kafka等发送接收消息。

------

## Spring Cloud CLI

Spring 基于 Spring Boot CLI，可以让你以命令行方式快速建立云组件。

------

## [Ribbon](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2FNetflix%2Fribbon)

Netflix 提供云端负载均衡，有多种负载均衡策略可供选择，可配合服务发现和断路器使用。

>    Ribbon 是 Netflix 发布的云中间层服务开源项目，其主要功能是提供客户侧软件负载均衡算法，将 Netflix 的中间层服务连接在一起。 特性： 

-   Multiple and pluggable load balancing rules
-   Integration with service discovery
-   Built-in failure resiliency
-   Cloud enabled
-   Clients integrated with load balancers
-    Archaius configuration driven client factory

------

## Turbine

Netflix Turbine是聚合服务器发送事件流数据的一个工具，用来监控集群下hystrix的metrics情况。

------

## [Feign](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2FNetflix%2Ffeign)

OpenFeign Feign是一种声明式、模板化的HTTP客户端。

>    通过Feign， 我们能把HTTP远程调用对开发者完全透明，得到与调用本地方法一致的编码体验。这一点与阿里Dubbo中暴露远程服务的方式类似，区别在于Dubbo是基于私有二进制协议，而Feign本质上还是个HTTP客户端。如果是在用Spring Cloud Netflix搭建微服务，那么Feign无疑是最佳选择。 

------

## Spring Cloud Task

Spring 提供云端计划任务管理、任务调度。

------

## Spring Cloud Connectors

Spring 便于云端应用程序在各种PaaS平台连接到后端，如：数据库和消息代理服务。

------

## Spring Cloud Cluster

Spring 提供Leadership选举，如：Zookeeper, Redis, Hazelcast, Consul等常见状态模式的抽象和实现。

------

## Spring Cloud Starters

Pivotal Spring Boot式的启动项目，为Spring Cloud提供开箱即用的依赖管理。