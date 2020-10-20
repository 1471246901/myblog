# Shiro编写身份认证 简单版

#### Controller 中的登录接口`login`

```java
    @RequestMapping("/login")
    public String login(String userName,String userPassword){
        if("".equals(userName)|| userName==null) return  "未填写用户名";
        if("".equals(userPassword)|| userPassword==null) return  "未填写密码";

        //Subject 相当于用户
        Subject sub = SecurityUtils.getSubject();
        UsernamePasswordToken auth = new UsernamePasswordToken(userName,userPassword,false);
        //执行sub.login(auth); 会进入到UserRealm认证里面去了
        try {
            sub.login(auth);
            return  "登陆成功";
        }catch (AuthenticationException e){
            e.printStackTrace();
            return "登录失败";
        }
    }
```

**步骤**

1.  userName , userPassword 判空非法等操作

2.  SecurityUtils.getSubject ()获取到一个用户

3.  UsernamePasswordToken(userName,userPassword);  生成一个Token

    `UsernamePasswordToken(userName,userPassword,false);` false 代表是否记住我选项

4.  使用Subject .login() 去认证这个Token   失败会抛出`AuthenticationException` 异常`org.apache.shiro.authc.IncorrectCredentialsException: Submitted credentials for token`

5.  根据成功失败做相应的逻辑

    相关的异常

       ```java
try{
    } catch ( UnknownAccountException uae ) {
    	System.out.println("用户名不存在");
    } catch ( IncorrectCredentialsException ice ) {
    	System.out.println("密码错误");
    } catch ( LockedAccountException lae ) {
    	System.out.println("用户被锁定，不能登录");
    } catch ( AuthenticationException ae ) {
    	System.out.println("严重的错误");
       ```
    

#### 编写用户认证代码   这里先介绍单身份

`UserRealm` 也就是继承自`AuthorizingRealm`  的认证类 需要在config中配置,一般一个就可以了,如果每一个用户都有好多身份或者好多种身份 可以配置多个

`realm`包下 `UserRealm`  类的身份认证方法

```java
//身份认证
@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken auth) throws AuthenticationException {
    log.info("进入到认证里面");
    //获取到本体  就是username
    String username = (String) auth.getPrincipal();
    String userpassword = new String((char[])auth.getCredentials());
    log.info("username :"+username);
    //接下来通过username在数据库拿到用户信息
    //在判断用户密码对不对

    //模拟 如果用户名大于5就存在这个用户
    if(username.length()<5)return null;
    // 模拟查询到用户密码为 ******
    String DBpassword = "we1234567";

    //顺序为 用户名   数据库中的密码  用户填写的密码   直接返回即可
    return new SimpleAuthenticationInfo(username,DBpassword,userpassword);
}
```

**步骤**

1.  获取到username  和 userpassword 
2.  在数据库中查到用户的相关信息
3.  使用  `SimpleAuthenticationInfo(username,DBpassword,userpassword);` 来认证

#### 编写相关页面验证即可

