# OAuth2认证

OAuth（开放授权）是一个开放标准，允许用户授权第三方移动应用访问他们存储在另外的服务提供者上的信息，而不需要将用户名和密码提供给第三方移动应用或分享他们数据的所有内容，OAuth2.0是OAuth协议的延续版本，但不向后兼容OAuth 1.0即完全废止了OAuth1.0

## 应用场景

**第三方应用授权登录**：在APP或者网页接入一些第三方应用时，时长会需要用户登录另一个合作平台，比如QQ，微博，微信的授权登录。

![image-20200925152717634](OAuth2%E8%AE%A4%E8%AF%81.assets/image-20200925152717634.png)

**原生app授权**：app登录请求后台接口，为了安全认证，所有请求都带token信息，如果登录验证、请求后台数据。

**前后端分离单页面应用（spa）**：前后端分离框架，前端请求后台数据，需要进行oauth2安全认证，比如使用vue、react后者h5开发的app。

传统的web开发登录都是基于session的,但是在前后端分离的架构中继续使用session会有很多不便,因为移动端app,或者小程序,都是不支持Cookie,或者使用不方便,这样的情况下使用OAuth2都能解决

## OAuth2角色

资源所有者: 资源所有者即用户,持有资源的一方,拥有头像战片,视频资源  表示用户

客户端: 客户端及第三方应用,比如豆瓣向qq请求授权服务

授权服务器: 授权服务器用来验证用户提供的登录信息是否正确,如果正确就会下发一个令牌给第三方应用使用

资源服务器: 资源服务器是提供给用户(资源所有者)资源的服务器

一般来说资源服务器和授权服务器可以是同一台服务器

## 授权模式

授权模式共分为4种

### 授权码模式   适用第三方登录等

![image-20200925152717634](OAuth2%E8%AE%A4%E8%AF%81.assets/image-20200925152717634.png)

授权码模式是最功能完整,流程严谨的授权模式,特点就是通过客户端的服务器与授权服务器进行交互,国内最常见的第三方平台就是使用此模式

图中  QQ授权服务  的角色是授权服务器,资源服务器, 获取到令牌后豆瓣网站(客户端)向QQ发起资源请求,( 用户浏览器和豆瓣网站相当于客户端)



### 简化模式   适用无服务器客户端

此模式不需要客户端服务器的参与(如果网站是纯静态网站) 可以采用此模式

![image-20200925155411009](OAuth2%E8%AE%A4%E8%AF%81.assets/image-20200925155411009.png)

### 密码模式  适用同一家公司或者合作公司之间的授权

用户把用户名密码告诉客户端,客户端使用这些信息向授权服务器申请令牌,这需要客户端的高度信任,但是如果客户端和服务提供商是一家公司就可以



### 客户端模式    适合非常信任的第三方无状态的服务

或者叫凭证模式 , 非常的容易理解

当对于我们针对一个非常信任的第三方去登陆时 , 可以采用这种模式 .

**首先要提供给第三方一个client_id 和 client_secret , 相当于公用的用户名密码 , 第三方拿着这俩东西去换取令牌 , 拿着令牌去取数据就可以了 .** 

**适合无状态的服务 ,因为客户端使用的是公用的用户名和密码**

基本只需要提供两个接口 , 一个是获取令牌 , 一个是获取数据

GET /token?

grant_type=client_credentials &

client_id = {客户端身份ID} & 相当于用户名

client_secret = {客户端秘钥} & 相当于密码

我们后端拿着这个用户名密码进行比对 , 比对成功 ,返回access_token

**第三方拿着access_token 获取数据**

GET /user?

access_token = {令牌} &

uid = {用户ID}

后端验证access_token 存在 , 并且没有过期 , 就验证通过 , 返回数据







## SpringSecurity 基础上搭建OAuth2 服务

### maven依赖

OAuth2是在SpringSecurity 基础上实现的,因此要用到Security依赖,而且使用redis存储令牌,则需要使用Redis

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.security.oauth</groupId>
            <artifactId>spring-security-oauth2</artifactId>
            <version>2.3.4.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

### 配置Redis

