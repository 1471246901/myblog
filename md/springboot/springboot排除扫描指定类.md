# spring boot排除扫描类

**默认下面描述的类都在一个包下面。**

-   第一步我们新建一个应用启动的类，一个类用来充当Configuration，为了能明显的感知到其到底有没生效，我编写如下:

```java
@SpringBootApplication
public class Test  {
    public static void main(String[] args) {
        new SpringApplication(Test.class).run(args);
    }
}
123456
@Configuration
public class MyConfig {

    @Bean
    public BeanPostProcessor beanPostProcessor() {
        System.out.println("初始化了 bean BeanPostProcessor");
        return new BeanPostProcessor() {
            @Override
            public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
                System.out.println("加载了bean " + beanName);
                return bean;
            }

            @Override
            public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
                return bean;
            }
        };
    }
}
1234567891011121314151617181920
```

我们可以启动应用发现输出

```java
初始化了 bean BeanPostProcessor
加载了bean org.springframework.context.event.internalEventListenerProcessor
加载了bean org.springframework.context.event.internalEventListenerFactory
加载了bean org.springframework.boot.autoconfigure.AutoConfigurationPackages
加载了bean org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration
加载了bean org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration
加载了bean objectNamingStrategy
加载了bean mbeanServer
加载了bean mbeanExporter
加载了bean org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration
加载了bean spring.info-org.springframework.boot.autoconfigure.info.ProjectInfoProperties
加载了bean org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration
加载了bean org.springframework.boot.autoconfigure.web.WebClientAutoConfiguration
12345678910111213
```

说明被@Configuration 修饰的类MyConfig本身被扫描了。

## 方法1：用exclude指明明确要排除的类

```java
@SpringBootApplication
@ComponentScan(excludeFilters = {@ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {MyConfig.class})})
public class Test  {
    public static void main(String[] args) {
        new SpringApplication(Test.class).run(args);
    }
}
1234567
```

用ComponentScan的excludeFilters属性，可以明确排除调需要扫描的类。

但是这里存在一个问题，如果要排除的类比较多，那这里会看起来很冗余。我们可以采用第二种方式。注解排除。

## 方法2 ： 用注解方式排除

**有问题**

```java
//启动类加入表示排除IgnoreScan 注解标注的类
@ComponentScan(excludeFilters={@ComponentScan.Filter(type = FilterType.ANNOTATION,classes = IgnoreScan.class)})
```

```java
public @interface IgnoreScan {
}
```

新建此注解。
在需要忽略的类上添加：

```java
@Configuration
@IgnoreScan
public class MyConfig {

    @Bean
    public BeanPostProcessor beanPostProcessor() {
        System.out.println("初始化了 bean BeanPostProcessor");
        return new BeanPostProcessor() {
            @Override
            public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
                System.out.println("加载了bean " + beanName);
                return bean;
            }

            @Override
            public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
                return bean;
            }
        };
    }
}
```

这样即可排除掉不被扫描了。

## 方法3 ：

第三种方式实现org.springframework.core.type.filter.TypeFilter，然后也是ComponentScan去排除指定的注解。网上写的好的文章很多，这里不多说了。

>    转载 自 [micro_hz](https://blog.csdn.net/micro_hz)