# session共享

spring boot 通过redis提供透明的session共享 

## session共享配置

### 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

### 配置

```yml
spring:
  redis:
    database: 0
    host: 192.168.31.245
    port: 6379
    password: root
    jedis:
      pool:
        max-active: 8
        max-idle: 8
        max-wait: -1ms
        min-idle: 0
```

### 编写测试controller

```java
@RestController
public class TestController {

    @Value("${server.port:8080}")
    String port;

    @RequestMapping("/save")
    public Object save(String name, HttpSession session){
        session.setAttribute("name",name);
        return "save success";
    }

    @RequestMapping("/get")
    public Object get(HttpSession session){
        return session.getAttribute("name");
    }
}
```

分别启动

```powershell
 java -jar .\redissession-0.0.1-SNAPSHOT.jar --server.port=8081
 java -jar .\redissession-0.0.1-SNAPSHOT.jar --server.port=8080
 
```

### 测试

```java
http://localhost:8081/save?name=asdaaaaaaa
http://localhost:8080/get
```

测试获得同一个session

查看redis

![image-20200907155713873](session%E5%85%B1%E4%BA%AB.assets/image-20200907155713873.png)