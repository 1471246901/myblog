# Hystix 服务降级

## 准备项目

为了简便不使用服务注册集群 

 需要三个 eureka ,provider,consumer

### eureka服务注册中心

创建模块 `cloud-eureka-single-server7003`

#### pom

```xml
<dependencies>
        <!--eureka-server-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.gushiyu.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--一般为通用配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

#### yml

```yml
server:
  port: 7003
spring:
  application:
    name: cloud-eureka-server
eureka:
  instance:
    # server端的hostname
    hostname: eureka7003.com
  client:
    # 不注册自己
    register-with-eureka: false
    # 表示自己是注册中心，不需要检索服务
    fetch-registry: false
    # 默认服务注册中心地址 多个使用，隔开
    # service-url:
    #   defaultZone: http://eureka7001.com:7001/eureka/
  server:
    # 禁用自我保护，在未收到服务的心跳包时立即将服务剔除
    enable-self-preservation: false
    # eureka server清理无效节点的时间间隔，默认60000毫秒，即60秒,
    eviction-interval-timer-in-ms: 2000
```

#### 启动类

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaSingleMain7003 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaSingleMain7003.class, args);
    }
}
```

### provider 服务提供者

创建模块 `cloud-provider-hystrix-payment8003`

#### pom

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.gushiyu.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

    </dependencies>
```

#### yml

```yml
server:
  port: 8003
spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/cloud2020?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=UTC
    username: root
    password: root
eureka:
  instance:
    instance-id: cloud-payment-service-8003
    # 是否使用IP地址进行注册
    prefer-ip-address: true
    # Eureka 客户端向服务端发送心跳的时间间隔，单位默认是30秒
    lease-renewal-interval-in-seconds: 30
    # Eureka 服务端在收到最后一次心跳之后的等待时间上限，单位默认为秒，超时未收到心跳则把服务剔除
    lease-expiration-duration-in-seconds: 90
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7003.com:7003/eureka/
    # 表示客户端每个多少秒去拉取服务注册信息，默认为30秒
    registry-fetch-interval-seconds: 30
```

#### 启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class PaymentHystrixMain8003 {
    public static void main(String[] args) {
            SpringApplication.run(PaymentHystrixMain8003.class, args);
    }
}
```

#### 编写业务逻辑

controller

```java
@RestController
public class PaymentController {

    @Autowired
    PaymentService paymentService;

    @GetMapping("/test/{id}")
    public String test(@PathVariable("id") int id){

        return paymentService.test(id);

    }
}
```

service

```java
@Service
public class PaymentService {


    public String test(int id){
		//等下会添加耗时操作等
        return "this is cloud-provider-hystrix-payment8003/test id : "+id;
    }

}
```



### consumer 服务消费者

创建模块 `cloud-consumer-hystrix-order8080`

#### pom

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.gushiyu.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        
    </dependencies>
```

#### yml

```yml
server:
  port: 8080
eureka:
  instance:
    instance-id: cloud-order-service-8080
    prefer-ip-address: true
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka7003.com:7003/eureka/
spring:
  application:
    name: cloud-order-service

ribbon:
  # 应当把ribbon的 ReadTimeout 和 ConnectTimeout 时间设置大一些,不然在使用 hystix 时超时 ribbon 也会报错
  # 指的是建立连接所用的时间,适用于网络状态正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
  # 指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000
  eureka:
    enabled: true
```

#### 启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
public class OrderHystrixMain8080 {
    public static void main(String[] args) {
            SpringApplication.run(OrderHystrixMain8080.class, args);
    }
}
```

#### 编写业务逻辑

controller

```java
@RestController
public class OrderController {

    @Autowired
    PaymentService paymentService;

    @GetMapping("/order/test/{id}")
    public String test(@PathVariable("id") int id){


        return paymentService.test(id);
    }
}
```

service

```java
@Component
@FeignClient("cloud-payment-service")
public interface PaymentService {

    @GetMapping("/test/{id}")
    public String test(@PathVariable("id") int id);
}
```

