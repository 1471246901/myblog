# Spring Security 自定义保护资源

默认情况下,受保护的资源都是默认的 ,只有登录才能查看,而且每种资源的保护程度都是一样的,想要自定义设置就需要重写 `WebSecurityConfigurerAdapter`   的 `configure(httpsecurity http)`  方法

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
                .passwordEncoder(passwordEncoder())
                .withUser("admin").password(passwordEncoder().encode("root")).roles("ADMIN","USER")
                .and()
                .withUser("gu").password(passwordEncoder().encode("root")).roles("USER")
                .and()
                .withUser("db").password(passwordEncoder().encode("root")).roles("ADMIN","DBA","USER");
        //基于内存的认证方式不需要添加前缀 'ROLE_' 和基于数据库的认证方式不一样
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        http.authorizeRequests() //开启httpsecurity 配置 允许基于该限制访问HttpServletRequest使用RequestMatcher的实现（即，经由URL模式）
                .antMatchers("/admin/**").hasRole("ADMIN")
                .antMatchers("/user/**").access("hasAnyRole('ADMIN','USER')")
                .antMatchers("/db/**").access("hasRole('ADMIN') and hasRole('DBA')")
                .anyRequest()  // 表示除了前面配置的url之外,所有的url 都会使用下面的配置
                .authenticated()  //都要验证身份 ,结合上面anyRequest 表示除了上面自定义的规则其余所有的url都需要遵守authenticated 即所有url都要登录
                .and()  // 开启新的配置
                .formLogin()  //开启表单登录
                .loginProcessingUrl("/login")  //设置登录接口为"/login" 设置其的主要目的是方便ajax  且 登录名为username 密码名为password  使用post 提交
                .permitAll()  //登录表单不需要验证 即登录表单可以直接访问,不需要保护
                .and()  // 开启新的配置
                .csrf()  //关闭 csrf
                .disable();  //应用

    }
}
```

 

` configure(AuthenticationManagerBuilder auth)`  方法 配置了三个用户

`configure(HttpSecurity http)` 方法 配置了各自能够访问的资源和登录策略

测试

```java
@RestController
public class TestController {

    @GetMapping("/admin/hello")
    public String admin(){
        return "admin ";
    }

    @GetMapping("/user/hello")
    public String user(){
        return "user ";
    }

    @GetMapping("/dba/hello")
    public String dba(){
        return "dba ";
    }

}
```

请求1 `/hello`  路由到 `/login`

登录  admin 账号后可以访问 `/hello`  ,  `/user/hello`  ,  `/admin/hello`  但是访问 `/db/hello` 显示 403 错误 



>   # There is no PasswordEncoder mapped for the id “null”异常解决办法
>
>   ## 一. 问题描述
>
>   Spring security 5.0中新增了多种加密方式,也改变了默认的密码格式.
>
>   我们来看一下官方文档:
>
>   ```
>   The general format for a password is:
>   {id}encodedPassword
>   Such that id is an identifier used to look up which PasswordEncoder should be used and encodedPassword is the original encoded password for the selected PasswordEncoder. The id must be at the beginning of the password, start with { and end with }. If the id cannot be found, the id will be null. For example, the following might be a list of passwords encoded using different id. All of the original passwords are "password".
>   
>   ```
>
>   **这段话的意思是说,现如今Spring Security中密码的存储格式是“{id}…………”.前面的id是加密方式,id可以是bcrypt、sha256等,后面跟着的是加密后的密码.也就是说,程序拿到传过来的密码的时候,会首先查找被“{”和“}”包括起来的id,来确定后面的密码是被怎么样加密的,如果找不到就认为id是null.**这也就是为什么我们的程序会报错:There is no PasswordEncoder mapped for the id “null”.官方文档举的例子中是各种加密方式针对同一密码加密后的存储形式,原始密码都是“password”.
>
>   ## 二. 解决办法
>
>   需要修改一下configure中的代码,我们要将前端传过来的密码进行某种方式加密,Spring Security 官方推荐的是使用bcrypt加密方式.
>
>   ### 1. 在内存中存取密码的修改方式
>
>   修改后是这样的:
>
>   ```java
>   protected void configure(AuthenticationManagerBuilder auth) throws Exception {
>   
>     //inMemoryAuthentication 从内存中获取
>   
>     auth.inMemoryAuthentication()
>         .passwordEncoder(new BCryptPasswordEncoder())
>         .withUser("user1").password(new BCryptPasswordEncoder().encode("123")).roles("USER");
>   
>   }
>   ```
>
>   inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())",这相当于登陆时用BCrypt加密方式对用户密码进行处理.以前的".password("123")" 变成了 ".password(new BCryptPasswordEncoder().encode("123"))",这相当于对内存中的密码进行Bcrypt编码加密.如果比对时一致,说明密码正确,才允许登陆.
>
>   ### 2. 在数据库中存取密码的修改方式
>
>   如果你用的是在数据库中存储用户名和密码,那么一般是要在用户注册时就使用BCrypt编码将用户密码加密处理后存储在数据库中,并且修改configure()方法,加入".passwordEncoder(new BCryptPasswordEncoder())",保证用户登录时使用bcrypt对密码进行处理再与数据库中的密码比对.如下:
>
>   ```java
>   //注入userDetailsService的实现类
>   
>   auth.userDetailsService(userService).passwordEncoder(new BCryptPasswordEncoder());
>   ```
>
>    