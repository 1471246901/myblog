# SpringBoot测试

spring boot 的测试所需类库

maven依赖

```xml
<dependecy>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
</dependecy>
```

spring boot针对mvc的测试用例

在test包创建测试用例

类注解@RunWith(SpringRunner.class) 使用`SpringRunner` 来执行测试用例

类注解@SpringBootTest 表明是springboot 的测试类

方法注解@before  标注的方法会在测试方法执行前被执行  springboot 2.x以后 使用 @BeforeEach 代替

方法注解@Test 标注的方法就是测试方法

在测试类中可以直接使用@Autowired 来注入容器的对象

常用的   WebApplicationContext  ...

#### 使用MockMvc来测试请求

```java
private MockMvc mockMvc;
```

初始化`MockMvc`

```java
@BeforeEach 
public void setup(){
	mockMvc = MockMvcBuilders.webAppContextSetup(webAppContext上下文).build();
}
```

使用`MockMvc`进行模拟请求

```java
    @Test
    public void mockMvcTest() throws Exception {
        //perform 代表一次请求
        //MockMvcRequestBuilders.get("") 代表get请求
        //contentType 代表编码和类型
        //mockMvc.perform(MockMvcRequestBuilders.get("/person/1").param("",""))      param("","") 添加参数
        //	    .andExpect(MockMvcResultMatchers.status().isOk())   对状态码进行检查 是否是200
        //	    .andExpect(MockMvcResultMatchers.content().contentType(MediaType.APPLICATION_JSON))   对返回类型进行检查
        //	    .andExpect(MockMvcResultMatchers.jsonPath("$.person.name").value("Jason"));   对返回的json进行jsonPath("$.person.name")取值 value("Jason")判断

        mockMvc.perform(MockMvcRequestBuilders.get("").contentType(MediaType.APPLICATION_JSON))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.jsonPath("$.person.name").value("Jason"));
    }
```

#### jsonpath 参数

[jsonpath ](https://github.com/json-path/JsonPath)   可查看 `MockMvcResultMatchers.jsonPath("$.person.name")` 的相关参数

部分翻译

| 算子          | 描述                                       |
| ------------- | ------------------------------------------ |
| `$`           | 要查询的根元素。这将启动所有路径表达式。   |
| `@`           | 筛选器谓词正在处理的当前节点。             |
| `*`           | 通 配 符。任何需要名称或数字的地方都可用。 |
| `..`          | 深度扫描。名称需要的任何位置都可用。       |
| `.`           | 点点化子                                   |
| `['' (, '')]` | 括号内带名的儿童或儿童                     |
| `[ (, )]`     | 数组索引或索引                             |
| `[start:end]` | 阵列切片运算符                             |
| `[?()]`       | 筛选器表达式。表达式必须计算为布尔值。     |

方法

| 功能     | 描述                     | 输出    |
| -------- | ------------------------ | ------- |
| min()    | 提供数字数组的最小值     | Double  |
| max()    | 提供数字数组的最大值     | Double  |
| avg()    | 提供数字数组的平均值     | Double  |
| stddev() | 提供数字数组的标准偏差值 | Double  |
| length() | 提供数组的长度           | Integer |
| sum()    | 提供数字数组的总和值     | Double  |