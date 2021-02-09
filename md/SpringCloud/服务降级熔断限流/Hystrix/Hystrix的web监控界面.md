# Hystrix 实现图形化监控

![image-20210205144949862](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210205144949862.png)

## 首先我们需要创建监控客户端

创建模块`cloud-provider-hystrix-dashboard9001`  

### pom

修改pom

```xml
<description>hystrix监控</description>


    <dependencies>
        <!--hystrix dashboard-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
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
    </dependencies>
```

### yml

```yml
server:
  port: 9001
```

### 启动类

```java
@SpringBootApplication
@EnableHystrixDashboard
public class HystrixDashboardMain9001 {
    public static void main(String[] args) {
            SpringApplication.run(HystrixDashboardMain9001.class, args);
    }
}
```

## 被监控的服务

被监控的服务要引入 actuator starter ，

```xml
<!--监控-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

## 测试

启动两个项目和eireka 查看

访问 `http://localhost:9001/hystrix`

![image-20210205150059421](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210205150059421.png)

填写监控地址  `http://localhost:8003/hystrix.stream`

如果出现 `Unable to connect to Command Metric Stream` 错误，需要在8003 被监控的服务上指定监控路径

8003 启动类  添加 `getServlet` 方法指定监控地址

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker
public class PaymentHystrixMain8003 {
    public static void main(String[] args) {
            SpringApplication.run(PaymentHystrixMain8003.class, args);
    }

    /**
     * 此配置是为了服务监控而配置，与服务容错本身无观，springCloud 升级之后的坑
     * ServletRegistrationBean因为springboot的默认路径不是/hystrix.stream
     * 只要在自己的项目中配置上下面的servlet即可
     * @return
     */
    @Bean
    public ServletRegistrationBean getServlet(){
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean<HystrixMetricsStreamServlet> registrationBean = new ServletRegistrationBean<>(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
}
```

进入到监控界面

![image-20210205151102391](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210205151102391.png)

![image-20210205151131850](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210205151131850.png)

![image-20210205151139264](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210205151139264.png)

监控之后可以方便的观察某个服务的流量情况