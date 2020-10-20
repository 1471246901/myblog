# SpringSecurity基本配置

[官网  https://spring.io/projects/spring-security](https://spring.io/projects/spring-security)

[文档 https://docs.spring.io/spring-security/site/docs/5.4.0-M1/reference/html5/#jc](https://docs.spring.io/spring-security/site/docs/5.4.0-M1/reference/html5/#jc)

Spring Security 是一个功能强大且高度可自定义的身份验证和访问控制框架。它是保护基于 Spring 的应用的基础。

Spring Security 是一个框架，侧重于向 Java 应用程序提供身份验证和授权。与所有spring项目一样，spring Security 的真正强大之处存在于如何轻松地扩展以满足自定义需求

以往实现这些功能    需要拦截器,过滤器  



Spring boot 中提供了SpringSecurity 的自动配置,所以 Springboot 中使用SpringSecurity  特别方便

## 基本使用

### 配置

maven依赖

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
```

只要开发者在项目中添加了  `spring-boot-starter-security` 场景 项目中的所有资源就会被保护起来

>   thymeleaf 配置
>
>   ```yml
>   spring:
>     thymeleaf:
>       cache: false
>       encoding: utf-8
>       prefix: classpath://templates
>       suffix: .html
>       servlet:
>         content-type: text/html
>   ```
>
>    并在  `templates` 下存在 hello.html

### 使用 

创建Controller 

```java
@Controller
public class HelloController {
    
    @GetMapping("/hello")
    public ModelAndView hello(ModelAndView modelAndView){
        modelAndView.setViewName("hello");
        return modelAndView;
    }
}
```

访问 ![image-20200916092600338](SpringSecurity%E5%9F%BA%E6%9C%AC.assets/image-20200916092600338.png)

![image-20200916092611261](SpringSecurity%E5%9F%BA%E6%9C%AC.assets/image-20200916092611261.png)

用户名默认是user ,登陆密码每次启动项目是随机生成,再看控制台查看 登陆后就可以正常访问资源



## 配置用户名和密码

用户可以在application.properties 中配置默认的用户名和密码

```yml
  security:
    user:
      name: gushiyu
      password: root
      roles: admin
```

## 基于内存的认证

开发者也可以自定义继承自 `WebSecurityConfigurerAdapter`  对Spring Security 进行更多自定义的配置



```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {



    /**
     *  对用户进行认证
     * @param auth  允许容易地建立在内存认证，LDAP认证，基于JDBC认证，加入UserDetailsService ，以及添加AuthenticationProvider的
     * @throws Exception
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            	// spring security 引入了多种密码加密方式开发者必须指定一种
            	.passwordEncoder(new BCryptPasswordEncoder())
                .withUser("admin").password("root").roles("ADMIN","USER")
                .and()
                .withUser("gu").password("root").roles("USER");
        //基于内存的认证方式不需要添加前缀 'ROLE_' 和基于数据库的认证方式不一样
    }
```

