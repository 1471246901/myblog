# Redis集群缓存

redis集群缓存的先决条件是在SpringBoot中已经配置好redis集群并已经能正常访问

## 配置

maven 依赖

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>io.lettuce</groupId>
                    <artifactId>lettuce-core</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

yml配置

```yml
spring:
  cache:
    cache-names: c1,c2
    redis:
      time-to-live: 1800s
  redis:
    cluster:
      nodes:
        - 192.168.1.107:8000
        - 192.168.1.107:8001
        - 192.168.1.107:8002
        - 192.168.1.107:8003
        - 192.168.1.107:8004
        - 192.168.1.107:8005
    jedis:
      pool:
        max-active: 8
        max-wait: -1ms
        max-idle: 8
        min-idle: 0
    password: root
    database: 0
```

检测rediscluster 是否配置成功

```java
@Autowired
    RedisTemplate redisTemplate;

    @Autowired
    StringRedisTemplate stringRedisTemplate;

    @Test
    public void testcluster(){
        ValueOperations<String, String> opsv = stringRedisTemplate.opsForValue();
        opsv.set("aaaa123","谷世玉");
        System.out.println(opsv.get("aaaa123"));
    }
```

## 使用

这时使用 `@CacheConfig `  `@Cacheable`  `@CachePut` `@CacheEvict`  即可控制缓存

若要自定义规则 则需自己配置

>    rediscache 由 RedisCacheManager  提供所以我们只需要提供该类即可 , 即由RedisConnectionFactory来构建RedisCacheManager  
>
>   RedisConnectionFactory 是配置reids集群时向容器中注入的对象

配置例子

```java
@Configuration
public class RedisCacheConfig {

    @Autowired
    RedisConnectionFactory factory;


    RedisCacheManager redisCacheManager(){
        Map<String, RedisCacheConfiguration> configurationMap = new HashMap<>();
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .prefixCacheNameWith("cache1")  //默认的前缀名
                .disableCachingNullValues()  //空值的处理
                .entryTtl(Duration.ofMinutes(30));  //缓存时间
        configurationMap.put("c1",redisCacheConfiguration);
        RedisCacheWriter cacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(factory);
        return new RedisCacheManager(cacheWriter,RedisCacheConfiguration.defaultCacheConfig(),configurationMap);

    }
}
```

当注解 `@CacheConfig(cacheNames = "cache规则名")`  为自己配置的时候,就是用该规则名对应的规则,当不存在该名称的时候就使用默认规则

默认规则缓存策略 通过调用`RedisCacheConfiguration` 的 `defaultCacheConfig` 方法获取 ,具体的默认配置策略可以查看该方法