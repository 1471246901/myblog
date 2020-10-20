# SpringSecurity注销登录

当启用WebSecurityConfigurerAdapter的时候，logout支持会自动启用

logoutUrl()  访问那个地址会触发登出逻辑。**默认情况下CROS是开启的，另外必须是POST方法**

logoutSuccessUrl()  当登出成功之后，会被重定向到的地址

logoutSuccessHandler()指定登出成功后的处理，如果指定了这个，那么`logoutSuccessUrl`就不会生效。

addLogoutHandler  添加登出时的Handler，在访问logout地址时会执行，内部是一个立碑，`SecurityContextLogoutHandler`默认会加到最后，

invalidateHttpSession 在登出后，是否要清空当前session

#### LogoutHandler

LogoutHandler 即在程序执行logout时一起参与执行其中的处理逻辑，**不能抛出异常**，官方默认提供了几个实现。

-   [PersistentTokenBasedRememberMeServices](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.spring.io%2Fspring-security%2Fsite%2Fdocs%2Fcurrent%2Fapi%2Forg%2Fspringframework%2Fsecurity%2Fweb%2Fauthentication%2Frememberme%2FPersistentTokenBasedRememberMeServices.html)
-   [TokenBasedRememberMeServices](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.spring.io%2Fspring-security%2Fsite%2Fdocs%2Fcurrent%2Fapi%2Forg%2Fspringframework%2Fsecurity%2Fweb%2Fauthentication%2Frememberme%2FTokenBasedRememberMeServices.html)  移除Token
-   [CookieClearingLogoutHandler ](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.spring.io%2Fspring-security%2Fsite%2Fdocs%2Fcurrent%2Fapi%2Forg%2Fspringframework%2Fsecurity%2Fweb%2Fauthentication%2Flogout%2FCookieClearingLogoutHandler.html)  清楚Cookie
-   [CsrfLogoutHandler](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.spring.io%2Fspring-security%2Fsite%2Fdocs%2Fcurrent%2Fapi%2Forg%2Fspringframework%2Fsecurity%2Fweb%2Fcsrf%2FCsrfLogoutHandler.html)   移除CSRF TOKEN
-   [SecurityContextLogoutHandler](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.spring.io%2Fspring-security%2Fsite%2Fdocs%2Fcurrent%2Fapi%2Forg%2Fspringframework%2Fsecurity%2Fweb%2Fauthentication%2Flogout%2FSecurityContextLogoutHandler.html)
-   [HeaderWriterLogoutHandler](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.spring.io%2Fspring-security%2Fsite%2Fdocs%2Fcurrent%2Fapi%2Forg%2Fspringframework%2Fsecurity%2Fweb%2Fauthentication%2Flogout%2FHeaderWriterLogoutHandler.html)

#### LogoutSuccessHandler

在调用完LogoutHandler之后，并且处理成功后调用，**可以抛出异常**，官方默认提供了两个

-   [SimpleUrlLogoutSuccessHandler](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.spring.io%2Fspring-security%2Fsite%2Fdocs%2Fcurrent%2Fapi%2Forg%2Fspringframework%2Fsecurity%2Fweb%2Fauthentication%2Flogout%2FSimpleUrlLogoutSuccessHandler.html)  不需要直接指定，在指定logoutSuccessUrl()会自己使用
-   HttpStatusReturningLogoutSuccessHandler  返回登出成功后的状态码



```java
@Override
    protected void configure(HttpSecurity http) throws Exception {



        http.authorizeRequests() //开启httpsecurity 配置 允许基于该限制访问HttpServletRequest使用RequestMatcher的实现（即，经由URL模式）
                //省略其他
            //省略其他
            //省略其他
            //省略其他
            //省略其他
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
```

测试 

访问 `http://localhost:8080/login` post方式 ,参数  `username=admin&password=root`

登陆成功

访问  `http://localhost:8080/logout`  结果 

```json
{
	"code": 200,
	"message": "登陆成功",
	"success": true,
	"data": {}
}
```

