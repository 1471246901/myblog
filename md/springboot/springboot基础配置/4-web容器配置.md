# web容器配置

springboot 可以内置Tomcat , Jetty , Undertow ,Netty 等容器 .我们可以根据应用的不同来选择适合的容器

## Tomcat配置

spring boot 默认会使用tomcat 作为容器,在使用tomcat 时 ,只需要引入web模块即可

tomcat 常用配置

```properties
# web容器的端口号
server.port=8080
# 项目出错的跳转页面   
server.error.path= /error
# session 失效时间  m为分   不写单位默认为秒 
# 如果是秒的话会转化为 最大不超过该秒数的分钟数
server.servlet.session.timeout= 30m  
# 项目名称 不写默认为 \
server.servlet.context-path=/name
# tomcat 的请求编码
server.tomcat.uri-encoding=utf-8
# 最大线程数   不同的容器使用不同的 第二名称
server.tomcat.threads.max= 500
# tomcat运行日志和历史文件的存放目录
server.tomcat.basedir=/home/sang/tmp

```

完整的配置可以参考[application-properties](https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html)

## jetty配置

jetty 是一个轻量级的servlet 容器,配置如下

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
            	<exclusion>
            		<groupId>org.springframework.boot</groupId>
            		<artifactId>spring-boot-starter-tomcat</artifactId>
            	</exclusion>
            </exclusions>
</dependency>
        
<dependency>
        	<groupId>org.springframework.boot</groupId>
        	<artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

在starter中排除tomcat 然后添加jetty的starter即可

![image-20200727161048845](4-web%E5%AE%B9%E5%99%A8%E9%85%8D%E7%BD%AE.assets/image-20200727161048845.png)

## Undertow配置

Undertow 公司的java服务器,性能很好,配置方式类似jetty

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
            	<exclusion>
            		<groupId>org.springframework.boot</groupId>
            		<artifactId>spring-boot-starter-tomcat</artifactId>
            	</exclusion>
            </exclusions>
        </dependency>
        
        <dependency>
        	<groupId>org.springframework.boot</groupId>
        	<artifactId>spring-boot-starter-undertow</artifactId>
        </dependency>
```

在starter中排除tomcat 然后添加Undertow的starter即可

![image-20200727161400299](4-web%E5%AE%B9%E5%99%A8%E9%85%8D%E7%BD%AE.assets/image-20200727161400299.png)