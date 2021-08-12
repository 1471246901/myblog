# mybatisPlus -SpringBoot

本文章是程序汪的朋友犬小哈分享

今天给大家带来一篇如何在 Spring Boot 中快速整合 Mybatis-Plus 的入门教程，希望对大家有所帮助。

## 一、先说说 MyBatis

在说 MyBatis-Plus 之前，小哈带小伙伴们先了解下什么是 MyBatis：

MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

> PS: 下面 Mybatis-Plus 的介绍以及特性讲解部分摘自官网，毕竟官网的说明更权威。

## 二、MyBatis-Plus 和 Mybatis 有啥区别？

MyBatis-Plus (opens new window)（简称 MP）是一个 MyBatis (opens new window)的增强工具，就像 IPhone手机一般都有个 plus 版本一样，它在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

MyBatis-Plus 的愿景是成为 MyBatis 最好的搭档，就像魂斗罗中的 1P、2P，基友搭配，效率翻倍。

## 三、MyBatis-Plus 的特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- **内置性能分析插件**：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

## 四、MyBatis-Plus 支持哪些数据库 ？

简单来说，MyBatis 支持的数据库，MyBatis-Plus 都支持, 如 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等。

## 五、框架结构

![图片](mybatisplus-springboot.assets/640)MyBatis-Plus 框架结构

## 六、快速开始

### 6.1 测试数据准备

开始之前，需要准备一个数据库环境，本教程以操作 MySQL 数据库为例，所以你要准备如下：

- 安装 MySQL 数据库；
- 造一批测试数据；

#### 6.1.1 安装 MySQL 数据库环境；

MySQL 数据库安装在本地或者云上均可~

至于如何安装，大家可自行搜索一下安装步骤就行，非常简单, 如果想用 Docker 安装 MySQL 环境，可参考小哈之前的一篇文章：**《**一文教您通过 Docker 快速搭建各种测试环境(Mysql, Redis, ES, MongoDB) | 建议收藏**》**

#### 6.1.2 准备一点测试数据；

新建一个名为 `test` 的测试数据库，创建一张用户表，Schema 建表脚本如下：

```
DROP TABLE IF EXISTS user;

CREATE TABLE `user` (
  `id` bigint(20) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `name` varchar(30) NOT NULL DEFAULT '' COMMENT '姓名',
  `age` int(11) NULL DEFAULT NULL COMMENT '年龄',
  `gender` tinyint(2) NOT NULL DEFAULT 0 COMMENT '性别，0：女 1：男',
  PRIMARY KEY (`id`)
) COMMENT = '用户表';
```

再插入一些测试数据：

```
DELETE FROM user;

INSERT INTO user (name, age, gender) VALUES
('犬小哈1', 16, 0),
('犬小哈2', 18, 1),
('犬小哈3', 20, 0),
('犬小哈4', 28, 1),
('犬小哈5', 30, 0);
```

### 6.2 新建一个 Spring Boot Demo 项目

数据库准备好了，我们新建一个 Spring Boot 项目来讲解如何使用 Mybatis-Plus，可以参考小哈之前发布的文章：**《**这可能是史上最易懂的 Spring Boot 入门教程**》**。

测试项目目录结构如下：

![图片](mybatisplus-springboot.assets/640-20210812173900162)

结构

> 全新的 `MyBatis-Plus` 3.0 版本基于 JDK8，提供了 `lambda` 形式的调用，所以安装集成 Mybatis-Plus 3.0 要求如下：
>
> - JDK 8+；
> - Maven or Gradle, 本文以 maven 作为版本管理工具；

### 6.3 添加依赖

新建好 Spring Boot 项目后，在 `pom.xml` 文件中添加以下依赖：

```
<!-- mybatis-plus 依赖 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.2</version>
</dependency>

<!-- 单元测试依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<!-- lombok 依赖 -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.16.10</version>
</dependency>

<!-- mysql 依赖 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

> **PS: 小哈在写这篇文章的时候，MyBatis-Plus 的最新版本为 `3.4.2`。**

> 警告：引入 `MyBatis-Plus` 之后请不要再次引入 `MyBatis` 以及 `MyBatis-Spring`，以避免因版本差异导致的问题。

### 6.4 添加配置

接下来，在 applicaiton.yml 配置文件中添加 MySQL 数据库的相关配置：

```
# 数据库配置
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/test?characterEncoding=utf-8
    username: root
    password: qwe123
