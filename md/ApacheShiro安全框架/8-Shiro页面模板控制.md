# Shiro页面控制

例如在页面上登录后就会显示退出,不登录会显示登录,权限低的不能看到权限高的选项等

### FreeMaker模板引擎

#### 引入freemaker模板引擎

##### 引入freemaker的依赖包

```xml
<!-- 引入freeMarker的依赖包. -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```

##### application.yml 或properties中配置

```properties
spring.freemarker.allow-request-override=false
spring.freemarker.cache=true
spring.freemarker.check-template-location=true
# 编码
spring.freemarker.charset=UTF-8
# 类型
spring.freemarker.content-type=text/html
# 前缀名
#spring.freemarker.prefix=
# 后缀名
spring.freemarker.suffix=.ftl   
# 模板加载路径
spring.freemarker.template-loader-path=classpath:/templates/
```

##### 编写测试

**classpath:/templates/index.ftl**

```html
<!DOCTYPE html>
<html>
<head lang="en">
<meta charset="UTF-8" />
<title></title>
</head>
<body>
    
</body>
</html>
```

**TestFreeMakerController**

```java

    @RequestMapping("/index")
    public String index(Map<String,String> map){
       map.put("tip","index "+username + " 是否在切换身份 "+runAs);
        return  "index";
    }
```

请求返回正确页面即可

#### Shiro和FreeMaker 整合实现页面的权限控制

##### 引入整合依赖包

```xml
<!-- 引入shiro和 freeMarker整合的依赖包. -->
        <dependency>
            <groupId>net.mingsoft</groupId>
            <artifactId>shiro-freemarker-tags</artifactId>
            <version>0.1</version>
        </dependency>
```

##### 将Shiro配置到FreeMaker中

`config` 包下 `FreeMakerShiroConfig` 类

```java
@Configuration
public class FreeMakerShiroConfig {

    @Autowired
    private FreeMarkerConfigurer configurer;

    @PostConstruct
    public void setShiroTags(){
        configurer.getConfiguration().setSharedVariable("shiro",new ShiroTags());
    }
    
}
```

```java
//new ShiroTags() 中的内容 标签中可以使用
this.put("authenticated", new AuthenticatedTag());
this.put("guest", new GuestTag());
this.put("hasAnyRoles", new HasAnyRolesTag());
this.put("hasPermission", new HasPermissionTag());
this.put("hasRole", new HasRoleTag());
this.put("lacksPermission", new LacksPermissionTag());
this.put("lacksRole", new LacksRoleTag());
this.put("notAuthenticated", new NotAuthenticatedTag());
this.put("principal", new PrincipalTag());
this.put("user", new UserTag());
```

>   @PostConstruct 注解 
>
>   再将@Configuration 注释的(或者配置文件)组件配置完成后执行该方法
>
>   常常用于更改已经存在与容器中但是需要再更改一下的组件

##### ftl模板文件中使用

```jsp
<p>hello html ${tip}</p>
<p>身份</p>
<ul>
    <@shiro.hasRole name="ADMIN">
        <li>admin</li>
    </@shiro.hasRole>

    <@shiro.hasRole name="USER">
    <li>user</li>
    </@shiro.hasRole>

    <@shiro.hasRole name="OTHER">
    <li>other</li>
    </@shiro.hasRole>
</ul>
<p>权限</p>
<ul>
    <@shiro.hasPermission  name="ADMIN:USER:UPDATE">
    <li>ADMIN:USER:UPDATE</li>
    </@shiro.hasPermission>

    <@shiro.hasPermission  name="ADMIN:USER:DELETE">
    <li>ADMIN:USER:DELETE</li>
    </@shiro.hasPermission>

    <@shiro.hasPermission  name="ADMIN:USER:INSERT">
    <li>ADMIN:USER:INSERT</li>
    </@shiro.hasPermission>

</ul>
```

##### 标签

```html
guest标签
　　<@shiro.guest>
　　</@shiro.guest>
　　用户没有身份验证时显示相应信息，即游客访问信息。

user标签
　　<@shiro.user>　　
　　</@shiro.user>
　　用户已经身份验证/记住我登录后显示相应的信息。

authenticated标签
　　<@shiro.authenticated>　　
　　</@shiro.authenticated>
　　用户已经身份验证通过，即Subject.login登录成功，不是记住我登录的。

notAuthenticated标签
　　<@shiro.notAuthenticated>
　　
　　</@shiro.notAuthenticated>
　　用户已经身份验证通过，即没有调用Subject.login进行登录，包括记住我自动登录的也属于未进行身份验证。

principal标签
　　<@shiro.principal/>
　　
　　<@shiro.principal property="username"/>
　　相当于((User)Subject.getPrincipals()).getUsername()。

lacksPermission标签
　　<@shiro.lacksPermission name="org:create">
　
　　</@shiro.lacksPermission>
　　如果当前Subject没有权限将显示body体内容。

hasRole标签
　　<@shiro.hasRole name="admin">　　
　　</@shiro.hasRole>
　　如果当前Subject有角色将显示body体内容。

hasAnyRoles标签
　　<@shiro.hasAnyRoles name="admin,user">
　　　
　　</@shiro.hasAnyRoles>
　　如果当前Subject有任意一个角色（或的关系）将显示body体内容。

lacksRole标签
　　<@shiro.lacksRole name="abc">　　
　　</@shiro.lacksRole>
　　如果当前Subject没有角色将显示body体内容。

hasPermission标签
　　<@shiro.hasPermission name="user:create">　　
　　</@shiro.hasPermission>
　　如果当前Subject有权限将显示body体内容
```

### thymeleaf 模板引擎