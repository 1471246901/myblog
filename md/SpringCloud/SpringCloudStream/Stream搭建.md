# Stream搭建

使用RabbitMQ进行搭建,首先要安装 RabbitMQ相关环境

## 生产者搭建

创建模块 `cloud-stream-rabbitmq-provider8801`

### pom

```xml
<!-- stream 依赖 -->
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
        
```

所有

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
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>

    </dependencies>
```

### yml

```yml
server:
  port: 8801
spring:
  application:
    name: cloud-stream-provider
  cloud:
    stream:
      binders:   # 再此处理要绑定的rabbitmq的服务信息;
        defaultRabbit:  # 表示定义的名称,用于binding的整合
          type: rabbit   # 消息组建的类型
          environment:   # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings:   # 服务的整合处理
        output:   # 这个名字是一个通道的名称
          destination: studyExchange    #  表示要使用的Exchange名称
          content-type: application/json  # 表示消息类型
      default-binder: defaultRabbit   # 表示要绑定的具体设置
eureka:
  instance:
    instance-id: cloud-stream-provider-8801
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

### 启动类

```java
@SpringBootApplication
public class StreamRabbitMain8801 {
    public static void main(String[] args) {
        SpringApplication.run(StreamRabbitMain8801.class, args);
    }
}
```

### 业务类

service

```java
@EnableBinding(Source.class)   //定义消息的发送管道
public class IMessageProviderImpl implements IMessageProvider {

    @Autowired
    private MessageChannel output;

    @Override
    public String send() {
        String serial = UUID.randomUUID().toString();
        output.send(MessageBuilder.withPayload(serial).build());
        System.out.println("serial: "+serial);
        return "serial: "+serial;

    }
}
```

controller

```java
@RestController
public class SendMessageController {


    @Autowired
    private IMessageProvider iMessageProvider;

    @GetMapping("/send")
    public String sendMessage(){

        return iMessageProvider.send();
    }
}
```

### 测试

启动rabbitmq,在启动 eureka 和 消息发送者

访问 `localhost:8801/send`

![image-20210215130912186](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210215130912186.png)

进入rabbitmq 管理界面查看

![image-20210215131050388](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210215131050388.png)

## 消费者搭建

搭建 `cloud-stream-rabbitmq-consumer8802`  和  `cloud-stream-rabbitmq-consumer8803`   两个消费者

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
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>

    </dependencies>
```

### yml

```yml
server:
  port: 8802
spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders:   # 再此处理要绑定的rabbitmq的服务信息;
        defaultRabbit:  # 表示定义的名称,用于binding的整合
          type: rabbit   # 消息组建的类型
          environment:   # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings:   # 服务的整合处理
        input:   # 这个名字是一个通道的名称
          destination: studyExchange    #  表示要使用的Exchange名称
          content-type: application/json  # 表示消息类型
      default-binder: defaultRabbit   # 表示要绑定的具体设置
eureka:
  instance:
    instance-id: cloud-stream-consumer-8802
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

8803 稍作修改即可

### 启动类

```java
@SpringBootApplication
public class StreamConsumerMain8802 {
    public static void main(String[] args) {
            SpringApplication.run(StreamConsumerMain8802.class, args);
    }
}
```

### 业务类

controller

```java
@Component   //这个类接收消息,不通过mvc,所以不应使用controller注解
@EnableBinding(Sink.class)
public class ReceiveMessageController {

    @Value("${server.port}")
    private String port;

    @StreamListener(Sink.INPUT)
    public void input(Message<String> message){

        System.out.println(port+"服务器收到 :"+message.getPayload());
    }

}

```

### 测试

启动rabbitmq,在启动 eureka 和 消息发送者 ,再启动  8802接收者和8803接收者

访问 `localhost:8801/send`

![image-20210215133725423](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210215133725423.png)

![image-20210215133736125](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210215133736125.png)



## 消息持久化,重复消费与广播

8802和8803 重复消费了 同一条信息,,原因在于他们两个服务器没有处于同一组下,

同一组下竞争消费,不同组下广播消费

### 重复消费与广播 的解决方法

修改配置,添加组(再yml配置文件中修改)

```yml

      binders:   # 再此处理要绑定的rabbitmq的服务信息;
        defaultRabbit:  # 表示定义的名称,用于binding的整合
          type: rabbit   # 消息组建的类型
          environment:   # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings:   # 服务的整合处理
        input:   # 这个名字是一个通道的名称
          destination: studyExchange    #  表示要使用的Exchange名称
          content-type: application/json  # 表示消息类型
          group: c1   # 这里添加组别
      default-binder: defaultRabbit   # 表示要绑定的具体设置

```

**测试**

8801 发送三条

![image-20210215134415156](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210215134415156.png)

8802接收到 1条

![image-20210215134435116](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210215134435116.png)

8803 接收到 2条

![image-20210215134458867](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210215134458867.png)

### 消息持久化解决方法

只要设置了group组,则消息是持久化的,没有设置消息就不是持久化的