# SpringSecurity 角色继承

我们定义了三种角色 一种是 USER 权限最低,还有是ADMIN ,权限最高的是DBA,admin一般既具有admin的权限还具有user的权限,这就需要SpringSecurity 的权限继承了

## 配置

假设DBA 具有最高权限,其次是ADMIN 再次是USER

我们只需要在`WebSecurityConfigurerAdapter` 配置类中提供一个 `RoleHierarchy  ` 类即可

```java
@Bean
    RoleHierarchy roleHierarchy(){
        RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();

        String hierarchy = "ROLE_DBA > ROLE_ADMIN ROLE_ADMIN > ROLE_USER";
        roleHierarchy.setHierarchy(hierarchy);
        return roleHierarchy;
    }
```

以前我们是这样配置的

```java
        http.authorizeRequests() //开启httpsecurity 配置 允许基于该限制访问HttpServletRequest使用RequestMatcher的实现（即，经由URL模式）
                .antMatchers("/admin/**").hasRole("ADMIN")
                .antMatchers("/user/**").access("hasAnyRole('ADMIN','USER','DBA')")
                .antMatchers("/db/**").access("hasRole('DBA')")
```

某一URL 配置access 来实现多个角色访问,现在调整一下

```java
http.authorizeRequests() //开启httpsecurity 配置 允许基于该限制访问HttpServletRequest使用RequestMatcher的实现（即，经由URL模式）
                .antMatchers("/admin/**").hasRole("ADMIN")
                .antMatchers("/user/**").access("hasRole('USER')")
                .antMatchers("/db/**").access("hasRole('DBA')")
```

## 测试

登录 admin账号 ,也可以访问user 的目录 ,但是访问dba的链接会被拒绝

