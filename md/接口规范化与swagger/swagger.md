# ![image-20200506193236414](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200506193236414.png)swagger 

### swagger简介

相信无论是前端还是后端开发，都或多或少地被接口文档折磨过。前端经常抱怨后端给的接口文档与实际情况不一致。**后端又觉得编写及维护接口文档会耗费不少精力，经常来不及更新**。其实无论是前端调用后端，还是后端调用后端，都期望有一个好的接口文档。但是这个接口文档对于程序员来说，就跟注释一样，经常会抱怨别人写的代码没有写注释，然而自己写起代码起来，最讨厌的，也是写注释。所以仅仅只通过强制来规范大家是不够的，随着时间推移，版本迭代，接口文档往往很容易就跟不上代码了。

-   号称世界上最流行的api框架
-   RestFul api 文档在线自动生成工具 `文档与api同步更新`
-   直接运行可测试在线api接口
-   支持多种语言

[swagger 官网](https://swagger.io/)

### spring boot 中使用swagger

*spring boot 项目需要是web项目*

#### maven依赖

<img src="https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200506194127030.png" alt="image-20200506194127030" style="zoom:67%;" />

```xml
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
<!-- swagger 核心包 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>

```

<img src="https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200506194241914.png" alt="image-20200506194241914" style="zoom:67%;" />

```xml
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
<!-- swagger的ui界面 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>

```

#### 集成swagger

新建config类,标注注解 `@Configuration` 

开启swagge2 类上标注注解 `@EnableSwagger2`

测试运行 访问 [项目地址/swagger-ui.html]() 查看

#### 配置swagger

##### 配置swagger 示例 `Docket`

在 swaggerconfiguration类 注入`Docket` 

```java
@Configuration
@EnableSwagger2
public class SwaggerConfiguration {
    @Bean
    public Docket docket(){
        return  new Docket(DocumentationType.SWAGGER_2);
    }
}
```

##### `Docket` 中可以配置的属性  可以使用链式修改值

```java
public Docket(DocumentationType documentationType) {
        this.apiInfo = ApiInfo.DEFAULT;
        this.groupName = "default";
        this.enabled = true;
        this.genericsNamingStrategy = new DefaultGenericTypeNamingStrategy();
        this.applyDefaultResponseMessages = true;
        this.host = "";
        this.pathMapping = Optional.absent();
        this.apiSelector = ApiSelector.DEFAULT;
        this.enableUrlTemplating = false;
        this.vendorExtensions = Lists.newArrayList();
        this.documentationType = documentationType;
    }
```

##### 配置ApiInfo

```java
    /**
     * 配置ApiInfo
     */
    private ApiInfo apiInfo(){
        return new ApiInfo("标题", "描述", "版本", "组织服务地址", new Contact("姓名","地址","e-mail"), "开源版本号  默认", "开源版本号 默认", new ArrayList());
    }
```

```java
	//ApiInfo类中可配置的属性
	public static final Contact DEFAULT_CONTACT = new Contact("name", "url", "e-mail");
    public static final ApiInfo DEFAULT;
    private final String version;  //版本信息
    private final String title;  //标题
    private final String description;  //描述
    private final String termsOfServiceUrl;   //组织服务地址
    private final String license;  //开源版本号
    private final String licenseUrl;  //开源版本号
    
```

启动后

![image-20200506202344091](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200506202344091.png)

##### 配置swagger 扫描指定接口 

默认扫描所有Controller

 **docket.select() ....... .build() 链式编程** 

select  和 build 中只有两个方法

```java
.apis()
.paths()
```

```java
@Bean
    public Docket docket(){
        return  new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                //RequestHandlerSelectors 配置扫描的方式
                //basePackage  指定要扫描的包
                .apis(RequestHandlerSelectors.basePackage("包名"))
                //扫描全部
                //.apis(RequestHandlerSelectors.any())
                //都不扫描
                //.apis(RequestHandlerSelectors.none())
                //扫描类上的注解  含有注解的扫描   需要注解的class
                //.apis(RequestHandlerSelectors.withClassAnnotation(RestController.class))
                //扫描方法上的注解
                //.apis(RequestHandlerSelectors.withMethodAnnotation(RequestMapping.class))

                //paths()过滤路径
                //PathSelectors.ant("/xxxx/**")  只扫描xxxx下的
                //.paths(PathSelectors.ant("/xxxx/**"))
                //PathSelectors.regex("正则")  基于正则过滤
                //.paths(PathSelectors.regex("正则"))
                .build();
    }
```

##### 配置是否启用扫描

Docket..enable( boolean ) 

```java
@Configuration
@EnableSwagger2
public class SwaggerConfiguration {
    @Bean
    public Docket docket(){
        return  new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                //配置是否扫描
                .enable(true)
```

**希望再生产环境中关闭,在开发\测试环境中打开**

spring boot 多环境配置参见spring boot 脚手架搭建

1.  获取当前环境

    可以使用@value 或者 @ConfigurationProperties 将配置文件的值注入到属性中

    或者

    使用  Environment  `org.springframework.core.env.Environment`  来判断是否处于某个环境中

    ```java
    @Configuration
    @EnableSwagger2
    public class SwaggerConfiguration {
        @Bean
        public Docket docket(Environment environment){  //将environment 对象拿到
            //判断是否在某种(多种)环境中
            boolean enableSwagger = environment.acceptsProfiles(Profiles.of("dev",""));
    
            return  new Docket(DocumentationType.SWAGGER_2)
                    .apiInfo(apiInfo())
                    //配置是否扫描
                    .enable(enableSwagger)
                    .select()
                    //RequestHandlerSelectors 配置扫描的方式
                    //basePackage  指定要扫描的包
    ```

    

##### 配置分组

使用docket..groupName("谷世玉") 来设置组 , 可以设置多个组`添加多个Docket示例即可`

```java
@Configuration
@EnableSwagger2
public class SwaggerConfiguration {
    @Bean
    public Docket docket(Environment environment){//将environment 对象拿到
        //判断是否在某种(多种)环境中
        boolean enableSwagger = environment.acceptsProfiles(Profiles.of("dev",""));

        return  new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                //配置是否开启swagger
                .enable(enableSwagger)
                .groupName("谷世玉")
```

![image-20200506211836391](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200506211836391.png)

##### 配置接口注释

Controller 中的注释

| 注解                                                         | 功能                                   |
| :----------------------------------------------------------- | -------------------------------------- |
| @ApiOperation(value = "示例", notes = "invokePost说明", httpMethod = "POST") | 注释在方法上,用于说明方法功能          |
| @ApiParam(name="传入对象",value="传入json格式",required=true) | 注释在controller中的参数上用于标明参数 |

传入实体类中的注释

| 注解                                                         | 功能                    |
| ------------------------------------------------------------ | ----------------------- |
| @ApiModel(value="演示类",description="请求参数类" )          | 标在类上 注释整个返回类 |
| @ApiModelProperty(value = "defaultStr",example="mockStrValue") | 标在字段上注释字段      |
|                                                              |                         |

![image-20200506214156294](D:/Typora/data/image/image-20200506214156294.png)



