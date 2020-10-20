



# 返回json数据

## jackson 默认实现

JSON是目前主流的前后端数据传输方式,springMVC 中使用了详细转换器 `HttpMessageConverter` 对JSON 的转化提供了很友好的支持,在springboot中添加web starter 即已经默认加入了 jackson-databind 作为json处理器

Jackson是`spring-boot-starter-json`依赖中的一部分，`spring-boot-starter-web`中包含`spring-boot-starter-json`。也就是说，当项目中引入`spring-boot-starter-web`后会自动引入`spring-boot-starter-json`。

##### jackson的maven依赖

```xml
<dependency> 
<groupId>com.fasterxml.jackson.core</groupId> 
<artifactId>jackson-databind</artifactId> 
<version>2.9.1</version> 
</dependency>
```

##### 实体类添加注解

```java
@Data
public class Laptop {

    @JsonProperty("型号")
    private String model;

    @JsonProperty("速度G")
    private Double speed;

    @JsonProperty("内存G")
    private Integer ram;

    @JsonProperty("硬盘")
    private Double hd;

    @JsonProperty("屏幕")
    private Double screen;

    @JsonProperty("价格")
    private Integer price;

}
```

##### 此时返回的结果

```json
{"code":20000,"message":"1100OfLaptop","success":true,"data":{"laptop":{"型号":"1100","速度G":123.0,"内存G":23,"硬盘":0.67,"屏幕":11.0,"价格":3499}}}
```

## 自定义返回的视图@JsonView (分组返回)

有时候我们不希望一个值或者多个值返回给前台,但是另一个请求又希望把这个值返回给前台

所以就需要json视图

创建返回的接口标识(仅仅是作为标识,可以继承)

![image-20200512171123745](%E8%BF%94%E5%9B%9Ejson%E6%95%B0%E6%8D%AE.assets/image-20200512171123745.png)

```java
public interface LaptopOutputGroupA {}
public interface LaptopOutputGroupB {}
```

在controller上标注@JsonView(视图接口.class)

```java
@RequestMapping("/testResult")
    @JsonView({LaptopOutputGroupA.class})
    public R testResult(){
        Laptop l = testmybatisService.selectLaptop("1100");
        return R.ok().data("laptop",l).message("1100OfLaptop");
    }
```

在对象的get方法(属性)上标注@JsonView(在什么视图的显示的接口.class)  不写默认全匹配

```java
    @JsonView(LaptopOutputGroupA.class)
    @JsonProperty("速度G")
    private Double speed;
```

## 控制返回json的规则

