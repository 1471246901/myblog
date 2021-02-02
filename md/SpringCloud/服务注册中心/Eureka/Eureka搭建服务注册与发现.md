# Euraka搭建服务注册与发现

![image-20210131104632398](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210131104632398.png)

按照图中去实现Eureka集群

## 实现Euraka集群

### 创建模块 编写pom

创建两个模块 `cloud-eureka-server7001` , `cloud-eureka-server7002`  作为Eureka集群

#### pom文件

```xml
<dependencies>
        <!--eureka-server-->
    	<!-- 使用 spring-cloud-starter-netflix-eureka-server starter 来构建eureka 服务端-->
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

### 修改host文件

在本机目录下修改host文件，以便模拟不同机器下的分布式环境

```properties
//将以下内容写入host文件
127.0.0.1 eureka7001.com 
127.0.0.1 eureka7002.com
//使用 ipconfig /flushdns 刷新host配置
```

### 编写application.yml文件

#### 7002

```yml
server:
  port: 7002
spring:
  application:
    name: cloud-eureka-server
eureka:
  instance:
    # server端的hostname
    hostname: eureka7002.com
  client:
    # 不注册自己
    register-with-eureka: false
    # 表示自己是注册中心，不需要检索服务
    fetch-registry: false
    # 默认服务注册中心地址 多个使用，隔开
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
  server:
    # 禁用自我保护，在未收到服务的心跳包时立即将服务剔除
    enable-self-preservation: false
    # eureka server清理无效节点的时间间隔，默认60000毫秒，即60秒,
    eviction-interval-timer-in-ms: 2000

```

#### 7001

```yml
server:
  port: 7001
spring:
  application:
    name: cloud-eureka-server
eureka:
  instance:
    # server端的hostname
    hostname: eureka7001.com
  client:
    # 不注册自己
    register-with-eureka: false
    # 表示自己是注册中心，不需要检索服务
    fetch-registry: false
    # 默认服务注册中心地址 多个使用，隔开
    service-url:
      defaultZone: http://eureka7002.com:7002/eureka/
  server:
    # 禁用自我保护，在未收到服务的心跳包时立即将服务剔除
    enable-self-preservation: false
    # eureka server清理无效节点的时间间隔，默认60000毫秒，即60秒,
    eviction-interval-timer-in-ms: 2000
```

和7001 基本相同

### 编写启动类

在两个 模块下编写启动类 `org.gushiyu/springcloud.EurekaMain7001`,`org.gushiyu/springcloud.EurekaMain7002`

#### EurekaMain7001

```java
@SpringBootApplication
@EnableEurekaServer
// EnableEurekaServer 表示开启EurekaServer 服务
public class EurekaMain7001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaMain7001.class,args);
    }
}
```

EurekaMain7002类似



## 实现服务提供

### 创建项目 编写pom

创建两个模块 `cloud-provider-payment8001`，`cloud-provider-payment8002`  

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
     
     	<!-- 使用 spring-cloud-starter-netflix-eureka-client eureka客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

    </dependencies>
```

两个模块的pom文件是一样的

### 编写application.yml文件

```yml
server:
  port: 8001
spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/cloud2020?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=UTC
    username: root
    password: root
mybatis:
  type-aliases-package: org.gushiyu.springcloud.entities   # entity所在包
  mapper-locations: classpath:mapper/*.xml
# 下面是关于eureka的配置
eureka:
  instance:
    instance-id: cloud-payment-service-8001
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
    # eureka 集群的地址
      defaultZone:       http://eureka7002.com:7002/eureka/,http://eureka7001.com:7001/eureka/
    # 表示客户端每个多少秒去拉取服务注册信息，默认为30秒
    registry-fetch-interval-seconds: 30
```

两个模块的yml配置文件一样，只是 instance-id 等替换为各自就好

### 编写启动类

编写 `org.gushiyu.springcloud.PaymentMain8001`,`org.gushiyu.springcloud.PaymentMain8002`

内容 连个模块的启动类类似，参照8001

```java
@SpringBootApplication
@EnableEurekaClient
// 开启 EurekaClient 的功能
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class, args);
    }
}

```

### 编写业务代码

实现`PaymentController`

实现简单的控制器

```java
	@Value("${server.port}")
    String port;

    @GetMapping("/test")
    public CommonResult<String> test(){
        return  new CommonResult<>(200,"port in data",port);
    }
```

## 编写服务消费

### 创建模块 编写pom

创建模块 `cloud-consumer-order8080` 

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
    	<!-- 在这里服务消费者和服务提供者都是用 eureka的客户端组件 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>
```

### 编写application.yml文件

```yml
server:
  port: 8080
eureka:
  instance:
    instance-id: cloud-order-service-8080
    # 使用ip进行注册
    prefer-ip-address: true
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
    # eureka集群的地址
      defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7001.com:7001/eureka/
spring:
  application:
    name: cloud-order-service
```

### 编写启动类

`org.gushiyu.springcloud.OrderMain8080` 类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient   //开启客户端服务发现，开启后客户端可以主动去选择要访问的服务
public class OrderMain8080 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain8080.class,args);
    }
}
```

### 编写业务代码

首先在没有Ribbon的时候，我们使用 RestTemplate来实现服务调用

向容器中注入 ` RestTemplate`  ,编写配置类注入即可

```java
@Configuration
public class ApplicationConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

`@LoadBalanced ` 注解会使用ribbon进行负载均衡，如果不使用注解，则restTemplate 不可以使用服务名进行服务调用，则应使用某一服务的地址进行调用

可以注入 

```java
@Autowired
DiscoveryClient discoveryClient;
```

用于服务发现，使用服务发现应当在启动类标注 `@EnableDiscoveryClient`

编写Controller类

```java
    public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";  //使用服务名进行调用

    @Autowired
    private RestTemplate restTemplate;

	//实现 test 控制器
    @GetMapping("/test")
    public CommonResult<String> getPort(){
        return restTemplate.getForObject(PAYMENT_URL+"/test",CommonResult.class);
    }
```

## 测试

访问 localhost:8080/test 可以以轮询形式访问两台服务提供，访问 localhost:7002 或7001 可以看到集群的服务信息

