# openfeign日志控制

我们可以通过日志来记录和查看openfeign的请求细节

## 日志级别

![image-20210203162456106](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210203162456106.png)

## 如何设置日志级别

像容器内注入 `Logger.Level` 

```java
@Bean
    public Logger.Level feignLogger(){
        //向个容器中注入日志级别
        return Logger.Level.FULL;
    }
```

yml配置文件开启日志级别

```yml
logging:
  level:
    org.gushiyu.springcloud.service.PaymentService: debug
```

