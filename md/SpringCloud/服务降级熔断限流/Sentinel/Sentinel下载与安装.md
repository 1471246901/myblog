# Sentinel下载与安装

![image-20210221172400713](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210221172400713.png)

## 控制台

### 下载地址

 https://github.com/alibaba/Sentinel/releases  

![image-20210221172712488](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210221172712488.png)

### 安装 

前提  sentinel-dashboard-1.8.1 是需要占用8080 端口的,所以8080 端口不能被占用

使用 `java -jar sentinel-dashboard-1.8.1.jar`

![image-20210221172852409](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210221172852409.png)

此时访问 localhost:8080  账号密码默认 sentinel

![image-20210221173013178](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210221173013178.png)

代表安装成功

## 核心库

修改微服务模块

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
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-nacos-config</artifactId>
        </dependency>
        <!--<dependency>    sentinel datasource-nacos持久化会用到
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>-->
        <!-- sentinel 依赖 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <!-- 可以使用openfeign  -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
    </dependencies>
```

### yml

这里只列出参考配置

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
        file-extension: yaml # 指定yaml作为配置文件的格式
        namespace: # 填写namespace
        group: #  填写不同的分组
# sentinel 的配置======================================
    sentinel:
      transport:
        # 监控仪表盘地址和端口
        dashboard: localhost:8080
        # 与监控仪表盘沟通的端口,默认8179 ,如被占用则会＋1扫描
        port: 8179
# sentinel 的配置======================================
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

启动类无改变

### 业务类

随意编写几个controller

```java
@GetMapping("/test/a")
    public String testA(){
        return "testA "+ UUID.randomUUID().toString();
    }

    @GetMapping("/test/b")
    public String testB(){
        return "testA "+ UUID.randomUUID().toString();
    }
```

## 测试

启动 Nacos  启动 Sentinel 启动 微服务

这时需要访问刚刚创建的controller 才能让sentinel知道有这样的一个请求 `/test/a`

和 `test/b`

这时登录sentinel查看

![image-20210221180004400](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210221180004400.png)

此后在web界面即可配置sentinel的相关规则

