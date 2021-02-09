# SpringConfig 与微服务整合

## 创建git仓库

使用github创建 `springcloud-config` 仓库 `Repository`

将获得的 `https://github.com/1471246901/springcloud-config.git` 复制下来

向仓库中提交文件

![image-20210209151432307](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210209151432307.png)

![image-20210209151645638](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210209151645638.png)





## 创建ConfigServer

新建模块 	`cloud-config-center-3344`  

### pom

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
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

### yml

```yml
server:
  port: 3344
spring:
  application:
    name: cloud-config-center  # 注册金eureka的微服务名
  cloud:
    config:
      server:
        git:
          uri: https://github.com/1471246901/springcloud-config.git
          search-paths:
            - springcloud-config
      label: master  # 读取master分支的内容
eureka:
  instance:
    instance-id: cloud-config-center-3344
    prefer-ip-address: true
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka7003.com:7003/eureka/
```

### 启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableConfigServer
public class ConfigCenterMain3344 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigCenterMain3344.class, args);
    }
}
```

### 启动并测试

启动服务 `cloud-config-center-3344` 和 eureka 服务端

访问 7003 eureka ![image-20210209154048380](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210209154048380.png)

访问 `localhost:3344/master/app1-dev.yml`

![image-20210209154211074](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210209154211074.png)

访问 `http://localhost:3344/master/app2-dev.yml`

![image-20210209154236462](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210209154236462.png)

## 创建config 客户端

创建模块 `cloud-config-client-app1-1230` 

### pom 

两个模块通用

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
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

### yml

#### **bootstrap.yml**

bootstrap.yml 是系统级的配置文件,优先级更高

在springboot 启动的时候,会创建一个Bootstrap Context , 作为Spring 应用的 Application Context 的父上下文,初始化的时候 Bootstrap Context 负责从外部源加载配置属性并且解析配置,这两个上下文共享一个才能够外部获取的 `environment` 

Bootstrap  属性具有高优先级,默认情况下,他们不会被本地配置覆盖, Bootstrap Context 和 Application Context 有着不同的约定,所以要新增一个bootstrap.yml 文件 保证 Bootstrap 和 Application  配置的分离

Bootstrap.yml 文件时比 application.yml 先加载的

所以我们在resources 文件夹下创建 `bootstrap.yml` 文件

**bootstrap.yml**   1230

```yml
server:
  port: 1230
spring:
  application:
    name: cloud-config-client-app1
  cloud:
    config:
      label: master  # 分支名称
      name: app1   # 配置文件名称
      profile: dev  # 配置文件后缀
      uri: http://localhost:3344  # configserver 配置中心地址
eureka:
  instance:
    instance-id: cloud-config-client-app1-1230
    prefer-ip-address: true
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka7003.com:7003/eureka/
```

### 启动类

```java
@EnableEurekaClient
@SpringBootApplication
public class ConfigClientMain1230 {
    public static void main(String[] args) {
            SpringApplication.run(ConfigClientMain1230.class, args);
    }
}
```

### 业务类

controller

```java
@RestController
public class ConfigController {

    @Value("${config.info}")
    String info;

    public String info ( ){
        return info;
    }
}
```

### 测试

启动 `cloud-config-client-app1-1230` ,  `cloud-config-center-3344` 和 eureka 服务端 

访问 `localhost:1230/info`

![image-20210209161505656](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210209161505656.png)

