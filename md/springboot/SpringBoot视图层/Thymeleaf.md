# Thymeleaf 模板

thymeleaf模板是新一代的java模板引擎,专注于HTML,支持HTML原型,既可以让前端工程师查看样式,也可以让后端工程师查看真实效果.

springboot 整合方式步骤:

## pom  依赖

thymeleaf 模板以web-stater 前提 ,提供了thymeleaf-stater 

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

## 配置Thymeleaf

spring boot 为thymeleaf 提供了自动化配置类   ThymeleafAutoConfiguration

```java
package org.springframework.boot.autoconfigure.thymeleaf;

// import .....
/**
 * {@link EnableAutoConfiguration Auto-configuration} for Thymeleaf.
 *
 * @author Dave Syer
 * @author Andy Wilkinson
 * @author Stephane Nicoll
 * @author Brian Clozel
 * @author Eddú Meléndez
 * @author Daniel Fernández
 * @author Kazuki Shimizu
 * @author Artsiom Yudovin
 * @since 1.0.0
 */
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(ThymeleafProperties.class)
@ConditionalOnClass({ TemplateMode.class, SpringTemplateEngine.class })
@AutoConfigureAfter({ WebMvcAutoConfiguration.class, WebFluxAutoConfiguration.class })
public class ThymeleafAutoConfiguration {

```

相观的配置属性在 ThymeleafProperties 类中

```java
/*
 * Copyright 2012-2019 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package org.springframework.boot.autoconfigure.thymeleaf;
/**
 * Properties for Thymeleaf.
 *
 * @author Stephane Nicoll
 * @author Brian Clozel
 * @author Daniel Fernández
 * @author Kazuki Shimizu
 * @since 1.2.0
 */
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {
	private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;
	public static final String DEFAULT_PREFIX = "classpath:/templates/";
	public static final String DEFAULT_SUFFIX = ".html";
	/**
	 * Whether to check that the template exists before rendering it.
	 */
	private boolean checkTemplate = true;
	/**
	 * Whether to check that the templates location exists.
	 */
	private boolean checkTemplateLocation = true;
	/**
	 * Prefix that gets prepended to view names when building a URL.
	 */
	private String prefix = DEFAULT_PREFIX;
	/**
	 * Suffix that gets appended to view names when building a URL.
	 */
	private String suffix = DEFAULT_SUFFIX;
	/**
	 * Template mode to be applied to templates. See also Thymeleaf's TemplateMode enum.
	 */
	private String mode = "HTML";
	/**
	 * Template files encoding.
	 */
	private Charset encoding = DEFAULT_ENCODING;

	/**
	 * Whether to enable template caching.
	 */
	private boolean cache = true;
	/**
	 * Order of the template resolver in the chain. By default, the template resolver is
	 * first in the chain. Order start at 1 and should only be set if you have defined
	 * additional "TemplateResolver" beans.
	 */
	private Integer templateResolverOrder;

	/**
	 * Comma-separated list of view names (patterns allowed) that can be resolved.
	 */
	private String[] viewNames;

	/**
	 * Comma-separated list of view names (patterns allowed) that should be excluded from
	 * resolution.
	 */
	private String[] excludedViewNames;

	/**
	 * Enable the SpringEL compiler in SpringEL expressions.
	 */
	private boolean enableSpringElCompiler;

	/**
	 * Whether hidden form inputs acting as markers for checkboxes should be rendered
	 * before the checkbox element itself.
	 */
	private boolean renderHiddenMarkersBeforeCheckboxes = false;

	/**
	 * Whether to enable Thymeleaf view resolution for Web frameworks.
	 */
	private boolean enabled = true;

```

由此配置可以看出    模板的默认位置在classpath:/templates  ,默认的模板后缀名为 .html , 而且 类中标注了 @ConfigurationProperties(prefix = "spring.thymeleaf")   所以可以在`applicatio.properties` 文件配置.

部分常见配置如下:

```properties
# 是否开启缓存,开发时为false 默认为true
spring.thymeleaf.cache=true
# 检查模板是否存在  默认true
spring.thymeleaf.check-template=true
# 检查模板位置是否存在  默认true
spring.thymeleaf.check-template-location=true
# 模板文件编码  UTF-8
spring.thymeleaf.encoding=utf-8
# 模板文件位置
spring.thymeleaf.prefix=classpath:/templates/
# content-type
spring.thymeleaf.servlet.content-type=text/html
# 模板文件位置
spring.thymeleaf.suffix=.html
```

yml配置

```yml
spring:
  thymeleaf:
    cache: false 模板缓存
    servlet:
      content-type: text/html
//不用配置也可这些都是默认配置
    suffix: .html
    encoding: UTF-8
    prefix: classpath:/templates/
```

## 使用Thymeleaf模板

控制器:

```java
@Controller
public class ThymeleafController {


    @GetMapping("/thymeleaf")
    public ModelAndView testThymeleaf(ModelAndView mav){
        //这里的返回值 ModelAndView 既可以当做参数引入,又可以自己new
        //返回值也可以是 String
        //redirect:表示重定向到一个地址（提交表单最好用重定向免得表单重复提交）
        //forward:表示转发到一个地址
        // "/" 代表当前项目路径
        //  setViewName();   在使用modelandview 的时候也可以重定向和转发
        mav.addObject("obj","obj to controller");
        mav.setViewName("/Thymeleaf");
        return mav;
    }
}

```

模板 html 注意引入html模板的时候添加 名称空间 `xmlns:th="http://www.thymeleaf.org"`

```html
<!DOCTYPE html>
<html lang="ch" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Thymeleaf</title>
</head>
<body>
<p>this Thymeleaf</p>

<p th:text="${obj}"></p>
</body>
</html>
```
