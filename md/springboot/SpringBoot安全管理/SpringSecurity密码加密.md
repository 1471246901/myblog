# 密码加密

因为很多人密码都会使用同一密码,即多个网站适合用同一密码,所以当密码泄露后,到用着就会去别的网站上尝试,所以密码需要加密存储

[参考文章](https://blog.csdn.net/zhouyan8603/article/details/80473083) 

## 加密方案

密码加密一般用到散列函数,又称散列算法,哈希函数,常用的散列函数有MD5消息摘要算法,安全散列算法

但是一般仅仅使用散列函数还不够,为了增加密码的安全性,一般在密码加密的过程中还需要加盐,盐可以是算技术,可以是用户名 传统的方式需要在数据库有专门的字段来存储盐,在SpringSecurity 中提供了多种密码加密方式,官方推荐使用 `BCryptPasswordEncoder`  其使用 `BCrypt` 强哈希函数,在使用时可以提供 `strength` 和 `SecureRandom` 实例 ,`strength`  越强大,密钥的迭代次数就越多 *密钥的迭代次数为2^strength 取值在 3-31之间*  

 在 `WebSecurityConfigurerAdapter` 配置类中配置

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {



    @Bean
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder(10);  //可以不指定 默认为10
    }
    
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
```



在 passwordEncoder() 中注入PasswordEncoder 方便在其他地方使用

在configure(AuthenticationManagerBuilder auth) 中使用 .passwordEncoder(passwordEncoder()) 设置

所有password 都要使用 `passwordEncoder`.encode("密码") 来加密密码

