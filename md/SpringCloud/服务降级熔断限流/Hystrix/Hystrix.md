# Hystrix

![image-20210204183101509](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210204183101509.png)

https://github.com/Netflix/hystrix/wiki

*可惜Hystrix 已经不再发布新版本*

Hystrix是一个用于处理分布式系统的延迟和容错的开源库,在分布式系统中,许多依赖不可避免的会调用失败,比如超时,异常等.

Hystrix能够保证在一个依赖出问题的情况下,不会导致整体服务的失败,避免级联故障,以提高分布式系统的弹性

断路器 本身是一种开关装置,当某个服务单元发生故障之后,通过断路器的故障监控(类似熔断保险丝),向调用方返回一个可预期的,可处理的备选相应(FallBack),而不是长时间的等待或者抛出方法无法处理的异常,这样就保证了服务调用方的线程不会被长时间,不必要的占用,从而避免了故障在分布式系统中的蔓延,乃至雪崩.

>   **服务雪崩**
>
>   如微服务调用链   `Service A` → `Service B` → `Service C`
>
>   而此时，`Service A`的流量波动很大，流量经常会突然性增加！那么在这种情况下，就算`Service A`能扛得住请求，`Service B`和`Service C`未必能扛得住这突发的请求。
>   此时，如果`Service C`因为抗不住请求，变得不可用。那么`Service B`的请求也会阻塞，慢慢耗尽`Service B`的线程资源，`Service B`就会变得不可用。紧接着，`Service A`也会不可用，这一过程如下图所示
>   ![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/725429-20190130225824355-156743654.jpg)
>
>   如上图所示，一个服务失败，导致整条链路的服务都失败的情形，我们称之为服务雪崩。

>    **服务熔断**：当下游的服务因为某种原因突然**变得不可用**或**响应过慢**，上游服务为了保证自己整体服务的可用性，不再继续调用目标服务，直接返回，快速释放资源。如果目标服务情况好转则恢复调用。

>   **服务降级**: 
>
>   -   当下游的服务因为某种原因**响应过慢**，下游服务主动停掉一些不太重要的业务，释放出服务器资源，增加响应速度！
>   -   当下游的服务因为某种原因**不可用**，上游主动调用本地的一些降级逻辑，避免卡顿，迅速返回给用户！
>   -   服务熔断属于服务降级的一种

## What Is Hystrix? 什么是Hystrix?

In a distributed environment, inevitably some of the many service dependencies will fail. Hystrix is a library that helps you control the interactions between these distributed services by adding latency tolerance and fault tolerance logic. Hystrix does this by isolating points of access between the services, stopping cascading failures across them, and providing fallback options, all of which improve your system’s overall resiliency.

>   在分布式环境中，许多服务依赖项中的一些不可避免地会失败。Hystrix 是一个库，它通过添加延迟容差和容错逻辑，帮助您控制这些分布式服务之间的交互。Hystrix 通过隔离服务之间的访问点、停止服务之间的级联故障以及提供回退选项来提供服务，所有这些都提高了系统的整体恢复能力。

## What Is Hystrix For? Hystrix  可以干啥?

Hystrix is designed to do the following:

-   Give protection from and control over latency and failure from dependencies accessed (typically over the network) via third-party client libraries.

-   Stop cascading failures in a complex distributed system.

-   Fail fast and rapidly recover.

-   Fallback and gracefully degrade when possible.

-   Enable near real-time monitoring, alerting, and operational control.

    

>   Hystrix 旨在执行以下操作：
>
>   -   保护和控制通过第三方客户端库访问（通常通过网络）访问的依赖项的延迟和故障。
>   -   停止复杂分布式系统中的级联故障。
>   -   失败，快速恢复。
>   -   回退，并尽可能正常降级。
>   -   实现近乎实时的监控、警报和操作控制。

## Hystrix工作流程

![image-20210205142956021](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210205142956021.png)

