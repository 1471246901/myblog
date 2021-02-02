# springcloud与consul服务注册与发现整合

## 将服务提供者注册进consul

### 创建项目,并修改pom

创建项目 `cloud-providerconsul-payment8006` 

修改pom文件

```xml
<description>支付服务的提供者之注册中心consul</description>

    <dependencies>
        <!--SpringCloud consul-server-->
        <!-- 这里是consul 服务注册使用的starter -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
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

### 创建并编写application.yml文件

```yml
server:
  # consul服务端口
  port: 8006
spring:
  application:
    name: cloud-provider-payment
  cloud:
    consul:
      # consul注册中心地址
      host: localhost
      port: 8500
      discovery:
      # 
        hostname: 127.0.0.1
        service-name: ${spring.application.name}
```

**consul 相关配置参数**

-   `debug`配置是否调试模式，如果打包发布的话，需要设置为`false`。
-   `server.port`配置的是 Spring Boot 服务的端口。
-   `spring.cloud.consul.host/port`配置的是本地 Consul 的地址和端口（Server 节点和 Client 节点都可以），Spring Cloud Consul 会调用 Consul HTTP REST 接口，进行服务注册。
-   `spring.cloud.consul.discovery.true`配置启动是否注册服务，
-   `spring.cloud.consul.discovery.hostname`配置 Spring Boot 服务的主机地址，也可以不进行配置，默认本机地址。
-   `spring.cloud.consul.discovery.serviceName`配置 Consul 注册的服务名称，`${spring.application.name}`变量是我们上面`application.properties`配置文件中添加的配置。
-   `spring.cloud.consul.discovery.healthCheckPath`配置 Consul 健康检查地址，Actuator 组件帮我们进行了实现，所以我们不需要额外的实现，地址在服务启动的时候，打印信息里面可以看到。
-   `spring.cloud.consul.discovery.healthCheckInterval`配置 Consul 健康检查频率，也就是心跳频率。
-   `spring.cloud.consul.discovery.tags`配置 Consul 注册服务的 Tags，设置为`urlprefix-/serviceName`的格式，是自动注册到 Fabio 集群中。
-   `spring.cloud.consul.discovery.instanceId`配置 Consul 注册服务 ID。

### 编写启动类

```java
/**
 * @Description
 * @Author Gushiyu
 * @Date 2021-02-01 16:36
 */
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain8006 {
    public static void main(String[] args) {
            SpringApplication.run(PaymentMain8006.class, args);
    }
}
```

### 编写业务逻辑

创建controller类

```java
@RestController
@Slf4j
public class PaymentController {
    @Value("${server.port}")
    private String serverPort;

    @RequestMapping(value = "/payment/consul")
    public String paymentConsul() {
        return "SpringCloud with consul:" + serverPort + "\t" + UUID.randomUUID().toString();
    }
}
```

### 测试

启动本模块.然后访问 `/payment/consul` 链接

![image-20210201164510703](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210201164510703.png)

访问consul的ui界面

![image-20210201164417108](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210201164417108.png)

## 将服务消费者注册金Consul

### 创建项目,并修改pom

创建 `cloud-consumerconsul-order8080` 项目,并且编写pom文件

#### pom

和服务提供者一样

```xml
<description>服务消费者之注册中心consul</description>

    <dependencies>
        <!--SpringCloud consul-server-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
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
        <!--监控-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--热部署-->
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

###  创建application.yml文件

```yml
server:
  port: 8080
spring:
  application:
    name: cloud-consumer-order
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        hostname: 127.0.0.1
        service-name: ${spring.application.name}
# 上文有解释
```

### 编写启动类

```
@SpringBootApplication
@EnableDiscoveryClient
public class OrderMain8080 {
    public static void main(String[] args) {
            SpringApplication.run(OrderMain8080.class, args);
    }
}
```

### 编写业务逻辑

#### 注入Bean ,,编写config

```java
@Configuration
public class ApplicationContextConfig {
    
    @Bean
    @LoadBalanced
    public RestTemplate resttemplate ( ){
        return new RestTemplate();
    }
}

```

#### 编写Controller

```java
/**
 * @Description
 * @Author Gushiyu
 * @Date 2021-02-01 16:57
 */
@RestController
public class OrderController {

    @Autowired
    RestTemplate restTemplate;

    private final String PAYMENT_URL = "http://cloud-provider-payment";

    @GetMapping("/concumer/consul")
    public String orderConsul ( ){
        return restTemplate.getForObject(PAYMENT_URL+"/payment/consul",String.class);
    }
}

```

## 测试

先启动服务提供者,在启动服务消费者,访问 `localhost:8080/concumer/consul` 

![image-20210201170506676](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210201170506676.png)

查看Consul的ui

![image-20210201170442559](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210201170442559.png)