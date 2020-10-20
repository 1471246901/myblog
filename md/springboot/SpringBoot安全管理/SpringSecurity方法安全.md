# SpringSecurity方法安全

在 `WebSecurityConfigurerAdapter`  中只可以配置url 对应的认证和授权,我们也可以通过注解来方便的控制方法级别的安全控制

要通过 `@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true)` 来开启对应注解 

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
```

**prePostEnabled 属性解锁 `@PreAuthorize` 和 `@PostAuthorize` 两个注解**

​	`@PreAuthorize` 表示在方法执行之前进行验证

 	`@PostAuthorize` 表示在方法之后进行验证

**securedEnabled 属性解锁 `@Secured`** 

​	`@Secured`："ROLE_ADMIN"允许调用此方法的角色。若多个角色均允许调用此方法，可并列多个角色，如： `@Secured("ROLE_ADMIN","ROLE_USER")`  

这里需要在角色前加上 `ROLE_`  

`@Secured` 不支持表达式,  `@PreAuthorize` 和  `@PostAuthorize`  支持表达式

**注解不只可以应用到controller中也可以应用到service中**

##  测试

```java
@RestController
@RequestMapping("/auth")
public class AuthController {


    @RequestMapping("/admin")
    @Secured("ROLE_ADMIN")
    public Object admin(){
        return "@Secured(\"ROLE_ADMIN\")";
    }

    @RequestMapping("/all")
    @PreAuthorize("hasAnyRole('ADMIN','USER','DBA')")
    public Object all(){
        return "@PreAuthorize(\"hasAnyRole('ADMIN','USER','DBA')\")";
    }

    @RequestMapping("/user")
    @PreAuthorize("hasRole('USER')")
    public Object user(){
        return "@PreAuthorize(\"hasRole('USER')\")";
    }

    @RequestMapping("/dba")
    @PostAuthorize("hasRole('DBA')")
    public Object dba(){
        return "@PostAuthorize(\"hasRole('DBA')\")";
    }

    @RequestMapping("/adminanddba")
    @PreAuthorize("hasRole('ADMIN') and hasRole('DBA')")
    public Object adminanddba(){
        return "@PreAuthorize(\"hasRole('ADMIN') and hasRole('DBA')\")";
    }
```

启动并登录 admin  账号

| URL                                    | 结果                                  |
| -------------------------------------- | ------------------------------------- |
| http://localhost:8080/auth/admin       | 成功访问                              |
| http://localhost:8080/auth/all         | 成功访问                              |
| http://localhost:8080/auth/user        | 成功访问                              |
| http://localhost:8080/auth/dba         | 访问失败(type=Forbidden, status=403). |
| http://localhost:8080/auth/adminanddba | 访问失败(type=Forbidden, status=403)  |

由此可见 注解生效

 `@PostAuthorize` 注解是方法执行后才生效  返回   拒绝访问 Access is denied 

`@PreAuthorize`  在方法执行前检查   返回  不允许访问