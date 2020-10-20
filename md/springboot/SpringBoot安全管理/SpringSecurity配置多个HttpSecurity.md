# SpringSecurity 配置多个HttpSecurity 或者 WebSecurity 和 AuthenticationManagerBuilder

代码

```java
@Configuration
public class WebSecurityConfig{



    @Bean
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }



    @Configuration
    @Order(1)   //数字越小优先级越大
    public static class AdminSecurityConfig extends WebSecurityConfigurerAdapter {

        @Override
        protected void configure(HttpSecurity http) throws Exception {

            //在这里配置admin的http认证
        }
    }

    @Configuration
    @Order(2)
    public static class UserSecurityConfig extends WebSecurityConfigurerAdapter {

        @Override
        protected void configure(HttpSecurity http) throws Exception {

            //在这里配置user的http认证
        }
    }
```

`WebSecurityConfig `不需要继承 `WebSecurityConfigurerAdapter` 在内部创建内部类继承 即可

然后可以在内部类中去可以分别配置