| **注解**              | **用法**                                                     |
| --------------------- | ------------------------------------------------------------ |
| **@JsonProperty**     | 用于属性，把属性的名称序列化时转换为另外一个名称。示例： @JsonProperty("birth_ d ate") private Date birthDate; |
| **@JsonFormat**       | 用于属性或者方法，把属性的格式序列化时转换成指定的格式。示例：@JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm") public Date getBirthDate() |
| @JsonPropertyOrder    | 用于类， 指定属性在序列化时 json 中的顺序 ， 示例：@JsonPropertyOrder({ "birth_Date", "name" })public class Person |
| @JsonCreator          | 用于构造方法，在传入参数和bean参数不相同时使用 ,和 @JsonProperty 配合使用，适用有参数的构造方法。 示例： @JsonCreator public Person(@JsonProperty("name")String name) {…}  [用法](https://blog.csdn.net/u012326462/article/details/83020222) |
| @JsonAnySetter        | 用于属性或者方法，设置未反序列化的属性名和值作为键值存储到 map 中 @JsonAnySetter public void set(String key, Object value) { map.put(key, value); }[用法](https://blog.csdn.net/u012326462/article/details/83020222) |
| @JsonAnyGetter        | 用于方法 ，获取所有未序列化的属性 public Map<String, Object> any() { return map; } |
| **@JsonIgnore**       | 在json序列化时将java bean中的一些属性**忽略掉**，序列化和反序列化都受影响。 |
| @JsonIgnoreProperties | 和 @JsonIgnore 的作用相同，都是告诉 Jackson 该忽略哪些属性，不同之处是 @JsonIgnoreProperties 是**类级别**的，并且可以同时指定多个属性。@JsonIgnoreProperties(value = {"age","color"}) |
| @JsonInclude          | @JsonInclude**(JsonInclude.Include.NON_NULL)**  这个注解表示，如果**值为null，则不返回**，还可以在类上添加这个注释，当实体类与json互相转换的时候，**属性值为null的不参与序列化**。 |

## 自定义转换器  

### Gson  google

依赖:

去除jackson-databind的依赖,引入Gson的依赖

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
			<exclusions>
				<exclusion>
					<groupId>com.fasterxml.jackson.core</groupId>
					<artifactId>jackson-databind</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>com.google.code.gson</groupId>
			<artifactId>gson</artifactId>
		</dependency>
		
```

spring boot 中默认提供了`GsonHttpMessageConvertersConfiguration` (`org.springframework.boot.autoconfigure.http;` 包下 )  ,因此Gson依赖添加成功后,可以像使用jackson那样使用Gson,但是在使用Gson 进行格式化的时候不能像jsckson那样对日期进行格式化,所以想要自定义日期格式化 n那么我们就需要提供自己的`HttpMessageConverter` 

源码:  `GsonHttpMessageConvertersConfiguration` 

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(Gson.class)
class GsonHttpMessageConvertersConfiguration {

	@Configuration(proxyBeanMethods = false)
	@ConditionalOnBean(Gson.class)
	@Conditional(PreferGsonOrJacksonAndJsonbUnavailableCondition.class)
	static class GsonHttpMessageConverterConfiguration {

		@Bean
		@ConditionalOnMissingBean
		GsonHttpMessageConverter gsonHttpMessageConverter(Gson gson) {
			GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
			converter.setGson(gson);
			return converter;
		}

	}
```

其中 @ConditionalOnMissingBean  是当容器中没有对象的时候才进行注入,那么我们就可以自己注入一个替代它

新建GsonConfig 配置类

```java
@Configuration
public class GsonConfig {


    @Bean
    public GsonHttpMessageConverter gsonHttpMessageConverter() {
        GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
        GsonBuilder gsonBuilder = new GsonBuilder();
        gsonBuilder.setDateFormat("yyyy-MM-dd");
        //gson解析的时候忽略 'PROTECTED' 修饰的字段
        gsonBuilder.excludeFieldsWithModifiers(Modifier.PROTECTED);
        Gson gson = gsonBuilder.create();
        converter.setGson(gson);
        return converter;
    }
}
```

### fastjson Alibaba

json的解析速度最快

依赖: 排除jackson 引用fastjson   

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
			<exclusions>
				<exclusion>
					<groupId>com.fasterxml.jackson.core</groupId>
					<artifactId>jackson-databind</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.2.47</version>
		</dependency>
```

而且`fastjson` 需要指定 version  

springboot没有为fastjson注入 `HttpMessageConverter`  所以我们需要自己注入

```java
@Configuration
public class FastjsonConfig {

	
	
	@Bean
	public FastJsonHttpMessageConverter fastJsonHttpMessageConverter() {
		
		FastJsonHttpMessageConverter Converter = fastJsonHttpMessageConverter();
		FastJsonConfig fastJsonConfig = new FastJsonConfig();
		fastJsonConfig.setDateFormat("yyyy-MM-dd");  //日期格式
		fastJsonConfig.setCharset(Charset.forName("UTF-8"));   //数据编码
		fastJsonConfig.setSerializerFeatures(
				SerializerFeature.WriteClassName,   //输出类名
				SerializerFeature.WriteMapNullValue,  //输出value为null的属性
				SerializerFeature.PrettyFormat,  //生成json格式化
				SerializerFeature.WriteNullListAsEmpty, //空list 输出[]  而不是null
				SerializerFeature.WriteNullStringAsEmpty //空字符串输出""  而不是null
				);
		Converter.setFastJsonConfig(fastJsonConfig);
		return Converter ;
	}
}

```

FastJsonHttpMessageConverter 输入完成之后还需要配置一下响应编码,否则返回的JSOn中文会乱码

application.properties 中配置

```
# 是否对HTTP响应强制对配置的字符集进行编码。
spring.http.encoding.force-response=true
```

另一种方法

实现WebMvcConfigurer接口 可以通过接口定义的方法对springmvc进行自定义配置,这里添加HttpMessageConverter也可以使用这种方法.

自定义WebMvcConfig类继承WebMvcConfigurer 接口

```java
@Configuration
public class WenMvcConfig implements WebMvcConfigurer {

	@Override
	public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
		// TODO Auto-generated method stub
		FastJsonHttpMessageConverter Converter = new FastJsonHttpMessageConverter();
		FastJsonConfig fastJsonConfig = new FastJsonConfig();
		fastJsonConfig.setDateFormat("yyyy-MM-dd");
		fastJsonConfig.setCharset(Charset.forName("UTF-8"));
		fastJsonConfig.setSerializerFeatures(
				SerializerFeature.WriteClassName,
				SerializerFeature.WriteMapNullValue,
				SerializerFeature.PrettyFormat,
				SerializerFeature.WriteNullListAsEmpty,
				SerializerFeature.WriteNullStringAsEmpty
				);
		Converter.setFastJsonConfig(fastJsonConfig);
		converters.add(Converter);
    }
}
```

