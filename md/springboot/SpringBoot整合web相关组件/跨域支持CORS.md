# 跨域支持CORS

https://www.cnblogs.com/wq-code/p/11439098.html

https://blog.csdn.net/rongtaoup/article/details/89497471

**当我们在一个域名为 http://IP:PORT/xxx.html 请求异于http ,IP ,PORT 地址的资源的时候就会涉及到跨域**

在java 中跨域在前端的解决方案有JSONP,但是JSONP只支持GET请求,所以我们使用CORS解决跨域请求

## CORS

CORS跨域资源共享是一种机制，它使用额外的 HTTP头来告诉浏览器 让运行在一个 origin上的Web应用被准许访问来自不同源服务器上的指定的资源。

### get,post,head请求

浏览器发起请求

```
Host:localhost:8080
Origin:http://localhost:8081
Referer:http://localhost:8081/index.html
```

表示当前域名`http://localhost:8081` ,想要跨域访问`localhost:8080` ,当前域名地址 `http://localhost:8081/index.html`

此时 服务器如果支持CORS,则服务器给出的响应信息如下:

```
Access-Control-Allow-Origin:http://localhost:8081
Content-Length:20
......
```

Access-Control-Allow-Origin 字段用于记录可以访问该资源的域,当浏览器收到带有 该字段的响应头之后,就知道跨域是允许的,所以就不会对跨域进行限制

### delete等其他请求

浏览器发送options请求

向服务端询问是否具备该资源的DELETE资源,当前浏览器域名为`http://localhost:8081`,跨域访问`localhost:8080` 的 DELETE

```
Access-Control-Request-Method DELETE
Connection keep-alive
Host localhost:8080
Origin:http://localhost:8081
```

浏览器支持CORS 的情况会响应

```
HTTP/1.1 200
Access-Control-Allow-Origin:http://localhost:8081
Access-Control-Allow-Method:DELETE
Access-Control-Max-Age: 1800
Allow: GET,  HTAD, POST, PUT , DELETE , OPTIONS , PATCH 
Content-Length:0 
.......
```

Allow 表示服务器支持的请求方法,Access-Control-Allow-Origin表示允许来自  http://localhost:8081 的域访问信息,Access-Control-Max-Age用来指定本次预检请求的有效期(秒)

此后浏览器发送跨域访问

```
Host:localhost:8080
Origin:http://localhost:8081
```

服务端响应

```
HTTP/1.1 200
Access-Control-Allow-Origin:http://localhost:8081
.....
```

所以重点就是来自服务器返回的`Access-Control-Allow-Origin` 字段表示允许那个域访问自己

## SpringBoot 配置 CORS

### @CrossOrigin 注解

@CrossOrigin(value = "http://localhost:8081",maxAge = 1800,allowedHeaders = "*")

`value `支持跨域域名,`maxage `请求有效期,`allowedHeaders `允许的请求头

可以将此注解标注到controller 的请求方法上,对某个方法进行精细控制

### WebMvcConfigurer 全局配置

此前WebMvcConfigurer 接口用于mvc相关配置其中  `addCorsMappings` 方法就是对跨域相关进行配置

```java
@Override
public void addCorsMappings(CorsRegistry registry) {
    super.addCorsMappings(registry);
    registry.addMapping("/cors/**")  //映射的路径
            .allowedHeaders("*")  //允许的请求头
            .allowedMethods("POST","GET") // 支持的跨域请求类型
            .allowedOrigins("*");   //支持跨域域名
}
```

### 传统过滤器配置  用于非springboot

```java
/**
 * 跨域请求拦截器
 */
@Component
public class CorsFilter implements Filter {
 
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        /* 自己的代码 */
        System.out.println("CorsFilter init...");
    }
 
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("CorsFilter doFilter...");
        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse res = (HttpServletResponse) response;
        //允许请求携带认证信息(cookie)
        res.setHeader("Access-Control-Allow-Credentials", "true");
        //指定允许其他域名访问
        res.setHeader("Access-Control-Allow-Origin", req.getHeader("Origin"));
        //允许请求的类型
        res.setHeader("Access-Control-Allow-Methods", "GET, HEAD, POST, PUT, DELETE, TRACE, OPTIONS, PATCH");
        //允许的请求头字段
        res.setHeader("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
        //设置预检请求的有效期
        //浏览器同源策略：出于安全考虑，浏览器限制跨域的http请求。怎样限制呢？通过发送两次请求：预检请求、用户请求。
        //1、预检请求作用：获知服务器是否允许该跨域请求：如果允许，才发起第二次真实的请求；如果不允许，则拦截第二次请求
        //2、单位:s,在此期间不用发送预检请求。
        //3、若为0：表示每次请求都发送预检请求,每个ajax请求之前都会先发送预检请求。
        res.setHeader("Access-Control-Max-Age", "3600");
        //OPTIONS Method表示浏览器发送的预检请求。
        if ("OPTIONS".equalsIgnoreCase(req.getMethod())) {
            res.setStatus(HttpServletResponse.SC_OK);
        } else {
            /* 自己的代码 */
            chain.doFilter(req, res);
        }
    }
 
    @Override
    public void destroy() {
        /* 自己的代码 */
        System.out.println("CorsFilter destroy...");
    }
}
```

## 解决跨域session共享

ajax请求中，加入**xhrFields:{withCredentials: true}**，表示携带cookie信息。

**非常重要：必须设置参数，否则每次跨域调用接口，都会新建一个session信息，相当于每次调用都是不同用户访问，是不合理，更是不安全的设计。**

```javascript
$.ajax({
        url:"http://127.0.0.1:8080/cors-server/login",
        type:"post",
        dataType:"json",
        xhrFields: {  //携带cookie信息就可以避免跨域是session丢失
            withCredentials: true
        },
        success:function(json){
            console.log(json);
        }
    });
```

