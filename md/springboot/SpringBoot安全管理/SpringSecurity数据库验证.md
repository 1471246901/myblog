# SpringSecurity数据库验证

真实情况下用户信息都在存在于数据库中,所以需要基于数据库进行验证

## 搭建数据库

### 设计用户角色表(以后还需要权限表)

![image-20200923083542757](SpringSecurity%E6%95%B0%E6%8D%AE%E5%BA%93%E9%AA%8C%E8%AF%81.assets/image-20200923083542757.png)

### 插入数据

![image-20200923084455035](SpringSecurity%E6%95%B0%E6%8D%AE%E5%BA%93%E9%AA%8C%E8%AF%81.assets/image-20200923084455035.png)

## 创建项目

### maven依赖

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
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.10</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
```

 使用mybatis 作为持久层框架

## 配置

### 配置数据库

```yml
spring:
  
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    username: root
    password: root
    url: jdbc:mysql://localhost:3306/user?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC
    #  高版本 使用 com.mysql.cj.jdbc.Driver
    driver-class-name: com.mysql.cj.jdbc.Driver
  thymeleaf:
    cache: false
    encoding: utf-8
    prefix: classpath:/templates/
    suffix: .html
    servlet:
      content-type: text/html
    mode: LEGACYHTML5
```

### 创建实体类

```java
public class Role {
    private Integer id;
    private String name;
    private String nameZh;
    //get and set
}


public class User implements UserDetails {
    private Integer id;
    private String username;
    private String password;
    private Boolean enable;
    private Boolean locked;

  	//储存角色信息
    private List<Role> roles;


    /**
     * 获取当前对象的所有角色信息
     * @return Collection
     */
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        List<SimpleGrantedAuthority> authorities = new ArrayList<>();
        for (Role role:roles){
            authorities.add(new SimpleGrantedAuthority(role.getName()));
        }
        return authorities;
    }

    /**
     * 获取密码
     * @return password
     */
    @Override
    public String getPassword() {
        return password;
    }

    /**
     * 获取用户名
     * @return username
     */
    @Override
    public String getUsername() {
        return username;
    }

    /**
     * 当前账户是否为过期
     * @return expired state
     */
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    /**
     * 当前账户是否被锁定
     * @return !locked
     */
    @Override
    public boolean isAccountNonLocked() {
        return !locked;
    }

    /**
     * 当前账户密码是否为过期
     * @return true CredentialsNonExpired
     */
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    /**
     * 账户是否可用
     * @return  enable
     */
    @Override
    public boolean isEnabled() {
        return enable;
    }

    //省略get 和 set
}

