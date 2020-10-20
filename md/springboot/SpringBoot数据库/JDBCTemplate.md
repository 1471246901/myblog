# JDBCTemplate

## JDBCTemplate 在springboot 中的自动配置

JDBCTemplate 是一套JDBC模板, 使用了aop技术来解决访问数据库时的大量重复代码. 虽然使用起来不如mybatis那样方便,但是比JDBC要方便得多,而且springboot 提供了 `JDBCTemplate` 的相关自动配置类 `JdbcTemplateAutoConfiguration` 

`JdbcTemplateAutoConfiguration`  源码

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ DataSource.class, JdbcTemplate.class })
@ConditionalOnSingleCandidate(DataSource.class)
@AutoConfigureAfter(DataSourceAutoConfiguration.class)
@EnableConfigurationProperties(JdbcProperties.class)
@Import({ JdbcTemplateConfiguration.class, NamedParameterJdbcTemplateConfiguration.class })
public class JdbcTemplateAutoConfiguration {

}
```

由此可知 `@ConditionalOnClass({ DataSource.class, JdbcTemplate.class })` 在容器中有这两个类时才会进行配置 

在`@AutoConfigureAfter(DataSourceAutoConfiguration.class)`  之后进行配置

 `@EnableConfigurationProperties(JdbcProperties.class)` 将 JdbcProperties 配置信息注入到容器中

通过 `@Import({ JdbcTemplateConfiguration.class, NamedParameterJdbcTemplateConfiguration.class })` 注解向容器中注入对象

进入到 `JdbcTemplateConfiguration.class` 中查看

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnMissingBean(JdbcOperations.class)
class JdbcTemplateConfiguration {

	@Bean
	@Primary
	JdbcTemplate jdbcTemplate(DataSource dataSource, JdbcProperties properties) {
		JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
		JdbcProperties.Template template = properties.getTemplate();
		jdbcTemplate.setFetchSize(template.getFetchSize());
		jdbcTemplate.setMaxRows(template.getMaxRows());
		if (template.getQueryTimeout() != null) {
			jdbcTemplate.setQueryTimeout((int) template.getQueryTimeout().getSeconds());
		}
		return jdbcTemplate;
	}

}
```

`@ConditionalOnMissingBean(JdbcOperations.class)` 在没有 `JdbcOperations` 类时向容器中注入对象

 `jdbcTemplate`  ,  ``jdbcTemplate`   是  `JdbcOperations`  的子类



随意想要使用 `jdbcTemplate`   只需提供 `DataSource` 和 `jdbcTemplate` 的依赖即可

## jdbcTemplate 配置和使用

### 依赖

在springboot 中  使用 `jdbcTemplate`  所需要的依赖

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
<!-- spring-boot-starter-jdbc 当中提供了数据库最基本的依赖,和jdbcTemplate  -->
<!-- jdbcTemplate位于 org.springframework.jdbc.core 包下 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
<!-- mysqljdbc实现类 -->

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.9</version>
        </dependency>
<!-- 阿里巴巴的数据库连接池 -->
```

###  数据库配置

```properties
#   datasource 使用哪种 datasource
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
# url
spring.datasource.url=jdbc:mysql://localhost:3306/race?useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true&useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=root
# 数据库驱动名
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# 方言
spring.datasource.platform: mysql
```

### 创建数据库实体类

```java
public class Player {
    Integer id;
    String name;
    Date birthday;
    Double height;
    String address;
	// getter setter  ...
}
```

### DAO

```java
@Repository
public class PlayerDao {

    @Autowired
    JdbcTemplate jdbcTemplate;

    public int addPlayer(Player player){
        return jdbcTemplate.update("insert into player(id,name,birthday,height,address) values (?,?,?,?,?)"
        ,player.getId(),player.getName(),player.getBirthday(),player.getHeight(),player.getAddress()
        );
    }

    public List<Player> getAllPlayer(){
        return jdbcTemplate.query("select id,name,birthday,height,address from player",
                new BeanPropertyRowMapper<>(Player.class));
    }

}
```

并编写controller 和 service 访问

