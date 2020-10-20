# Shiro角色和权限验证

#### 需要开启Shiro注解

先前已经配置过了

```java
//开启Shrio注解
    @Bean
    public AuthorizationAttributeSourceAdvisor getAuthorizationAttributeSourceAdvisor(UserRealm userRealm){
        AuthorizationAttributeSourceAdvisor aasa = new AuthorizationAttributeSourceAdvisor();
        aasa.setSecurityManager(getDefaultWebSecurityManager(userRealm));
        return aasa;
    }

```

#### 在userRealm中编写doGetAuthenticationInfo方法

```java
//权限认证
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        //根据凭据类型返回身份
        String p1 = principalCollection.oneByType(String.class);
        //获取一个主要的/随机的 身份
        String p3 = (String) principalCollection.getPrimaryPrincipal();

        log.info("p1 :" + p1);
        log.info("p3 :" + p3);

        //根据身份在数据库查找到响应的角色和权限
        List<String> roleList = new ArrayList<>(); //角色
        roleList.add("ADMIN");
        roleList.add("USER");
        List<String> permissionList = new ArrayList<>(); //权限
        permissionList.add("ADMIN:USER:UPDATE");
        permissionList.add("ADMIN:USER:DELETE");
        //将角色和权限加入进去返回即可
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        simpleAuthorizationInfo.addRoles(roleList);
        simpleAuthorizationInfo.addStringPermissions(permissionList);
        return simpleAuthorizationInfo;
    }
```

**步骤**

1.  过去到认证的身份
2.  身份在数据库查找到响应的角色和权限
3.  将角色和权限加入SimpleAuthorizationInfo返回

#### 编写测试方法

RoleController   **角色**

```java
 	@RequiresRoles({"ADMIN"})
    @RequestMapping("/admin")
    public String adminRole(){
        return  "admin";
    }

	@RequiresRoles({"OTHER"})
    @RequestMapping("/OTHER")
    public String otherRole(){
        return  "other";
    }
```

注解 `@RequiresRoles({"ADMIN"})`  可标注多个  是**与**的关系

标注`RequiresRoles` 只会在角色满足的时候才可以进入请求



PermissionsController  **权限**

```java
	@RequiresPermissions({"ADMIN:USER:UPDATE","ADMIN:USER:DELETE"})
    @RequestMapping("/upanddel")
    public String upAndDelPermission(){
        return  "upanddel";
    }
    @RequiresPermissions({"ADMIN:USER:INSERT","ADMIN:USER:SELECT"})
    @RequestMapping("/insandsel")
    public String insAndSelPermission(){
        return  "insAndSel";
    }
```

注解`RequiresPermissions({"ADMIN:USER:UPDATE","ADMIN:USER:DELETE"})` 

标注 `RequiresPermissions` 只会在权限满足的时候才可以进入请求