# NacosSpringCloud搭建

作为服务注册中心

英文文档

https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_nacos_discovery

中文文档

https://nacos.io/zh-cn/docs/what-is-nacos.html

## Nacos 服务端下载安装

### 下载地址

https://github.com/alibaba/Nacos

https://github.com/alibaba/nacos/releases/tag/1.4.1

下载之后

![image-20210218112053876](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210218112053876.png)

### 解压

![image-20210218112149283](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210218112149283.png)

#### 启动

运行bin目录下的startup.cmd 或者startup.sh  

使用 ` .\startup.cmd -m standalone` 命令启动单机模式

![image-20210218112723427](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210218112723427.png)

访问 `http://localhost:8848/nacos` 

输入 nacos   nacos  登录

![image-20210218113233658](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210218113233658.png)

## 搭建服务生产者

搭建两台服务生产者   `cloud-nacos-payment9001` `cloud-nacos-payment9002`   用于提供服务和测试负载均衡

### pom

再父pom中应添加spring-cloud-alibaba-dependencies  dependencyManagement

```xml
<!--Spring cloud alibaba 2.1.0.RELEASE-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring.cloud.alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
```

子模块

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
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-nacos-discovery</artifactId>
        </dependency>
    </dependencies>
```

### yml

`cloud-nacos-payment9001`

```yml
server:
  port: 9001
spring:
  application:
    name: cloud-nacos-payment
  cloud:
    nacos:
      discovery:
        # 注册的地址
        server-addr: 127.0.0.1:8848
        # 注册的服务名,默认为 spring.application.name
        service: ${spring.application.name}
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

`cloud-nacos-payment9002`

```yml
server:
  port: 9002
spring:
  application:
    name: cloud-nacos-payment
  cloud:
    nacos:
      discovery:
        # 注册的地址
        server-addr: 127.0.0.1:8848
        # 注册的服务名,默认为 spring.application.name
        service: ${spring.application.name}
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

### 启动类

`cloud-nacos-payment9001`  和 `cloud-nacos-payment9002`

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosPaymentMain9001 {
    public static void main(String[] args) {
            SpringApplication.run(NacosPaymentMain9001.class, args);
    }
}
```

### 业务类

**config**    `cloud-nacos-payment9001`  和 `cloud-nacos-payment9002`

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

**controller**   `cloud-nacos-payment9001`  和 `cloud-nacos-payment9002`

```java
@RestController
public class PaymentController {

    @Value("${server.port}")
    String serverPort;

    @GetMapping("/test")
    public String test ( ){

        return "test on "+serverPort;
    }
}
```

### 测试

启动 `cloud-nacos-payment9001`  和 `cloud-nacos-payment9002` 和 nacos服务端

访问 `localhost:8848/nacos`

![image-20210218133622922](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210218133622922.png)

访问 `localhost:9001/test`  和 `localhost:9002/test`

![image-20210218133715813](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210218133715813.png)

## 搭建服务消费者

创建模块 `cloud-nacos-consumer8001`

### pom

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
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
        </dependency>
    </dependencies>
```

### yml

```yml
server:
  port: 8001
spring:
  application:
    name: cloud-nacos-consumer
  cloud:
    nacos:
      discovery:
        # 注册的地址
        server-addr: 127.0.0.1:8848
        # 注册的服务名,默认为 spring.application.name
        service: ${spring.application.name}
management:
  endpoints:
    web:
      exposure:
        include: '*'
# 微服务名称
service-url:
  cloud-nacos-payment: http://cloud-nacos-payment

```

### 启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosConsumerMain8001 {
    public static void main(String[] args) {
            SpringApplication.run(NacosConsumerMain8001.class, args);
    }
}

```

### 业务类

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
public class ConsumerController {

    @Autowired
    private RestTemplate restTemplate;

    @Value("${service-url.cloud-nacos-payment}")
    private String paymentUrl;

    @GetMapping("/c/test")
    public String test() {
        System.out.println(paymentUrl);
        return restTemplate.getForObject(paymentUrl+"/test", String.class);
    }
}

```

## 测试

启动 `cloud-nacos-consumer8001` `cloud-nacos-payment9001`  和 `cloud-nacos-payment9002` 和 nacos服务

访问 `http://localhost:8848/nacos/`

![image-20210218135238492](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210218135238492.png)

访问 `localhost:8001/c/test`

![image-20210218135700424](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210218135700424.png)

实现了负载均衡,**并且nacos的负载均衡使用Ribbon实现**