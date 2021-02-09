# 断言和过滤器

## 断言

常用的断言

![image-20210206154925411](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210206154925411.png)

也就是yml文件中断言配置的

predicates:
            -` Path`=/test/** # 断言，路径相匹配的进行路由

path 指明使用那个类型的断言

### 断言种类

| 断言                          | 作用                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| 1.After Route Predicate       | 在时间之后 才能路由                                          |
| 2.Before Route Predicate      | 在时间之前 才能路由                                          |
| 3.Between Route Predicate     | 在时间区间内才能路由                                         |
| 4.Cookie Route Predicate      | 路由断言工厂会取两个参数 - cookie 名称对应的的key 和value，当cookie 和 Cookied 断言工厂配置的cookie 一致则匹配成功 |
| 5.Header Route Predicate      | 根据header 配置 ，如果携带的header 和 断言工厂的一致则匹配成功，否则匹配失败 |
| 6.Host Route Predicate        | 对请求中Host 进行断言，如果成功则 路由                       |
| 7.Method Route Predicate      | 对请求的方法 进行配置，如果成功则路由。如 Post,Get           |
| 8.Path Route Predicate        | 对请求的路径进行匹配                                         |
| 9.Query Route Predicate       | 对请求参数进行断言，如果成功则路由。如 localhost:80/id?foo:bb |
| 10.RemoteAddr Route Predicate | 对网段字符串或者ip进行断言 ，成功则路由。                    |
| 11.Weight Route Predicate     | 基于路由权重                                                 |

### 示例

**官网示例**

https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-request-predicates-factories

#### after

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: before_route
        uri: https://example.org
        predicates:
        - Before=2017-01-20T17:42:47.789-07:00[America/Denver]  # 时间的生成可以使用ZoneDateTime
```

#### Between

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: between_route
        uri: https://example.org
        predicates:
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```

#### Cookie

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: https://example.org
        predicates:
        - Cookie=chocolate, ch.p   # chocolate 名  ch.p 正则表达式 ,匹配上则通过
```

#### Header

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+  # 请求头有 X-Request-Id 且 值为  \d+ 正则表达式
```

.......

## 过滤器

过滤器类似servlet 的过滤器

![image-20210206160608077](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210206160608077.png)

### 生命周期

pre 前置

post 后置

### 种类 

查看 https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories

GatewayFilter  

GlobalFilter

**示例**

The `AddRequestParameter GatewayFilter` Factory

https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-addrequestparameter-gatewayfilter-factory

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_parameter_route
        uri: https://example.org
        filters:
        - AddRequestParameter=red, blue
```

匹配的routes 请求会被在请求头上 加上一对 red:blue 的值



## 自定义过滤器

我们在做鉴权,统一日志等都需要使用自定义过滤器

### 配置方法

创建类 实现 `GlobalFilter` , `Ordered ` 两个接口 ,`GlobalFilter`  接口表示过滤器,`Ordered ` 接口表示多个过滤器的先后顺序,小的优先

```java
@Component
public class filters implements GlobalFilter, Ordered {

    //如果没有uname参数,则不放行
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        System.out.println("come in global filter: {}"+new Date());
        ServerHttpRequest request = exchange.getRequest();
        String uname = request.getQueryParams().getFirst("uname");
        if (uname == null) {
            System.out.println("用户名为null，非法用户");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        // 放行
        return chain.filter(exchange);

    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

### 启动运行

访问 `http://localhost:9527/test/0` 

![image-20210206162233572](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210206162233572.png)

携带 uname 参数

![image-20210206162306701](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210206162306701.png)



