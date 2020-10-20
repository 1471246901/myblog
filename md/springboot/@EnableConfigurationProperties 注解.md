# 关与 @EnableConfigurationProperties 注解

@EnableConfigurationProperties注解的作用是：使使用 @ConfigurationProperties 注解的类生效。

#### 说明：

如果一个配置类只配置@ConfigurationProperties注解，而没有使用@Component，那么在IOC容器中是获取不到properties 配置文件转化的bean。说白了 @EnableConfigurationProperties 相当于把使用 @ConfigurationProperties 的类进行了一次注入。
测试发现 @ConfigurationProperties 与 @EnableConfigurationProperties 关系特别大。

测试证明：
`@ConfigurationProperties` 与 `@EnableConfigurationProperties` 的关系。

`@EnableConfigurationProperties` 文档中解释：
当`@EnableConfigurationProperties`注解应用到你的`@Configuration`时， 任何被`@ConfigurationProperties`注解的beans将自动被Environment属性配置。 这种风格的配置特别适合与SpringApplication的外部YAML配置进行配合使用。

测试发现：
1.使用 `@EnableConfigurationProperties` 进行注册



```kotlin
@ConfigurationProperties(prefix = "service.properties")
public class HelloServiceProperties {
    private static final String SERVICE_NAME = "test-service";

    private String msg = SERVICE_NAME;
       set/get
}


@Configuration
@EnableConfigurationProperties(HelloServiceProperties.class)
@ConditionalOnClass(HelloService.class)
@ConditionalOnProperty(prefix = "hello", value = "enable", matchIfMissing = true)
public class HelloServiceAutoConfiguration {

}

@RestController
public class ConfigurationPropertiesController {

    @Autowired
    private HelloServiceProperties helloServiceProperties;

    @RequestMapping("/getObjectProperties")
    public Object getObjectProperties () {
        System.out.println(helloServiceProperties.getMsg());
        return myConfigTest.getProperties();
    }
}
```

# 自动配置设置



```xml
service.properties.name=my-test-name
service.properties.ip=192.168.1.1
service.user=kayle
service.port=8080
```

一切正常，但是 HelloServiceAutoConfiguration 头部不使用 `@EnableConfigurationProperties`，测访问报错。

2.不使用 `@EnableConfigurationProperties` 进行注册，使用 `@Component` 注册



```java
@ConfigurationProperties(prefix = "service.properties")
@Component
public class HelloServiceProperties {
    private static final String SERVICE_NAME = "test-service";

    private String msg = SERVICE_NAME;

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

}
```

Controller 不变，一切正常，如果注释掉 @Component 测启动报错。
由此证明，两种方式都是将被 @ConfigurationProperties 修饰的类，加载到 Spring Env 中。