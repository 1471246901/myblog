# openfeign简单实用

openfeign 以来了Ribbon 所以在引入openfeign后应删除Ribbon的引用



## maven依赖

```xml
<!--openfeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

```

所有依赖

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

## yml配置文件

yml中只需要配置好eureka的服务发现功能即可

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
    # 对于使用 openfeign 是必要的
    service-url:
      defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7001.com:7001/eureka/
spring:
  application:
    name: cloud-order-service
```

## 启动类标注注解

在启动类中标注 `@EnableFeignClients ` 注解,如果无需配置服务对应的负载均衡规则,那么可以不标注 `@RibbonClient` 

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient   //开启客户端服务发现，开启后客户端可以主动去选择要访问的服务
@ComponentScan(excludeFilters = {@ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {MyRuleConfig.class})})
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE", configuration = MyRuleConfig.class)
@EnableFeignClients
public class OrderMain8080 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain8080.class,args);
    }
}
```

## 编写业务接口

### 编写调用接口

根据服务提供者提供的服务,去编写service接口

```java
@RestController
public class PaymentController {

    @Autowired
    private PaymentService paymentService;

    @GetMapping("/payment/{id}")
    public CommonResult<Payment> getPayment(@PathVariable("id") long id){
        Payment p = paymentService.selectByPrimaryKey(id);
        return new CommonResult<>(200,"查询结果1",p);
    }


    @Value("${server.port}")
    String port;

    @GetMapping("/test")
    public CommonResult<String> test(){
        return  new CommonResult<>(200,"port in data",port);
    }
}
```

业务接口

```java
@Component
@FeignClient("CLOUD-PAYMENT-SERVICE")
public interface PaymentService {

    @GetMapping("/test")
    CommonResult<String> test();

    @GetMapping("/payment/{id}")
    CommonResult<Payment> getPayment(@PathVariable("id") long id);
}
```

编写的接口可以使用springmvc的相关注解 进行序列化

### 编写controller

```java
	@Autowired
    private PaymentService paymentService;
    
    @GetMapping("/payment/{id}")
    public CommonResult<Payment> getPayment(@PathVariable("id") long id) {
        return paymentService.getPayment(id);

    }

    @GetMapping("/port")
    public CommonResult<String> getPort(){
        return paymentService.test();
    }
```

## 测试

启动 eureka 集群,消费提供者集群,还有当前项目

访问 `localhost:8080/port` , `localhost:8080/payment/11111`  查看是否调用成功且有负载均衡

![image-20210203154930835](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210203154930835.png)



