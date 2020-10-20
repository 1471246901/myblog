# Shiro获取用户登录信息

主要使用  SecurityUtils.getSubject(); 获取

在通过Subject 的一些方法

下面是一些主要的方法

```java
 @RequestMapping("/index")
    public String index(){


        Subject subject = SecurityUtils.getSubject();
        //再认证的时候Principal 可以是用户对象 ,不单单只是username 是用什么认证就转化为什么对象
        String username = (String) subject.getPrincipal();
        //获取到多个认证
        PrincipalCollection s = subject.getPrincipals();
        //得到切换身份之前的身份，一个用户可以切换很多次身份，之前的身份使用栈数据结构来存储；
        PrincipalCollection previousPrincipals = subject.getPreviousPrincipals();
        //表示当前用户是否是RunAs用户，即已经切换身份了；
        boolean runAs = subject.isRunAs();

        return  "index "+username + " 是否在切换身份 "+runAs;
    }
```

