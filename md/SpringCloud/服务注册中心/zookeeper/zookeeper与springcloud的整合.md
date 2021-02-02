# zookeeper与springcloud 的整合

使用zookeeper为springcloud提供服务注册与发现

准备一台zookeeper服务器，安装方法见zookeeper篇

我的服务器地址是 192.168.1.20

## 将服务提供者注册进zookeeper

### 创建模块并修改pom

创建 `cloud-provider-payment8004` 模块，修改pom

#### pom

```xml
<description>Zookeeper服务提供者</description>



    <dependencies>
        <dependency>
            <groupId>org.gushiyu.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--SpringBoot整合Zookeeper客户端-->
        <!-- 使用此starter完成与springcloud的整合 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            
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

<span style="color:red">注意</span> `spring-cloud-starter-zookeeper-discovery` 中的zookeeperAPi可能与自己的zookeeper版本不同，需要排除zookeeperapi，在引入自己版本的api既可 例：

```xml
<!--SpringBoot整合Zookeeper客户端-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.6.2</version>
        </dependency>
```

### 编写yml文件

在`cloud-provider-payment8004` 模块创建并编写 application.yml文件

```yml
server:
  # 8004表示注册到zookeeper服务器的支付服务提供者端口号
  port: 8004
spring:
  application:
    # 服务别名---注册zookeeper到注册中心的名称
    name: cloud-provider-payment
  cloud:
    zookeeper:
      # 默认localhost:2181 ,zookeeper 集群使用，号隔开
      connect-string: 192.168.1.20:2181

```

### 编写启动类

编写 `org.gushiyu.springcloud.PaymentMain8004` 类

```java
/**
 * @Description
 * @Author Gushiyu
 * @Date 2021-01-31 13:25
 */
@SpringBootApplication
@EnableDiscoveryClient
//在不使用eureka之后开启服务注册与发现均使用此注解
public class PaymentMain8004 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8004.class,args);
    }
}
```

### 编写业务逻辑

```java
@RestController
@Slf4j
public class PaymentController {

    @Value("${server.port}")
    private String serverPort;

    @RequestMapping(value = "payment/zk")
    public String paymentZk() {
        return "SpringCloud with zookeeper:" + serverPort + "\t" + UUID.randomUUID().toString();
    }
}
```

### 测试

在 zookeeper启动的情况下启动模块 `cloud-provider-payment8004` 查看zookeeper中有无服务注册信息，访问 `localhost:8004/payment/zk` 是否有效

![image-20210131133718289](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210131133718289.png)

![image-20210131133556138](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210131133556138.png)

## 将服务消费者注册进zookeeper

### 创建模块并修改pom

创建模块 `cloud-consumerzk-order8080`  

#### 修改pom

```xml
    <description>订单消费者之注册中心zookeeper</description>

    <dependencies>
        <dependency>
            <groupId>org.gushiyu.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--SpringBoot整合Zookeeper客户端-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.6.2</version>
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

### 编写yml文件

在`cloud-consumerzk-order8080` 模块创建并编写 application.yml文件

```yml
server:
  port: 8080
spring:
  application:
    # 服务别名
    name: cloud-consumer-order
  cloud:
    zookeeper:
      # 注册到zookeeper地址
      connect-string: 192.168.1.20:2181
```

### 编写启动类

```java
/**
 * @Description
 * @Author Gushiyu
 * @Date 2021-01-31 13:56
 */
@SpringBootApplication
@EnableDiscoveryClient
public class OrderZkMain8080 {
    public static void main(String[] args) {
            SpringApplication.run(OrderZkMain8080.class, args);
    }
}

```

### 编写业务代码

向容器中注入 

#### RestTemplate

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced 
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

#### controller

```java
/**
 * @Description
 * @Author Gushiyu
 * @Date 2021-01-31 14:01
 */
@RestController
@Slf4j
public class OrderController {
    public static final String INVOKE_URL = "http://cloud-provider-payment";

    @Autowired
    private RestTemplate restTemplate;


    /**
     * http://localhost:8080/consumer/payment/zk
     *
     * @return
     */
    @GetMapping("/consumer/payment/zk")
    public String paymentInfo() {
        return restTemplate.getForObject(INVOKE_URL + "/payment/zk", String.class);
    }

}

```

### 测试

在 zookeeper启动的情况下启动模块 `cloud-consumerzk-order8080`和 `cloud-provider-payment8004`  查看zookeeper中有无服务注册信息，访问 `localhost:8080/consumer/payment/zk` 是否有效

![image-20210131141028025](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210131141028025.png)

![image-20210131140844779](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210131140844779.png)