# SpringSecurity动态权限配置

前面在数据库验证的时候,只添加了用户和角色以及角色用户关联表,我们在配置权限的时候还可以配置权限表或者叫资源表,表示角色对某一资源的权限

[spring 之 ObjectPostProcessor](https://blog.csdn.net/u012881904/article/details/103544151)

## 数据库

在此前的基础上在增加一张资源表和资源角色关联表,资源表中定义了能够访问的URL模式,资源角色表则定义了访问该模式的URL需要什么样的角色

**增加数据库表**

![image-20200924121226831](SpringSecurity%E5%8A%A8%E6%80%81%E6%9D%83%E9%99%90%E9%85%8D%E7%BD%AE.assets/image-20200924121226831.png)

## 自定义 FilterInvocationSecurityMetadataSource

此组件主要自定义能访问资源的角色

创建组件  `FilterInvocationSecurityMetadataSource` 

```java
@Component
public class CustomFilterInvocationSecurityMetadataSource
        implements FilterInvocationSecurityMetadataSource {

    AntPathMatcher antPathMatcher = new AntPathMatcher();

    @Autowired
    MenuMapper menuMapper;


    @Autowired
    RoleMapper roleMapper;


    /**
     * 此方法用户对传入的url设置相应的角色 ,方法内的 menuMapper.listByAll(); 可以缓存在缓存服务器中提速
     * @param object 传入的认证资源 , 在本项目中为 FilterInvocation
     * @return Collection<ConfigAttribute> 可以访问该资源的角色集合
     * @throws IllegalArgumentException  抛出的异常表明向方法传递了一个不合法或不正确的说法
     */
    @Override
    public Collection<ConfigAttribute> getAttributes(Object object) throws IllegalArgumentException {

        String requestUrl = ((FilterInvocation) object).getRequestUrl();

        List<Menu> all = menuMapper.listByAll();
        System.out.println(menuMapper);
        for(Menu menu : all){
            if(antPathMatcher.match(menu.getPattern(),requestUrl)){


                List<String> roles = new ArrayList<>();
                menu.getRoles().iterator().forEachRemaining((role)-> {
                    roles.add(role.getName());
                });
                return SecurityConfig.createList(roles.toArray(String[]::new));
            }
        }
        return SecurityConfig.createList("ROLE_LOGIN");
    }

    /**
     * 如果不需要校验则返回null
     * 此方法用来返回所有定义好的权限资源,也就是 上一方法中可以返回的  ConfigAttribute  ,在启动的时候Security会校验定义好的资源
     * @return  定义好的权限资源
     */
    @Override
    public Collection<ConfigAttribute> getAllConfigAttributes() {
        return null;
    }


    /**
     * 返回类对象是否支持校验
     * @param clazz
     * @return false
     */
    @Override
    public boolean supports(Class<?> clazz) {
        return false;
    }
}

```

##### 1.Collection getAttributes(Object object) throws IllegalArgumentException;

获取某个受保护的安全对象object的所需要的权限信息,是一组ConfigAttribute对象的集合，如果该安全对象object不被当前SecurityMetadataSource对象支持,则抛出异常IllegalArgumentException。
该方法通常配合boolean supports(Class<?> clazz)一起使用，先使用boolean supports(Class<?> clazz)确保安全对象能被当前SecurityMetadataSource支持，然后再调用该方法。

##### 2.Collection getAllConfigAttributes()

获取该SecurityMetadataSource对象中保存的针对所有安全对象的权限信息的集合。该方法的主要目的是被AbstractSecurityInterceptor用于启动时校验每个ConfigAttribute对象。

##### 3.boolean supports(Class<?> clazz)

这里clazz表示安全对象的类型，该方法用于告知调用者当前SecurityMetadataSource是否支持此类安全对象，只有支持的时候，才能对这类安全对象调用getAttributes方法。

## 自定义AccessDecisionManager

在通过 `FilterInvocationSecurityMetadataSource`  获取到该URL所需要的URL后进入到

`AccessDecisionManager` 进行判断, 其会提供 用户的登录信息,访问的资源,访问资源所需要的角色

开发者通过decide 方法进行比对即可

```java
/**
 * @Description 进行角色比对
 * @Author Gushiyu
 * @Date 2020-09-24 16:05
 */
public class CustomAccessDecisionManager implements AccessDecisionManager {


    /**
     * 该方法判断当前登录的用户是否具有请求当前URL的做需要的角色信息
     * 如果不具备访问该URL的权限 就抛出AccessDeniedException
     * 如果具备 则不做任何操作
     * @param authentication  包含当前用户的登录信息
     * @param object    是FilterInvocation 对象 和 FilterInvocationSecurityMetadataSource 是一个意思
     *                  表示请求的资源,可以获取当前请求的对象
     * @param configAttributes 就是 FilterInvocationSecurityMetadataSource 返回的可访问URL的对象
     * @throws AccessDeniedException
     * @throws InsufficientAuthenticationException
     */
    @Override
    public void decide(Authentication authentication,
                       Object object,
                        Collection<ConfigAttribute> configAttributes) throws AccessDeniedException, InsufficientAuthenticationException {
        Collection<? extends GrantedAuthority> auths = authentication.getAuthorities();

        //对访问此URL需要的角色进行遍历
        for (ConfigAttribute configAttribute:configAttributes){

            //如果当前的信息只需要登陆{"ROLE_LOGIN".equals(configAttribute.getAttribute())},
            // 并且{authentication instanceof UsernamePasswordAuthenticationToken} 表示当前用户信息为如果为 UsernamePasswordAuthenticationToken 则表示用户已经登陆
            // 直接返回放行就行
            if("ROLE_LOGIN".equals(configAttribute.getAttribute()) && authentication instanceof UsernamePasswordAuthenticationToken){
                return;
            }

            //进入正常判断流程  ,获取到用户具的角色并进行遍历
            for (GrantedAuthority authority:auths){
                //如果存在一个角色和需要的角色相等则通过
                if(configAttribute.getAttribute().equals(authority.getAuthority())){
                    return;
                }
            }
        }
        //都不通过则报错权限不足
        throw new AccessDeniedException("权限不足");
    }

    @Override
    public boolean supports(ConfigAttribute attribute) {
        return false;
    }

    @Override
    public boolean supports(Class<?> clazz) {
        return false;
    }
}

```

## 将两个组件配置到SecurityConfigurerAdapter

在 `WebSecurityConfigurerAdapter `中配置

```java
	@Autowired
    CustomAccessDecisionManager customAccessDecisionManager;

    @Autowired
    CustomFilterInvocationSecurityMetadataSource customFilterInvocationSecurityMetadataSource;



@Override
    protected void configure(HttpSecurity http) throws Exception {


        http.authorizeRequests() // 开启httpsecurity 配置 允许基于该限制访问HttpServletRequest使用RequestMatcher的实现（即，经由URL模式）
//                .antMatchers("/admin/**").hasRole("ADMIN")
//                .antMatchers("/user/**").access("hasRole('USER')")
//                .antMatchers("/dba/**").access("hasRole('DBA')")
                .withObjectPostProcessor(new ObjectPostProcessor<FilterSecurityInterceptor>() {

                    //将两个组件注册到Security中
                    @Override
                    public <O extends FilterSecurityInterceptor> O postProcess(O object) {
                        object.setSecurityMetadataSource(customFilterInvocationSecurityMetadataSource);
                        object.setAccessDecisionManager(customAccessDecisionManager);
                        return null;
                    }
                })
                .anyRequest()  // 表示除了前面配置的url之外,所有的url 都会使用下面的配置
                .authenticated()  //都要验证身份 ,结合上面anyRequest 表示除了上面自定义的规则其余所有的url都需要遵守authenticated 即所有url都要登录
                .and()  // 开启新的配置
        
```

**解释**

`withObjectPostProcessor`()     设置处理器

`FilterSecurityInterceptor`类   方法级别的权限拦截器  在使用方法过滤的时候就是使用的此类

`postProcess`()  方法 将自己定义的组件加载到  `FilterSecurityInterceptor` 中 然后配置到

因为Security 并不会扫描容器中的组件来 所以需要   `ObjectPostProcessor `来适配



## 测试



测试成功!!!!!!!!!!





有可能造成loginpage页面一直重定向  因为  loginpage 没有登陆又会跳转到loginpage     .permitAll()  也不管用