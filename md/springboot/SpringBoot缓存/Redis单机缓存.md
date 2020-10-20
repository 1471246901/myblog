# Redis单机缓存

如果Classpath下存在着redis的相关依赖,并且redis 已经配置好了此时默认就会使用RedisCacheManager 作为缓存的提供者

`RedisCacheConfiguration` 中的配置

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisConnectionFactory.class)
@AutoConfigureAfter(RedisAutoConfiguration.class)
@ConditionalOnBean(RedisConnectionFactory.class)
@ConditionalOnMissingBean(CacheManager.class)
@Conditional(CacheCondition.class)
class RedisCacheConfiguration {

	@Bean
	RedisCacheManager cacheManager(CacheProperties cacheProperties, CacheManagerCustomizers cacheManagerCustomizers,
			ObjectProvider<org.springframework.data.redis.cache.RedisCacheConfiguration> redisCacheConfiguration,
			ObjectProvider<RedisCacheManagerBuilderCustomizer> redisCacheManagerBuilderCustomizers,
			RedisConnectionFactory redisConnectionFactory, ResourceLoader resourceLoader) {
		RedisCacheManagerBuilder builder = RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(
				determineConfiguration(cacheProperties, redisCacheConfiguration, resourceLoader.getClassLoader()));
		List<String> cacheNames = cacheProperties.getCacheNames();
		if (!cacheNames.isEmpty()) {
			builder.initialCacheNames(new LinkedHashSet<>(cacheNames));
		}
		redisCacheManagerBuilderCustomizers.orderedStream().forEach((customizer) -> customizer.customize(builder));
		return cacheManagerCustomizers.customize(builder.build());
	}
```

## 配置

### maven依赖

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
```

### application.yml

```yml
spring:
  cache:
    # 默认缓存名称,缓存在redis 的前缀  key=[缓存名称::其他]
    cache-names: c1,c2
    redis:
      # key 的过期时间
      time-to-live: 1800s
  redis:
    database: 0
    host: 192.168.1.106
    port: 6380
    password: root
    jedis:
      pool:
        max-active: 8
        max-wait: -1ms
        max-idle: 8
        min-idle: 0
```

### 开启缓存 @EnableCaching

```java
@SpringBootApplication
@EnableCaching
public class RediscacheApplication {
```

## 使用

使用方法和 `Ehcache ` 一样 使用  `@CacheConfig `  `@Cacheable`  `@CachePut` `@CacheEvict`  控制缓存

自定义缓存key方法仍然和`Ehcache `  一样

`@CacheConfig ` 可以通过 cacheNames 属性指定缓存名称前缀

## 测试

测试用例

```java
@Autowired
    BookDao bookDao;

    @Test
    public void testCache(){
        System.out.println("执行第一次");
        bookDao.getAll(1);

        System.out.println("执行第二次");
        bookDao.getAll(1);

        System.out.println("另一个id执行第一次");
        bookDao.getAll(2);


    }
```

结果

```
执行第一次
book Dao 被执行 id:1
执行第二次
另一个id执行第一次
book Dao 被执行 id:2
```

redis 

![image-20200914102150836](Redis%E5%8D%95%E6%9C%BA%E7%BC%93%E5%AD%98.assets/image-20200914102150836.png)



