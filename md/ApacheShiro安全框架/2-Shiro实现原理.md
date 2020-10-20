# Shiro实现原理

![img](D:\Typora\data\image\20181205210949620.png)

应用代码通过 Subject 来进行认证和授权，而 Subject 又委托给 SecurityManager； 我们需要给 Shiro 的 SecurityManager 注入 Realm，从而让 SecurityManager 能得到合法
的用户及其权限进行判断。