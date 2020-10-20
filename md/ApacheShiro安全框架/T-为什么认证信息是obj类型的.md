# 为什么认证信息是Object类型的

#### subject.getPrincipal(); 解释

Principal 实际是代表用户 他可以是用户类 也可以是用户名 

而`SimpleAuthenticationInfo(username,DBpassword,userpassword); ` 对象实际是

用户 , 用户的凭证,用户提交的凭证  ,而不是单纯的账号密码关系

