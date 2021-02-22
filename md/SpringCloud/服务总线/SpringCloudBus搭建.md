# SpringCloudBus搭建

使用SpringCloudBus 搭建动态刷新配置的功能,利用消息总线触发一个服务端ConfigServer的/bus/refresh端点，从而刷新所有客户端配置

SpringCloudConfig 的基础上搭建 , 另外需要准备rabbitMQ 消息中间件

新建一个ConfigClient的模块,功能和 `cloud-config-client-app1-1230 ` 相同代码相同

## 新建模块,修改原有的 1230模块

创建模块`cloud-config-client-app1-1231 ` 

### pom   

 `cloud-config-client-app1-1230 `  也要添加 `spring-cloud-starter-bus-amqp`

```xml
<dependencies>
    	<!-- 添加了spring-cloud-starter-bus-amqp支持  -->
    	<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
    
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    	<!-- 对于动态刷新配置actuator 也是必要的 -->
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

### yml      configclient中的yml

1230

```yml
server:
  port: 1230
spring:
  application:
    name: cloud-config-client-app1
  cloud:
    config:
      label: master
      name: app1
      profile: dev
      uri: http://localhost:3344
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
eureka:
  instance:
    instance-id: cloud-config-client-app1-1230
    prefer-ip-address: true
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka7003.com:7003/eureka/
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

1231

```yml
server:
  port: 1231
spring:
  application:
    name: cloud-config-client-app1
  cloud:
    config:
      label: master
      name: app1
      profile: dev
      uri: http://localhost:3344
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
eureka:
  instance:
    instance-id: cloud-config-client-app1-1231
    prefer-ip-address: true
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka7003.com:7003/eureka/
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

### 启动类

1231 

```java
@EnableEurekaClient
@SpringBootApplication
public class ConfigClientMain1231 {
    public static void main(String[] args) {
            SpringApplication.run(ConfigClientMain1231.class, args);
    }
}
```

1230 无需改变

### 业务类

controller

```java
package org.gushiyu.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Description
 * @Author Gushiyu
 * @Date 2021-02-09 16:06
 */
@RestController
@RefreshScope   //标注动态刷新是必要的
public class ConfigController {

    @Value("${config.info}")
    String info;

    @RequestMapping("/info")
    public String info ( ){
        return info;
    }
}
```

## 修改ConfigCenter

### pom

添加 `spring-cloud-starter-bus-amqp` starter

```xml
<dependencies>
        <!-- 添加了spring-cloud-starter-bus-amqp支持  -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
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

添加rabbitMQ的配置

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
#  -------------------- 以下是添加的内容------------------------
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

management:
  endpoints:
    web:
      exposure:
        #  暴露刷新的端点
        include: 'bus-refresh'

#  -------------------- 以上是添加的内容------------------------
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

## 测试

首先启动 rabbitMQ服务, 然后启动 eureka 和 Config Center ,再启动 1230 和 1231 

访问 `http://localhost:3344/master/app1-dev.yml`   Config Center

![image-20210211125602678](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210211125602678.png)

访问 `http://localhost:1230/info`   1230

![image-20210211125649272](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210211125649272.png)

访问 `http://localhost:1231/info`

![image-20210211125717928](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210211125717928.png)

修改github上的配置信息

![image-20210211125820111](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210211125820111.png)

在访问 `http://localhost:3344/master/app1-dev.yml`   Config Center

![image-20210211125905287](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210211125905287.png)

在访问 `http://localhost:1230/info`   1230

![image-20210211125937039](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210211125937039.png)

可见1230 没有变化,这个时候向 ConfigCenter发送刷新消息,有ConfigCenter再向总线发送刷新消息,这时1230 和 1231 的配置信息应该都会更新

刷新消息 `curl -X POST "http://localhost:3344/actuator/bus-refresh"`

向 3344 配置中心 发送 post 请求 地址为 actuator/bus-refresh ,即为刚才配置暴露的监控端口

![image-20210211130534078](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210211130534078.png)

再次查看 1230

![image-20210211130551807](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210211130551807.png)

1231

![image-20210211130603246](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210211130603246.png)



## 定点刷新配置

实现只通知1230 而不通知1231 ,只实现1230 刷新,而不通知 1231 刷新

`http://localhost:3344/actutor/bus-refresh/{destination}`

## {destination} 

/bus/refresh请求不再发送到具体的服务实力上，而是发给config server通过destination参数指定需要更新配置的服务或实例

destination 为 `微服务名:端口号`

这时 刷新url就为 `curl -X POST "http://localhost:3344/actuator/bus-refresh/cloud-config-client-app1:1230"`

### 测试

修改github上的配置

![image-20210211131336433](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210211131336433.png)

使用 `curl -X POST "http://localhost:3344/actuator/bus-refresh/cloud-config-client-app1:1230"` 进行消息广播

访问 1230

![image-20210211131449396](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210211131449396.png)

访问 1231 

![image-20210211131506659](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210211131506659.png)

可以看到 1231没有更新配置