```

配置好数据库驱动、连接、用户名和密码。

然后，在  Spring Boot 启动类中添加  `@MapperScan` 注解，配置好 Mapper 文件夹的全路径，作用是为了 mybatis-plus 能够扫描到：

```
@SpringBootApplication
@MapperScan("com.quanxiaoha.mybatisplusdemo.mapper")
public class MybatisPlusDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(MybatisPlusDemoApplication.class, args);
    }

}
```

### 6.5 添加实体类

前面已经建好了用户表，下面编写用户表对应的实体类 `User.java` :

```
@Data
@Builder
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String name;
    private Integer age;
    private Integer gender;
}
```

`@TableId(type = IdType.AUTO)` 注解指定了字段 `id` 为表的主键，同时指定主键为自增类型。

### 6.6 添加 Mapper 类

编写 Mapper 类 `UserMapper.java` :

```
public interface UserMapper extends BaseMapper<User> {
    
}
```

编写的接口需要继承 `BaseMapper` 接口，该接口定义了一些通用的操作数据库方法,  单表大部分 CRUD 操作都能直接搞定，相比原生的 mybatis，效率提高了很多：

```
public interface BaseMapper<T> extends Mapper<T> {
    int insert(T entity);

    int deleteById(Serializable id);

    int deleteByMap(@Param("cm") Map<String, Object> columnMap);

    int delete(@Param("ew") Wrapper<T> queryWrapper);

    int deleteBatchIds(@Param("coll") Collection<? extends Serializable> idList);

    int updateById(@Param("et") T entity);

    int update(@Param("et") T entity, @Param("ew") Wrapper<T> updateWrapper);

    T selectById(Serializable id);

    List<T> selectBatchIds(@Param("coll") Collection<? extends Serializable> idList);

    List<T> selectByMap(@Param("cm") Map<String, Object> columnMap);

    T selectOne(@Param("ew") Wrapper<T> queryWrapper);

    Integer selectCount(@Param("ew") Wrapper<T> queryWrapper);

    List<T> selectList(@Param("ew") Wrapper<T> queryWrapper);

    List<Map<String, Object>> selectMaps(@Param("ew") Wrapper<T> queryWrapper);

    List<Object> selectObjs(@Param("ew") Wrapper<T> queryWrapper);

    <E extends IPage<T>> E selectPage(E page, @Param("ew") Wrapper<T> queryWrapper);

    <E extends IPage<Map<String, Object>>> E selectMapsPage(E page, @Param("ew") Wrapper<T> queryWrapper);
}
```

### 6.7 开始使用

添加测试类，对用户表进行增删改查操作, 测试代码如下：

```
@SpringBootTest
class MybatisPlusDemoApplicationTests {

    @Autowired
    private UserMapper userMapper;

    /**
     * 查询数据
     */
    @Test
    public void testSelectUser() {
        System.out.println(("----- 开始测试 mybatis-plus 查询数据 ------"));
        //  selectList() 方法的参数为 mybatis-plus 内置的条件封装器 Wrapper，这里不填写表示无任何条件，全量查询
        List<User> userList = userMapper.selectList(null);

        userList.forEach(System.out::println);
    }

    /**
     * 新增一条数据
     */
    @Test
    public void testInsertUser() {
        System.out.println(("----- 开始测试 mybatis-plus 插入数据 ------"));
        User user = User.builder()
                .name("犬小哈教程 www.quanxiaoha.com")
                .age(30)
                .gender(1)
                .build();

        userMapper.insert(user);
    }

    /**
     * 删除数据
     */
    @Test
    public void testDeleteUser() {
        System.out.println(("----- 开始测试 mybatis-plus 删除数据 ------"));
        // 根据主键删除记录
        userMapper.deleteById(1);

        // 根据主键批量删除记录
        userMapper.deleteBatchIds(Arrays.asList(1, 2));
    }

    /**
     * 更新数据
     */
    @Test
    public void testUpdateUser() {
        System.out.println(("----- 开始测试 mybatis-plus 更新数据 ------"));
        User user = User.builder()
                .id(1L)
                .name("犬小哈教程 www.quanxiaoha.com")
                .build();

        userMapper.updateById(user);
    }
}
```

## 七、结语

本文简单介绍了在 Spring Boot 中如何快速整合集成 Mybatis-Plus, 同时在本地搭建了一个 Mysql 环境，演示了如何通过 Mybatis-Plus 做一些简单的增删改查（CRUD）操作，其他更丰富的功能，小伙伴们可以参考官网 https://baomidou.com/guide。

## Ref

- https://mybatis.org/mybatis-3/zh/index.html
- https://baomidou.com/guide