### 启动测试

#### 查看eureka 

访问 `http://localhost:7003/`

![image-20210204191246152](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210204191246152.png)

访问payment 的 `localhost:8003/test/2`

![image-20210204191344797](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210204191344797.png)

访问 consumer `localhost:8080/order/test/2`

![image-20210204191546781](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210204191546781.png)

## provider 侧服务降级

### hystix 的 maven 依赖

向 provider  的pom中添加 hystrix starter

```xml
<!--hystrix-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
```

### 激活 hystix  @EnableCircuitBreaker

主启动类激活  添加 `@EnableCircuitBreaker`  也可以添加`@EnableHystrix`  

因为 `@EnableHystrix`继承了`@EnableCricuitBreaker`

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker
public class PaymentHystrixMain8003 {
    public static void main(String[] args) {
            SpringApplication.run(PaymentHystrixMain8003.class, args);
    }
}
```

### 修改 provider 的service  @HystrixCommand

向服务添加 @HystrixCommand 注解

```java
//代表只接受此服务运行3秒钟,超过三秒则跳转到  `test_TimeOutHandler` 方法中,执行服务降级
    @HystrixCommand(fallbackMethod = "test_TimeOutHandler", commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000")
        }
    )
    public String test(int id){
		
        return "this is cloud-provider-hystrix-payment8003/test id : "+id;
    }
```

此时我们应在test 方法中加入耗时操作,模拟操作有时超时 ,在添加 `test_TimeOutHandler` 方法

```java
//代表只接受此服务运行3秒钟,超过三秒则跳转到  `payment_TimeOutHandler` 方法中,执行服务降级
    @HystrixCommand(fallbackMethod = "test_TimeOutHandler", commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000")
        }
    )
    public String test(int id){
        //随机随眠几秒
        int i = (int) (Math.random() * 6);
        for (int i1 = 0; i1 < i; i1++) {
            ThreadUtil.sleep(1000);
        }

        return "this is cloud-provider-hystrix-payment8003/test id : "+id;
    }

    public String test_TimeOutHandler(int id){
        return "触发了服务降级 this is cloud-provider-hystrix-payment8003/test_TimeOutHandler id : "+id;
    }
```

### 测试

多次访问 `localhost:8003/test/1`

在睡眠不超过3秒时 正常返回

![image-20210204194144453](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210204194144453.png)

在超过3秒时

![image-20210204194215007](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210204194215007.png)

此时通过  consumer  端进行访问可以缩短访问时间,保证 consumer  不会因 provicer 的变慢而变慢

多次访问 `localhost:8080/order/test/2`

![image-20210204194503871](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210204194503871.png)

## consumer  侧服务降级

consumer  也可以通过hystix进行服务降级,和provicer相同,但也有一点不同

#### maven依赖

添加 hystix starter

```xml
<!--hystrix-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
```

#### 激活Hystix 

这次使用 `@EnableHystrix`

```java
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
@EnableHystrix
public class OrderHystrixMain8080 {
    public static void main(String[] args) {
            SpringApplication.run(OrderHystrixMain8080.class, args);
    }
}
```

#### ! 在feign 中配置 hystrix

因为 consumer 端的service 是靠 feign 实现的,所以要在  feign 中开启对hystrix 的支持

或者这应是通用配置

yml中配置

```yml
 feign:
  hystrix:
    enabled: true
```

#### ! 修改service

此时我们遇到了问题 , 就是这时的service 是接口的形式,我们无法在其中创建方法

这时我们可以创建一个类 继承自 我们的service接口

```java
/**
 * 这时我们可以将Fallback 单独生成一个类 分离后更清晰
 */
@Component
public class PaymentServiceFallback implements PaymentService {
    @Override
    public String test(int id) {
        return "触发了服务降级 在PaymentServiceFallback 中的test 方法 id:"+id;
    }
}
```

在 PaymentService 接口上标注 `@FeignClient` 的注解加上 fallback 属性 并指向我们刚刚创建的类 `PaymentServiceFallback` ,这时我们由Feign 进行 Fallback的执行 *虽然底层还是由 Hystrix 实现的,这算是Feign 对Hystrix 的兼容?* 

```java
@Component
@FeignClient(value = "cloud-payment-service",fallback = PaymentServiceFallback.class)
public interface PaymentService {

