# Spring Security 自定义json登录接口和登陆页面

想要实现上述功能则继续对 `configure(HttpSecurity http)` 方法进行配置

`public class WebSecurityConfig extends WebSecurityConfigurerAdapter` 类

```java
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
                .loginPage("/login_page")   //登陆页面
                .loginProcessingUrl("/login")  //设置登录接口为"/login" 设置其的主要目的是方便ajax  且 登录名为username 密码名为password  使用post 提交
                .usernameParameter("username") //配置登录账户名对应的参数名
                .passwordParameter("password")  //配置登录账户密码对应的参数名
            
            	/**
            	*
            	*
            	* 成功后的处理
            	*
            	*
            	**/
            
            
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
            /**
            *失败后的处理
            **/
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
                .csrf()  //关闭 csrf
                .disable();  //应用

    }
```



自定义登陆页面

```java
.formLogin()  //开启表单登录
                .loginPage("/login_page")   //登陆页面
                .loginProcessingUrl("/login")  //设置登录接口为"/login" 设置其的主要目的是方便ajax  且 登录名为username 密码名为password  使用post 提交
                .usernameParameter("username") //配置登录账户名对应的参数名
                .passwordParameter("password")  //配置登录账户密码对应的参数名
            
            	/**
            	*
            	*
            	* 成功后的处理
            	*
            	*
            	**/
            
            
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
            /**
            *失败后的处理
            **/
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
```

测试 `http://localhost:8080/login` post 方法

成功

```json
{
	"code": 200,
	"message": "登录成功",
	"success": true,
	"data": {
		"login_data": {
			"password": null,
			"username": "gu",
			"authorities": [
				{
					"authority": "ROLE_USER"
				}
			],
			"accountNonExpired": true,
			"accountNonLocked": true,
			"credentialsNonExpired": true,
			"enabled": true
		}
	}
}
```

失败

```json
{
	"code": 401,
	"message": "登录失败",
	"success": false,
	"data": {
		"detail": "账户名或者密码输入错误"
	}
}
```

