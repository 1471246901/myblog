# Shiro一处登录与退出

#### 退出

```java

	@RequestMapping("/loginout")
    public String loginOut(){
        Subject sub = SecurityUtils.getSubject();
        sub.logout();
        return  "userlogin ";
    }
    
```

#### 一处登录

一处登录指的是用户只能在一台设备上登录,如果在另一台设备登录 则需要处理(先前用户退出)

实现方法.在认证之前看一下sessionDao中有没有这个用户的session

SessionDao在config配置的时候已经注入到springIOC了 注入即可

修改  `realm`包下 `UserRealm`  类的身份认证方法

```java
//身份认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken auth) throws AuthenticationException {
        log.info("进入到认证里面");
        //获取到本体  就是username
        String username = (String) auth.getPrincipal();
        String userpassword = new String((char[])auth.getCredentials());
        log.info("username :"+username);
        //再认证之前 查看是否已经登录
        Collection<Session> sessions = sessionDAO.getActiveSessions();
        for(Session session:sessions) {
            //用户  也就是   auth.getPrincipal();得到的信息
            Object o = session.getAttribute(DefaultSubjectContext.PRINCIPALS_SESSION_KEY);
            //是否已经登录
            session.getAttribute(DefaultSubjectContext.AUTHENTICATED_SESSION_KEY);
            //判断session 是否存在
            if(username.equals(String.valueOf(o))){
                //将其下线
                session.setTimeout(0);
                //或者
                sessionDAO.delete(session);

            }

        }
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

