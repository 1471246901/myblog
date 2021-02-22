# Sentinel的流控规则

流量限制控制规则

## Sentinel名词解释

-   资源名：唯一名称，默认请求路径
-   针对来源：Sentinel可以针对调用者进行限流，填写微服务名，指定对哪个微服务进行限流 ，默认default(不区分来源，全部限制)
-   阈值类型/单机阈值：
    1、QPS(每秒钟的请求数量)：当调用该接口的QPS达到了阈值的时候，进行限流；
    2、线程数：当调用该接口的线程数达到阈值时，进行限流,对响应线程数做出限制
-   是否集群：不需要集群,及对集群中所有该请求同时设置规则
-   流控模式：
    1、直接：接口达到限流条件时，直接限流
    2、关联：当关联的资源达到阈值时，就限流自己
    3、链路：只记录指定链路上的流量（指定资源从入口资源进来的流量，如果达到阈值，就可以限流）[api级别的针对来源]
-   流控效果
    1、快速失败：直接失败
    2、Warm Up：即请求 QPS 从 threshold / 3 开始，经预热时长逐渐升至设定的 QPS 阈值
    3、排队等待

## 流控模式-直接-默认

接口达到限流条件时，直接限流

![image-20210221181120535](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210221181120535.png)

访问超过1秒直接失败

## 流控模式-关联

当关联的资源达到阈值时，就限流自己

![image-20210221181501559](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210221181501559.png)

对`/test/a` 进行流控,当 `/test/b` 满足流控条件是,对 `/test/a` 进行限流

## 流控模式-链路

只记录指定链路上的流量（指定资源从入口资源进来的流量，如果达到阈值，就可以限流）[api级别的针对来源]

记录调用链路上的某一个资源,当那个资源满足流控规则是,对当前资源限流

## 流控效果-快速失败

满足流控规则就直接失败

实现源码 com.alibaba.csp.sentinel.slots.block.controller.DefaultController

## 流控效果-预热

即请求 QPS 从 threshold / 3 开始，经预热时长逐渐升至设定的 QPS 阈值

当突然遇到大量的请求时,慢慢的经过预热时长 达到最初设定的阈值,适合突然间大量的请求并且保持一定时间

实现源码 com.alibaba.csp.sentinel.slots.block.controller.DefaultController

![image-20210221182708298](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210221182708298.png)

## 流控效果-排队等待

排成一队,按照设定的阈值一个接一个处理

适合 这一秒突然大量请求,下一秒突然安静 

实现源码 com.ailibaba.csp.sentinel.slots.block.controller.RateLimiterController

![image-20210221183005361](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210221183005361.png)