# Ribbon

Spring Cloud Ribbon是一个基于HTTP和TCP的客户端负载均衡工具，它基于Netflix Ribbon实现。通过Spring Cloud的封装，可以让我们轻松地将面向服务的REST模版请求自动转换成客户端负载均衡的服务调用。Spring Cloud Ribbon虽然只是一个工具类框架，它不像服务注册中心、配置中心、API网关那样需要独立部署，但是它几乎存在于每一个Spring Cloud构建的微服务和基础设施中。因为微服务间的调用，API网关的请求转发等内容，实际上都是通过Ribbon来实现的，包括后续我们将要介绍的Feign，它也是基于Ribbon实现的工具。所以，对Spring Cloud Ribbon的理解和使用，对于我们使用Spring Cloud来构建微服务非常重要。

通过Spring Cloud Ribbon的封装，我们在微服务架构中使用客户端负载均衡调用非常简单，只需要如下两步：

​    ▪️服务提供者只需要启动多个服务实例并注册到一个注册中心或是多个相关联的服务注册中心。

​    ▪️服务消费者直接通过调用被@LoadBalanced注解修饰过的RestTemplate来实现面向服务的接口调用。

​    这样，我们就可以将服务提供者的高可用以及服务消费者的负载均衡调用一起实现了。

## 常用配置

#### 禁用 Eureka

当我们在 RestTemplate 上添加 @LoadBalanced 注解后，就可以用服务名称来调用接口了，当有多个服务的时候，还能做负载均衡。

这是因为 Eureka 中的服务信息已经被拉取到了客户端本地，如果我们不想和 Eureka 集成，可以通过下面的配置方法将其禁用。

\# 禁用 Eureka
ribbon.eureka.enabled=false

当我们禁用了 Eureka 之后，就不能使用服务名称去调用接口了，必须指定服务地址。

####  配置接口地址列表

上面我们讲了可以禁用 Eureka，禁用之后就需要手动配置调用的服务地址了，配置如下：

\# 禁用 Eureka 后手动配置服务地址
ribbon-config-demo.ribbon.listOfServers=localhost:8081,localhost:8083

这个配置是针对具体服务的，前缀就是服务名称，配置完之后就可以和之前一样使用服务名称来调用接口了。

#### 配置负载均衡策略

Ribbon 默认的策略是轮询，从我们前面讲解的例子输出的结果就可以看出来，Ribbon 中提供了很多的策略，这个在后面会进行讲解。我们通过配置可以指定服务使用哪种策略来进行负载操作。

#### 超时时间

Ribbon 中有两种和时间相关的设置，分别是请求连接的超时时间和请求处理的超时时间，设置规则如下：

\# 请求连接的超时时间
ribbon.ConnectTimeout=2000
\# 请求处理的超时时间
ribbon.ReadTimeout=5000

也可以为每个Ribbon客户端设置不同的超时时间, 通过服务名称进行指定：
ribbon-config-demo.ribbon.ConnectTimeout=2000
ribbon-config-demo.ribbon.ReadTimeout=5000

#### 并发参数

\# 最大连接数
ribbon.MaxTotalConnections=500
\# 每个host最大连接数
ribbon.MaxConnectionsPerHost=500



## maven引入

在springcloud的新版starter中使用 类似服务注册`spring-cloud-starter-consul-discovery` 或者 `spring-cloud-starter-netflix-eureka-client` 都是默认引用了Ribbon

### 自己引入

```xml
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```



















>     部分引用自https://www.jianshu.com/p/1bd66db5dc46