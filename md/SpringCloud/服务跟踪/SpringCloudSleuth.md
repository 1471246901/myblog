# SpringCloudSleuth

Spring Cloud Sleuth为Spring Cloud实现了分布式跟踪解决方案。在分布式系统中提供追踪解决方案并且兼容支持了zipkin ,就是对zipkin做了一些增强

Spring Cloud Sleuth借用了Google

Dapper的术语。

**Span**:工作的基本单位。例如，发送RPC是一个新的跨度，就像发送响应到RPC一样。Span是由一个唯一的64位ID来标识的，而另一个64位ID用于跟踪。span还具有其他数据，如描述、时间戳事件、键值标注(标记)、导致它们的span的ID和进程ID(通常是IP地址)。

可以启动和停止跨度，并跟踪其时间信息。 创建跨度后，必须在将来的某个时刻停止它。

启动跟踪的初始范围称为根跨度。 该范围的ID值等于跟踪ID。

**Trace**：一组span形成树状结构。 例如，如果运行分布式大数据存储，则可能由PUT请求形成跟踪。

**注解**：用于及时记录事件的存在。 使用Brave工具，我们不再需要为Zipkin设置特殊事件，以了解客户端和服务器是谁，请求开始的位置以及结束位置。

**cs**：客户已发送。 客户提出了请求。 此注释表示跨度的开始。
**sr**：Server Received：服务器端获得请求并开始处理它。 从此时间戳中减去cs时间戳会显示网络延迟。
**ss**：服务器已发送。 在完成请求处理时（当响应被发送回客户端时）注释。 从此时间戳中减去sr时间戳会显示服务器端处理请求所需的时间。
**cr**：客户收到了。 表示跨度的结束。 客户端已成功收到服务器端的响应。 从此时间戳中减去cs时间戳会显示客户端从服务器接收响应所需的全部时间。

下图显示了Span和Trace在系统中的外观以及Zipkin注解：

![image-20210217175524248](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210217175524248.png)

## 搭建SpringCloudSleuth



### 搭建SpringCloudSleuth 服务端

SpringCloudSleuth 从F版启,就独立使用jar包了,不需要使用SpringBoot构建

#### 下载 zipkin

https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/

![image-20210217180433150](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210217180433150.png)



#### 运行 zipkin

使用 `java -jar ` 命令运行

![image-20210217180640272](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210217180640272.png)

#### 查看图形界面

访问 `http://localhost:9411/zipkin/` 

![image-20210217180904375](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210217180904375.png)

### 搭建Sleuth 客户端

创建模块  `cloud-provider-payment8010` 服务提供者和 `cloud-consumer-order8011` 服务消费者

#### pom

 `cloud-provider-payment8010` 服务提供者和 `cloud-consumer-order8011` 服务消费者 都使用

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
    
    <!-- 这是zipkin客户端所依赖的 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        </dependency>
    </dependencies>
```

#### yml

`cloud-provider-payment8010`

```yml
server:
  port: 8010
spring:
  application:
    name: cloud-payment-service
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
      # 采样率 介于0-1之间,0采样率最小,一般使用0.5
      probability: 1
eureka:
  instance:
    instance-id: cloud-payment-service-8010
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

`cloud-consumer-order8011`

```yml
server:
  port: 8011
eureka:
  instance:
    instance-id: cloud-order-service-8011
    prefer-ip-address: true
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka7003.com:7003/eureka/
spring:
  application:
    name: cloud-order-service
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
      # 采样率 介于0-1之间,0采样率最小,一般使用0.5
      probability: 1
```

#### 启动类

`cloud-provider-payment8010`

```java
@SpringBootApplication
@EnableEurekaClient
public class PaymentProviderMain8010 {
    public static void main(String[] args) {
            SpringApplication.run(PaymentProviderMain8010.class, args);
    }
}
```

`cloud-consumer-order8011`

```java
@SpringBootApplication
@EnableEurekaClient
public class OrderConsumerMain8011 {
    public static void main(String[] args) {
            SpringApplication.run(OrderConsumerMain8011.class, args);
    }
}
```

#### 业务类

`cloud-provider-payment8010`

```java
@RestController
public class PaymentController {


    @GetMapping("/test/{id}")
    public String test (@PathVariable("id") int id){

        return "id: "+id;
    }
}
```

`cloud-consumer-order8011`

config

```java
@Configuration
public class AppConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

controller

```java
@RestController
public class OrderController {

    @Resource
    private RestTemplate restTemplate;

    @GetMapping("/order/test/{id}")
    public String test (@PathVariable("id") int id){

        return restTemplate.getForObject("http://cloud-payment-service/test/" + id, String.class);
    }
}
```

#### 测试客户端搭建

启动 eureka ,SpringCloudSleuth服务端 ,SpringCloudSleuth客户端两个服务

访问 `localhost:8010/test/1`

![image-20210217184659427](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210217184659427.png)

访问 `localhost:8011/order/test/1`

![image-20210217184741549](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210217184741549.png)

此时eureka 和两个服务均运行正常

### 测试

访问 `http://localhost:9411/zipkin/` 根据服务名查找

![image-20210217184850457](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210217184850457.png)

进入其中一个

![image-20210217184939589](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210217184939589.png)

可以看到服务调用链路,耗时等