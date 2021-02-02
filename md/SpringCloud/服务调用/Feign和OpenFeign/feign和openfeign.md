# OpenFeign

OpenFeign 和 Feign 一样,在 Feign 停止更新之后 springcloud 推出了 OpenFeign 

Feign 是一个客户端调用工具

Feign是一个声明式的Web服务客户端,让编写Web服务客户端变得非常容易,只需  创建一个接口并在接口上添加注解即可,可以使用接口,进行调用,支持springmvc对调用的参数和返回的结果进行解析, OpenFeign可以和 Ribbon和eureka一起构建声明式调用方式



在Spring Cloud OpenFeign中，除了OpenFeign自身提供的注解（annotation）之外，还可以使用JAX-RS注解，或者Spring MVC注解。下面还是以OpenFeign注解为例介绍用法。



## OpenFeign 和 Feign的区别

![d](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210202171912542.png)

## @FeignClient

OpenFeign提供了两个重要注解@FeignClient和@EnableFeignClients。

@FeignClient注解用于声明Feign客户端可访问的Web服务。

@EnableFeignClients注解用于修饰Spring Boot应用的入口类，以通知Spring Boot启动应用时，扫描应用中声明的Feign客户端可访问的Web服务。

### @FeignClient注解的参数

-   name, value (默认"")，两者等价
-   qualifier (默认"")
-   url (默认"")
-   decode404 (默认false)
-   configuration (默认FeignClientsConfiguration.class)
-   fallback (默认void.class)
-   fallbackFactory (默认void.class)
-   path (默认"")
-   primary (默认true)

### @FeignClient注解的configuration参数

@FeignClient注解的configuration参数，默认是通过FeignClientsConfiguration类定义的，可以配置Client，Contract，Encoder/Decoder等。

FeignClientsConfiguration类中的配置方法及默认值如下：

-   **feignContract**: SpringMvcContract
-   **feignDecoder**: ResponseEntityDecoder
-   **feignEncoder**: SpringEncoder
-   **feignLogger**: Slf4jLogger
-   **feignBuilder**: Feign.Builder
-   **feignClient**: LoadBalancerFeignClient（开启Ribbon时）或默认的HttpURLConnection

### 定制@FeignClient注解的configuration类

@FeignClient注解的默认配置类为FeignClientsConfiguration，我们可以定义自己的配置类如下：

```java
@Configuration
public class MyConfiguration {
	@Bean
	public Contract feignContract(...) {...}

	@Bean
	public Encoder feignEncoder() {...}
	@Bean
	public Decoder feignDecoder() {...}
	...
}
```

然后在使用@FeignClient注解时，给出参数如下：

```java
@FeignClient(name = "myServiceName", configuration = MyConfiguration.class, ...)
public interface MyService {
    @RequestMapping("/")
    public String getName();
    ...
}
```

当然，定制@FeignClient注解的configuration类还可以有另一个方法，直接配置**application.yaml**文件即可，示例如下：

```yml
feign:
  client:
	config:
	  feignName: myServiceName
		connectTimeout: 5000
		readTimeout: 5000
		loggerLevel: full
		encoder: com.example.MyEncoder
		decoder: com.example.MyDecoder
		contract: com.example.MyContract
```

## @EnableFeignClients

### @EnableFeignClients注解的参数

-   value, basePackages (默认{})
-   basePackageClasses (默认{})
-   defaultConfiguration (默认{})
-   clients (默认{})

