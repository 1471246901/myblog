# Nacos做服务配置中心



搭建Nacos 配置中心



## Nacos 服务端下载安装

参照[NacosSpringCloud搭建](../../服务注册中心/Nacos/NacosSpringCloud搭建.md/#Nacos 服务端下载安装)

进入到 nacos 管理界面, 创建配置文件

### 配置文件拉取规则

${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension} 

### 创建文件

cloud-nacos-consumer-dev.yaml  和 cloud-nacos-payment-dev.yaml

![image-20210218143115607](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210218143115607.png)

注意 yml 获取不到配置

![image-20210218144213847](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210218144213847.png)



## Nacos Config 客户端搭建

修改  NacosSpringCloud搭建 中的三个项目

### pom

增加Config 相关starter

```xml
<dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-nacos-config</artifactId>
        </dependency>
```

### yml

在三个模块中的 application.yml 添加 `spring.profiles.active` 配置,并且将其他配置移动到对应的 bootstrap.yml中

```yml
spring:
  profiles:
    active: dev
```

创建 bootstrap.yml    8001

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
      config:
        server-addr: 127.0.0.1:8848  # config center 的 URL
        file-extension: yaml # 指定yaml作为配置文件的格式,我们这里填写的是yaml,那么我们创建配置文件时也应时yaml ,不应是yml
# ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension} 即是从 config center 中拉取的配置文件
management:
  endpoints:
    web:
      exposure:
        include: '*'
# 微服务名称
service-url:
  cloud-nacos-payment: http://cloud-nacos-payment

```

创建 bootstrap.yml    9001

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
      config:
        server-addr: 127.0.0.1:8848  # config center 的 URL
        file-extension: yaml # 指定yaml作为配置文件的格式
      # ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension} 即是从 config center 中拉取的配置文件
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

创建 bootstrap.yml    9002

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
      config:
        server-addr: 127.0.0.1:8848  # config center 的 URL
        file-extension: yaml # 指定yaml作为配置文件的格式
      # ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension} 即是从 config center 中拉取的配置文件

management:
  endpoints:
    web:
      exposure:
        include: '*'
```

### 业务类

增加业务类

controller

```java
@RestController
@RefreshScope   //增加动态刷新功能
public class PaymentController {

    @Value("${server.port}")
    String serverPort;

    
    @GetMapping("/test")
    public String test ( ){

        return "test on "+serverPort;
    }
    
    //增加一个请求用于获取配置信息
    @Value("${config.info}")
    String configInfo;
    
    @GetMapping("/config/info")
    public String info ( ){
        return configInfo;
    }
}

```



## 测试

启动  nacos config center  和 三个模块

访问 `localhost:8001/config/info` 和 `localhost:9001/config/info` 或 `localhost:9002/config/info`

![image-20210218144523389](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210218144523389.png)

### 测试刷新功能

进入 nacos控制台,修改配置文件的内容,再次访问`localhost:9002/config/info`等

![image-20210218144646661](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210218144646661.png)

可以发现配置自动发生改变



## nacos 配置中心的几种配置拉取规则

nacos 的层级关系设计为 namespace+ group + DATAID ,**这样的层级关系服务注册也可以使用**

**namespace**  位于最顶层,提供应用级别的分级,不同应用通过不同的namespace 分隔开,也可以作为不同开发环境之间的分组

**group**  位于中层,在一个应用中提供分组功能,将所有微服务分为不同的组

**DATAID**  为每一个微服务提供配置管理

在 配置文件可以指定 

```yml
spring:
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848  # config center 的 URL
        file-extension: yaml # 指定yaml作为配置文件的格式
        namespace: # 填写namespace
        group: #  填写不同的分组
```