    @GetMapping("/test/{id}")
    public String test(@PathVariable("id") int id);

}
```

现在即可测试,可见PaymentService 实现了服务降级功能,可是无法控制时间

<div style="color:red">这个时候我们发现 `@HystrixCommand` 注解消失不见了 ,那么这个时候应该怎么控制超时时间呢?</div>

**测试**

这时我们在 interface PaymentService 中的 test方法中添加  `@HystrixCommand` 注解

但是启动后访问就会报错,且无法解决

```java
@Component
@FeignClient(value = "cloud-payment-service",fallback = PaymentServiceFallback.class)
public interface PaymentService {


    @GetMapping("/test/{id}")
    @HystrixCommand(commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "1000")
    })
    public String test(@PathVariable("id") int id);
```

又在 PaymentServiceFallback 类中 加入了 `@HystrixCommand` 注解 ,显然更不对

这个时候我使用  `@DefaultProperties` 注解 标注在`interface PaymentService ` 没有报错,但是仍然无法控制超时时间 无语了!!!!

>   -    `@HystrixCommand` 注解不只可以 标注在 service 上 ,还可以标注在conreoller 上和provider 相同 使用 `@HystrixCommand` 注解 和 `@HystrixProperty` 注解 设置降级的规则
>   -   如果开启的`Hystrix`就不建议在用feign的超时配置了

## 设置默认 fallback 

在之前我们一个请求对应一个 fallback  太麻烦因为很多服务无关紧要,使用共同的一个fallback方法就可以,这时我们还要写好多的 `@HystrixCommand` 注解  和 `@HystrixProperty`规则 ,太麻烦

这时我们可以使用 `@DefaultProperties` 注解,它可以将一个类的同一规则提炼出来,标注在类上,这是在方法中  直接标注 `@HystrixCommand` 即可,不需要在`@HystrixCommand` 注解中写一堆配置

**之前**

```java
@Service
public class PaymentService {

    //每个方法写一堆
    @HystrixCommand(fallbackMethod = "test_TimeOutHandler", commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000")
        }
    )
    public String test(int id){
        int i = (int) (Math.random() * 6);
        for (int i1 = 0; i1 < i; i1++) {
            ThreadUtil.sleep(1000);
        }
        return "this is cloud-provider-hystrix-payment8003/test id : "+id;
    }

    public String test_TimeOutHandler(int id){
        return "触发了服务降级AAAAAAAAA this is cloud-provider-hystrix-payment8003/test_TimeOutHandler id : "+id;
    }
}
```

**现在**

```java
@Service
@DefaultProperties(defaultFallback = "test_TimeOutHandler",commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000")
})
public class PaymentService {

    //只需写 HystrixCommand 即可,不配置默认采用 @DefaultProperties的配置
    @HystrixCommand
    public String test(int id){
        //随机随眠几秒
        int i = (int) (Math.random() * 6);
        for (int i1 = 0; i1 < i; i1++) {
            ThreadUtil.sleep(1000);
        }

        return "this is cloud-provider-hystrix-payment8003/test id : "+id;
    }

    public String test_TimeOutHandler(int id){
        return "触发了服务降级AAAAAAAAA this is cloud-provider-hystrix-payment8003/test_TimeOutHandler id : "+id;
    }
}
```

## @HystrixProperty中的规则

HystrixProperty 规则很多,但我不知在哪里,其实是在 `com.netflix.hystrix.HystrixCommandProperties`类中,找到该类查看即可

![image-20210204211427026](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210204211427026.png)



![image-20210204211432902](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210204211432902.png)

![image-20210204211443709](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210204211443709.png)

![image-20210204211459796](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210204211459796.png)



