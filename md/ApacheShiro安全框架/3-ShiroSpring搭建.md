# ShiroSpring搭建

#### 引入maven依赖

pom引入

```xml
<!--Shiro-->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-core</artifactId>
            <version>1.5.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-web</artifactId>
            <version>1.5.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-ehcache</artifactId>
            <version>1.5.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring</artifactId>
            <version>1.5.2</version>
        </dependency>
```

#### 引入Shiro配置文件

ehcache-shiro.xml   在资源路径下

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<ehcache name="es">  
    <diskStore path="java.io.tmpdir"/>  
         <!--  
       name:缓存名称。  
       maxElementsInMemory:缓存最大数目  
       maxElementsOnDisk：硬盘最大缓存个数。   
       eternal:对象是否永久有效，一但设置了，timeout将不起作用。   
       overflowToDisk:是否保存到磁盘，当系统当机时  
       timeToIdleSeconds:设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。  
       timeToLiveSeconds:设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。  
       diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.   
       diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。   
       diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。  
       memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。   
       clearOnFlush：内存数量最大时是否清除。  
       memoryStoreEvictionPolicy:  
            Ehcache的三种清空策略;  
            FIFO，first in first out，这个是大家最熟的，先进先出。  
            LFU， Less Frequently Used，就是上面例子中使用的策略，直白一点就是讲一直以来最少被使用的。如上面所讲，缓存的元素有一个hit属性，hit值最小的将会被清出缓存。  
            LRU，Least Recently Used，最近最少使用的，缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存。  
    -->  
    <defaultCache  
            maxElementsInMemory="10000"  
            eternal="false"  
            timeToIdleSeconds="120"  
            timeToLiveSeconds="120"  
            overflowToDisk="false"  
            diskPersistent="false"  
            diskExpiryThreadIntervalSeconds="120"  
            memoryStoreEvictionPolicy="LRU"
            />  
    <!-- 登录记录缓存锁定10分钟 -->  
    <cache name="passwordRetryCache"  
           maxEntriesLocalHeap="2000"  
           eternal="false"  
           timeToIdleSeconds="3600"  
           timeToLiveSeconds="0"  
           overflowToDisk="false"  
           statistics="true">  
    </cache>  
</ehcache>  
```

#### 创建java配置文件

`config包下`

```java

@Configuration
public class ShiroConfiguration {

    @Bean
    public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager){
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        //必须设置securityManager
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        //拦截器
        Map<String,String> filterChainDefinitionMap = new LinkedHashMap<>();
        //添加放开的请求   anon 代表放开
        filterChainDefinitionMap.put("/login","anon");
        filterChainDefinitionMap.put("/userlogin","anon");
        //设置要认证的请求  要拦截的请求
        filterChainDefinitionMap.put("/**","authc");
        //将map设置进shiroFilterFactoryBean 里
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);

        //设置登录接口的url
        shiroFilterFactoryBean.setLoginUrl("/login");
        //设置未登录跳转页面
        shiroFilterFactoryBean.setUnauthorizedUrl("/login");
        return shiroFilterFactoryBean;
    }


    @Bean
    public EhCacheManager getEhCacheManager(){
        EhCacheManager em  = new EhCacheManager();
        //设置ehcache-shiro.xml 配置文件的目录
        em.setCacheManagerConfigFile("classpath:ehcache-shiro.xml");
        return em;
    }

    @Bean
    public DefaultAdvisorAutoProxyCreator getDefaultAdvisorAutoProxyCreator(){
        //配置代理
        DefaultAdvisorAutoProxyCreator daap = new DefaultAdvisorAutoProxyCreator();
        daap.setProxyTargetClass(true);
        return daap;
    }

    @Bean
    public DefaultWebSessionManager getDefaultWebSessionManager(){
        DefaultWebSessionManager defaultWebSessionManager = new DefaultWebSessionManager();
        defaultWebSessionManager.setSessionDAO(getMemorySessionDAO());
        defaultWebSessionManager.setGlobalSessionTimeout(1*60*60*1000);
        defaultWebSessionManager.setSessionValidationSchedulerEnabled(true);
        defaultWebSessionManager.setSessionIdCookieEnabled(true);
        defaultWebSessionManager.setSessionIdCookie(getSimpleCookie());
        return defaultWebSessionManager;
    }


    @Bean
    public MemorySessionDAO getMemorySessionDAO(){
        MemorySessionDAO memorySessionDAO = new MemorySessionDAO();
        memorySessionDAO.setSessionIdGenerator(JavaUuidSessionIdGenerator());
        return memorySessionDAO;
    }

    @Bean
    public JavaUuidSessionIdGenerator JavaUuidSessionIdGenerator(){
        return new JavaUuidSessionIdGenerator();
    }

    //session 自定义cookie名
    @Bean
    public SimpleCookie getSimpleCookie(){
        SimpleCookie simpleCookie = new SimpleCookie();
        simpleCookie.setName("security.session.id");
        simpleCookie.setPath("/");
        return simpleCookie;
    }


    @Bean
    public LifecycleBeanPostProcessor getLifecycleBeanPostProcessor(){
        return new LifecycleBeanPostProcessor();
    }

    @Bean(name = "securityManager")
    public DefaultWebSecurityManager getDefaultWebSecurityManager(UserRealm userRealm){
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();

        defaultWebSecurityManager.setRealm(userRealm);
        //用户授权/认证信息cache,采用ECache 缓存
        defaultWebSecurityManager.setCacheManager(getEhCacheManager());
        defaultWebSecurityManager.setSessionManager(getDefaultWebSessionManager());
        return defaultWebSecurityManager;
    }

    @Bean
    public UserRealm userRealm(EhCacheManager cacheManager){
        UserRealm userRealm = new UserRealm();
        userRealm.setCacheManager(cacheManager);
        return userRealm;
    }

    //开启Shrio注解
    @Bean
    public AuthorizationAttributeSourceAdvisor getAuthorizationAttributeSourceAdvisor(UserRealm userRealm){
        AuthorizationAttributeSourceAdvisor aasa = new AuthorizationAttributeSourceAdvisor();
        aasa.setSecurityManager(getDefaultWebSecurityManager(userRealm));
        return aasa;
    }


}

```



#### 创建用户认证过程

创建认证类   继承 `AuthorizingRealm` 类   `Realm 包下`

```java
public class UserRealm extends AuthorizingRealm {
    
    //权限认证
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    //身份认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        return null;
    }
}

```

#### 启动spring 项目验证

在浏览器中输入 `127.0.0.1:8080` 随后浏览器会路由到 `127.0.0.1:8080/login`

只要是未登录都会路由到指定页面

![image-20200417135807414](D:\Typora\data\image\image-20200417135807414.png)



