# SpringCloudConfig 动态刷新配置

每次Config Center 更改配置 client 并不能及时的获知更新,这样让我们更新配置后还需重启服务,Config Center  在git更改之后可以获得更新,但是 client 端不能获得更新,我们利用actuator 的刷新来实现此功能

动态刷新配置需要引入 `actuator` starter 

### 引入 actuator starter

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

### 修改配置文件 

修改client 的 配置文件,暴露监控端点

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
eureka:
  instance:
    instance-id: cloud-config-client-app1-1230
    prefer-ip-address: true
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka7003.com:7003/eureka/
# 暴露监控端点  此为新增
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

### 加入注解 @RefreshScope

controller 中加入注解 `@RefreshScope`   

此注解的作用是 能利用 `http://localhost:1230/actuator/refresh` 请求来刷新微服务

```java
@RestController
@RefreshScope
public class ConfigController {

    @Value("${config.info}")
    String info;

    @@RequestMapping("/info")
    public String info ( ){
        return info;
    }
}

```

### 测试

启动 `cloud-config-client-app1-1230` ,  `cloud-config-center-3344` 和 eureka 服务端 

访问 `localhost:1230/info`

![image-20210209163258329](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210209163258329.png)

修改github 上的 app1-dev 文件,将 version 改为 2

![image-20210209163403221](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210209163403221.png)

访问 `http://localhost:3344/master/app1-dev.yml`

![image-20210209163537811](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210209163537811.png)

使用 post 请求访问 `http://localhost:1230/actuator/refresh`  来刷新

![image-20210209163915483](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210209163915483.png)

再次访问 `localhost:1230/info`

![image-20210209163952281](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210209163952281.png)

配置生效