本次使用本地Redis  

```yml
spring:
  redis:
    database: 0
    host: 127.0.0.1
    port: 6379
    password: root
    jedis:
      pool:
        max-active: 8
        max-idle: 8
        max-wait: -1ms
        min-idle: 0
```

### 配置授权服务器

授权服务器和资源服务器可以是同一台服务器,也可以是不同的服务器

`@EnableAuthorizationServer` 表示开启授权服务器

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig  extends AuthorizationServerConfigurerAdapter {


    @Autowired
    AuthenticationManager authenticationManager;

    @Autowired
    RedisConnectionFactory redisConnectionFactory;

    @Autowired
    UserDetailsService userDetailsService;

    @Bean
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder(10);
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.allowFormAuthenticationForClients();  //表示支持Client_id 和 client_secret做登录认证
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("password")
                .authorizedGrantTypes("password","refresh_token")   //password的认证模式
                .accessTokenValiditySeconds(1800)   //过期时间
                .resourceIds("rid")   //资源id
                .scopes("all")  //作用域，用来限制客户端权限访问的范围
                .secret(passwordEncoder().encode("root"));  //配置密码
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {

        endpoints.tokenStore(new RedisTokenStore(redisConnectionFactory))  //设置令牌存储方式
                .authenticationManager(authenticationManager)  
                .userDetailsService(userDetailsService);
    }
}
```



### 配置资源服务器

`@EnableResourceServer` 开启资源服务

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources.resourceId("rid").stateless(true);  //设置资源ID(需要和授权服务器的id保持一致),然后设置这些资源仅基于令牌认证

    }

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/admin/**").hasRole("ADMIN")
                .antMatchers("/user/**").hasRole("USER")
                .anyRequest().authenticated();
    }
}
```

### 配置Spring Security

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    /**
    在这里分别配置了两个用户
    */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("admin")
                .password(passwordEncoder().encode("root"))
                .roles("ADMIN")
                .and()
                .withUser("user")
                .password(passwordEncoder().encode("root"))
                .roles("USER");
    }

    @Bean
    PasswordEncoder passwordEncoder(){
        return  new BCryptPasswordEncoder(10);
    }


    @Bean
    @Override
    protected AuthenticationManager authenticationManager() throws Exception {
        return super.authenticationManager();
    }

    @Bean
    @Override
    protected UserDetailsService userDetailsService() {
        return super.userDetailsService();
    }

    /**
    Spring Security 配置高于资源服务器中的配置
    */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.antMatcher("/oauth/**").authorizeRequests()
                .antMatchers("/oauth/**").permitAll()
                .and().csrf().disable();
    }
}
```

### 测试验证





#### 编写测试

```java
@RestController
public class HelloController {

    @RequestMapping("/admin/hello")
    public String admin(){
        return "admin";
    }

    @RequestMapping("/user/hello")
    public String user(){
        return "user";
    }

    @RequestMapping("/hello")
    public String hello(){
        return "hello";
    }


}
```



#### 请求授权

![image-20200930122944617](OAuth2%E8%AE%A4%E8%AF%81.assets/image-20200930122944617.png)

返回的数据

```json
{
	"access_token": "1b77b1e0-54bb-4194-a21e-e4911d6371f3",
	"token_type": "bearer",
	"refresh_token": "3ba7b62d-ad16-4a70-9a0f-320e6b8f202f",
	"expires_in": 1799,
	"scope": "all"
}
```

`access_token`  获取资源时要使用的令牌

`refresh_token`  刷新令牌时用的令牌



#### 刷新令牌

![image-20200930124002075](OAuth2%E8%AE%A4%E8%AF%81.assets/image-20200930124002075.png)

#### 请求资源

请求user下的资源 

![image-20200930124508705](OAuth2%E8%AE%A4%E8%AF%81.assets/image-20200930124508705.png)

请求admin下的资源

![image-20200930124547580](OAuth2%E8%AE%A4%E8%AF%81.assets/image-20200930124547580.png)

### redis中的数据

![image-20200930124847368](OAuth2%E8%AE%A4%E8%AF%81.assets/image-20200930124847368.png)