```

用户需要实现`UserDetails` 接口接口定义了7个方法,需要用户来实现,并不需要用户自己比对角色等信息,开发者只需要提供即可 ,在比对不匹配时会抛出相应异常

`getAuthorities` 方法 用来获取用户所具有的角色信息,在本案例中角色信息存储在roles中所以便利字段返回即可

### 创建mapper 和 mapper.xml并配置

```yml
mybatis:
  mapper-locations: classpath:mapping/*Mapper.xml
  type-aliases-package: org.gushiyu.security.entity
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace：命名空间，做sql隔离 -->
<mapper namespace="org.gushiyu.security.mapper.UserMapper">

    <select id="loadUserByUsername" resultType="org.gushiyu.security.entity.User">
        select id,username,password,enabled,locked from user where username = #{username,jdbcType=VARCHAR}
    </select>

    <select id="getUserRolesByUid" resultType="org.gushiyu.security.entity.Role">

        select id,name,nameZh from role r right join user_role ur on r.id = ur.rid where ur.uid = #{id,jdbcType=INTEGER}
    </select>
</mapper>
```

```java
@Mapper
public interface UserMapper {
    User loadUserByUsername(String username);
    List<Role> getUserRolesByUid(Integer id);
}
```

>   在整合 mybatis 和 SpringSecurity 的时候 实体类要继承userdetails  所以需要重写isEnabled() 方法
>
>   如果数据库字段也是 enabled 在 就会有 getEnabled() 方法 所以在登录的时候 mybatis 会报错不符合java bean 的规范  此时可以更改字段 或者 删除 getEnabled() 方法

### 创建service  `UserDetailsService`

```java
@Service
public class UserService implements UserDetailsService {

    @Autowired
    UserMapper userMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userMapper.loadUserByUsername(username);

        if(user == null){
            throw new UsernameNotFoundException("账户不存在");
        }
        user.setRoles(userMapper.getUserRolesByUid(user.getId()));
        return user;
    }
} 
```

用户实现 `UserDetailsService`  并重写  `loadUserByUsername ` 方法

该方法的参数就是用户登录时输入的用户名,具体比对工作有SpringSecurity 进行,所以  `loadUserByUsername` 方法在用户登录时自动调用

### 配置SpringSecurity

本文 重点看  `UserService ` 注入和  `configure(AuthenticationManagerBuilder auth)` 方法

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {


    //在这里注入 userService
    @Autowired
    UserService userService;

    @Bean
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }


    /**
     *  对用户进行认证
     * @param auth  设置用户认证的服务
     * @throws Exception
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {

        auth.userDetailsService(userService);
    }


    @Override
    public void configure(WebSecurity web) throws Exception {
        super.configure(web);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {



        http.authorizeRequests() //开启httpsecurity 配置 允许基于该限制访问HttpServletRequest使用RequestMatcher的实现（即，经由URL模式）
                .antMatchers("/admin/**").hasRole("ADMIN")
                .antMatchers("/user/**").access("hasAnyRole('ADMIN','USER','DBA')")
                .antMatchers("/db/**").access("hasRole('DBA')")
                .anyRequest()  // 表示除了前面配置的url之外,所有的url 都会使用下面的配置
                .authenticated()  //都要验证身份 ,结合上面anyRequest 表示除了上面自定义的规则其余所有的url都需要遵守authenticated 即所有url都要登录
                .and()  // 开启新的配置
                .formLogin()  //开启表单登录
                .loginPage("/loginpage")   //登陆页面   ,如果使用纯json通信方式,改成 loginProcessingUrl() 相同的地址
                .successForwardUrl("/hello")  //登陆成功跳转
                .loginProcessingUrl("/login")  //设置登录接口为"/login" 设置其的主要目的是方便ajax  且 登录名为username 密码名为password  使用post 提交
                .usernameParameter("username") //配置登录账户名对应的参数名
                .passwordParameter("password")  //配置登录账户密码对应的参数名
                .successHandler(new AuthenticationSuccessHandler() {   //登录成功后的处理
                    @Override
                    public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
                        Object principal = authentication.getPrincipal();   //获取认证信息
                        httpServletResponse.setContentType("application/json;charset=utf-8");   //设置返回格式
                        httpServletResponse.setStatus(200);  //设置状态
                        Result<Object> result = Result.setResult(ResultCodeEnum.LOGIN_SUCCESS);  //获取成功模板
                        result.data("login_data",principal);  //放入登录信息
                        ObjectMapper om = new ObjectMapper();  //jackson的json库
                        PrintWriter writer = httpServletResponse.getWriter();   //获取返回输出流
                        writer.write(om.writeValueAsString(result));  //输出
                        writer.flush();
                        writer.close();
                    }
                })
                .failureHandler(new AuthenticationFailureHandler() {
                    @Override
                    public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
                        httpServletResponse.setContentType("application/json;charset=utf-8");   //设置返回格式
                        httpServletResponse.setStatus(401);  //设置状态
                        Result<Object> result = Result.setResult(ResultCodeEnum.LOGIN_FAIL);  //获取成功模板
                        String key = "detail";
                        if(e instanceof LockedException){
                            result.data(key,"账户被锁定");
                        } else if (e instanceof BadCredentialsException){
                            result.data(key,"账户名或者密码输入错误");
                        } else if (e instanceof DisabledException){
                            result.data(key,"账户被禁用");
                        } else if (e instanceof AccountExpiredException){
                            result.data(key,"账户已过期");
                        } else if(e instanceof CredentialsExpiredException){
                            result.data(key,"密码已过期");
                        }else {
                            result.data(key,"登陆失败");
                        }

                        ObjectMapper om = new ObjectMapper();  //jackson的json库
                        PrintWriter writer = httpServletResponse.getWriter();   //获取返回输出流
                        writer.write(om.writeValueAsString(result));  //输出
                        writer.flush();
                        writer.close();

                    }
                })
                .permitAll()  //登录表单不需要验证 即登录表单可以直接访问,不需要保护
                .and()  // 开启新的配置
                .logout()  //注销登录配置
                .logoutUrl("/logout")  //注销url 默认也是logout
                .clearAuthentication(true)  //是否清除认证信息
                .invalidateHttpSession(true)  //表示是否使session失效
                .addLogoutHandler(new LogoutHandler() {   //不可以抛出异常,抛出异常后注销失败
                    @Override
                    public void logout(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) {
                        //在logout 中用户可以实现一些自己的数据清除操作
                        //而且SpringSecurity 中提供了几个默认的实现
                        // 1.  CookieClearingLogoutHandler  清除cookie
                        // 2.  CsrfLogoutHandler   移除CSRF TOKEN
                        // 3.  CompositeLogoutHandler
                        System.out.println("注销操作");
                    }
                })
                .logoutSuccessHandler(new LogoutSuccessHandler() {   //可以抛出异常
                    @Override
                    public void onLogoutSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
                         //这里处理注销成功后的逻辑,可以像登录成功后那样返回一段json ,或者提示跳转到某个页面
                        httpServletResponse.setContentType("application/json;charset=utf-8");   //设置返回格式
                        httpServletResponse.setStatus(200);  //设置状态
                        System.out.println("处理注销成功后的逻辑");
                        Result<Object> result = Result.setResult(ResultCodeEnum.LOGOUT_SUCCESS);
                        ObjectMapper om = new ObjectMapper();  //jackson的json库
                        PrintWriter writer = httpServletResponse.getWriter();   //获取返回输出流
                        writer.write(om.writeValueAsString(result));  //输出
                        writer.flush();
                        writer.close();
                    }
                })

                .and()
                .csrf()  //关闭 csrf
                .disable();  //应用

    }
}

```

## 测试

登录 admin root  

```json
{"code":200,"message":"登录成功","success":true,"data":{"login_data":{"id":2,"username":"admin","password":"$2a$10$Yb.f4/BVBfmx2XYu8Mt9a.48yManDzb.yUZbWQ7PuVF5nujmpWN0a","enabled":true,"locked":false,"roles":[{"id":2,"name":"ROLE_ADMIN","nameZh":"系统管理员"}],"authorities":[{"authority":"ROLE_ADMIN"}],"credentialsNonExpired":true,"accountNonExpired":true,"accountNonLocked":true}}}
```

