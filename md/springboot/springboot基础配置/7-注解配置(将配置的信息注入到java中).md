# 注解配置(使用注解将properties/yml 文件中的配置信息注入到对象中)

## @value注解

可以直接写值

![image-20200727172538519](7-%E6%B3%A8%E8%A7%A3%E9%85%8D%E7%BD%AE(%E5%B0%86%E9%85%8D%E7%BD%AE%E7%9A%84%E4%BF%A1%E6%81%AF%E6%B3%A8%E5%85%A5%E5%88%B0java%E4%B8%AD).assets/image-20200727172538519.png)

可以使用${key}的形式从配置文件中获取值 如:

![img](7-%E6%B3%A8%E8%A7%A3%E9%85%8D%E7%BD%AE(%E5%B0%86%E9%85%8D%E7%BD%AE%E7%9A%84%E4%BF%A1%E6%81%AF%E6%B3%A8%E5%85%A5%E5%88%B0java%E4%B8%AD).assets/clip_image001.png)

 

可以使用#{SPEL语法 表达式}来指定值（      #{11*2}  ）如:

![img](7-%E6%B3%A8%E8%A7%A3%E9%85%8D%E7%BD%AE(%E5%B0%86%E9%85%8D%E7%BD%AE%E7%9A%84%E4%BF%A1%E6%81%AF%E6%B3%A8%E5%85%A5%E5%88%B0java%E4%B8%AD).assets/clip_image002.png)

## @ConfigurationProperties

@ConfigurationProperties 支持将一系类的值注入到对象,集合,map 中

例如:  

![image-20200727172801284](7-%E6%B3%A8%E8%A7%A3%E9%85%8D%E7%BD%AE(%E5%B0%86%E9%85%8D%E7%BD%AE%E7%9A%84%E4%BF%A1%E6%81%AF%E6%B3%A8%E5%85%A5%E5%88%B0java%E4%B8%AD).assets/image-20200727172801284.png)

或者:

![image-20200727173004200](7-%E6%B3%A8%E8%A7%A3%E9%85%8D%E7%BD%AE(%E5%B0%86%E9%85%8D%E7%BD%AE%E7%9A%84%E4%BF%A1%E6%81%AF%E6%B3%A8%E5%85%A5%E5%88%B0java%E4%B8%AD).assets/image-20200727173004200.png)

在对象中注入(对象需要有getset方法)

![image-20200727173732489](7-%E6%B3%A8%E8%A7%A3%E9%85%8D%E7%BD%AE(%E5%B0%86%E9%85%8D%E7%BD%AE%E7%9A%84%E4%BF%A1%E6%81%AF%E6%B3%A8%E5%85%A5%E5%88%B0java%E4%B8%AD).assets/image-20200727173732489.png)

验证:

![image-20200727174129795](7-%E6%B3%A8%E8%A7%A3%E9%85%8D%E7%BD%AE(%E5%B0%86%E9%85%8D%E7%BD%AE%E7%9A%84%E4%BF%A1%E6%81%AF%E6%B3%A8%E5%85%A5%E5%88%B0java%E4%B8%AD).assets/image-20200727174129795.png)

**如果你配置的文件在别的properties 文件中,并不在application.properties文件中**

你可以使用 **@PropertySource("classpath:<span style="color:red;font-size:20px">文件路径</span>")**  注解 标注到相同位置

![image-20200727174638523](7-%E6%B3%A8%E8%A7%A3%E9%85%8D%E7%BD%AE(%E5%B0%86%E9%85%8D%E7%BD%AE%E7%9A%84%E4%BF%A1%E6%81%AF%E6%B3%A8%E5%85%A5%E5%88%B0java%E4%B8%AD).assets/image-20200727174638523.png)

或者 将 **@EnableConfigurationProperties("classpath:<span style="color:red;font-size:20px">文件路径</span>") ** 标注到启动类上



### 另外一种使用@ConfigurationProperties   的方法



使用配置类:

**BookConfig**

```java
@Configuration
public class BookConfig {
	
	
	
	@Bean
	@ConfigurationProperties(prefix = "book")
	public Book createBook() {
		return new Book();
	}

}

```

**Book**

```java
public class Book {
	
	public String name;
	public String author;
	public int price;
    
    //get set ...
}
```

这样也可以将属性注入到对象中

## @value 和@ConfigurationProperties 的区别

`@Value` 注解 和 `@ConfigurationProperties`的区别

| 功能                 | @value                              | @ConfigurationProperties                                     |
| -------------------- | ----------------------------------- | ------------------------------------------------------------ |
| 注入                 | 只能注入一个值                      | 可以注入一组值                                               |
| 松散绑定（松散语法） | 不支持松散绑定                      | 支持松散绑定 last -name == lastName 驼峰命名（ lastName）会自动转化到（last -name） |
| SpEL表达式           | 支持SPEL 语法                       | 不支持SPEL语法                                               |
| JSR303数据校验       | 不支持                              | 支持                                                         |
| 复杂类型封装         | 不支持 只支持一些基本数据类型的输入 | 支持  复杂类型的注入  如 map   对象  list                    |

如果只是引用一些基本的值 用Value 
如果是复杂映射就使用ConfigurationProperties

>   *JSR303**配置文件注入值数据校验：对javabean的值进行校验*  *方法：*
>   *对类名加上注解*  *@Validated*
>   *在对bean内的字段加上校验规则注解*  *如**@E**mail (**校验字符串是否是* *email* *格式**)*
>   *会在注入式进行校验* *确保注入值的正确*



## 传统使用xml 配置bean  @imporresource  (beans.xml)

例如使用传统的 beans.xml  进行注入和配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    ...

</beans>
```

可以使用  @imporresource("classpath:<span style="color:red;font-size:20px">文件路径</span>") 将其注入到容器中

