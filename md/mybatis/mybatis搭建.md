# mybatis搭建

#### maven配置相应的依赖包

```xml
		<!--mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.5</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.6</version>
        </dependency>
```

#### 添加SqlMapConfig.xml文件

<img src="https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200427204952281.png" alt="image-20200427204952281" style="zoom:67%;" />

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <!-- 和spring整合后 environments配置将废除-->
    <environments default="development">
        <environment id="development">
            <!-- type="JDBC" 代表使用JDBC的提交和回滚来管理事务 -->
            <transactionManager type="JDBC" />

            <!-- mybatis提供了3种数据源类型，分别是：POOLED,UNPOOLED,JNDI -->
            <!-- POOLED 表示支持JDBC数据源连接池 -->
            <!-- UNPOOLED 表示不支持数据源连接池 -->
            <!-- JNDI 表示支持外部数据源连接池 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/mytest?characterEncoding=utf-8" />
                <property name="username" value="root" />
                <property name="password" value="root" />
            </dataSource>

        </environment>
    </environments>

    <mappers>
        <mapper resource="----.xml"/>
    </mappers>

</configuration>

```

#### 编写实体类

```properties
......实体类
```

#### 创建对应xml文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace：命名空间，做sql隔离 -->
<mapper namespace="laptop">
    <!--
    id：sql语句唯一标识
    parameterType：指定传入参数类型（pojo类中对应的类型，不是数据库中的类型）
    resultType：返回结果集类型
    #{}：占位符，如果传入的类型是基本类型（string，long，double，int，boolean，float等），那么#{}中变量名称可以任意
     -->
    <select id="findLaptopByModel" parameterType="java.lang.String" resultType="gushiyu.po.laptop">
        SELECT * FROM laptop WHERE model=#{model}
    </select>

    <!--
    如果返回的结果为集合，resultType中也是配置为集合中泛型的类型，即resultType="cn.itheima.pojo.User"
    ${}：拼接符，如果传入的类型是基本类型（string，long，double，int，boolean，float等），那么${}中变量名称必须是value
     -->
    <select id="findLaptopByPrice" parameterType="java.lang.Double" resultType="gushiyu.po.laptop">
        SELECT * FROM laptop WHERE price = #{price}
    </select>

    <!--如果传入的是pojo对象类型，则#{}中变量名称必须是pojo中对应的属性.属性.属性......
    如果要返回数据库自增主键，可以使用SELECT LAST_INSERT_ID()-->
    <insert id="insertLaptop" parameterType="gushiyu.po.laptop">
        <!-- 执行SELECT LAST_INSERT_ID()数据库函数，返回自增的主键
            keyProperty：将返回的主键放入传入的参数的Id中保存（保存到user对象中的id属性）
            order：当前函数相对于insert语句的执行顺序，在insert前执行用BEFORE，在insert后执行用AFTER
            resultType：id的类型，也就是keyProperty中属性类型
         -->
        <selectKey keyProperty="id" order="AFTER" resultType="java.lang.Integer">
            SELECT LAST_INSERT_ID()
        </selectKey>
        INSERT INTO user (username,birthday,sex,address) VALUES (#{username},#{birthday},#{sex},#{address})
    </insert>


    <insert id="insertLaptop" parameterType="gushiyu.po.laptop">
        INSERT INTO laptop (model,speed,ram,hd,screen,price) VALUES (#{model},#{speed},#{ram},#{hd},#{screen},#{price})
    </insert>

    <delete id="deleteLaptopByModel" parameterType="java.lang.String">
        DELETE FROM laptop WHERE model=#{model}
    </delete>

    <update id="updateLaptopPriceByModel" parameterType="gushiyu.po.laptop">
        UPDATE laptop SET  price=#{price} WHERE model=#{model}
    </update>
</mapper>
```

#### 在sqlmapconfig.xml中配置laptop.xml

![image-20200427212409121](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200427212409121.png)

#### 测试

```java
import gushiyu.po.Laptop;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.jupiter.api.Test;

import java.io.InputStream;
import java.util.List;

public class TestMybatis {


    @Test
    public void findLaptopByModel() throws Exception {
        String resource = "SqlMapConfig.xml";
        // 通过流将核心配置文件读取进来
        InputStream inputStream = Resources.getResourceAsStream(resource);
        // 通过核心配置文件创建会话工厂
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);

        // 通过会话工厂创建会话
        SqlSession session = factory.openSession();
        Laptop laptop = session.selectOne("laptop.findLaptopByModel", "2001");
        System.out.println(laptop);
    }

    @Test
    public void findLaptopByPrice() throws Exception {
        String resource = "SqlMapConfig.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);

        SqlSession session = factory.openSession();
        List<Laptop> list = session.selectList("laptop.findLaptopByPrice", 3499.0);
        for (Laptop l:list) {
            System.out.println(l);
        }
    }

    @Test
    public void testInsertUser() throws Exception {
        String resource = "SqlMapConfig.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);

        SqlSession session = factory.openSession();
        Laptop laptop = new Laptop();
        laptop.setHd(3.0);
        laptop.setModel("3001");
        laptop.setPrice(3499);
        laptop.setRam(23);
        laptop.setScreen(15.6);
        laptop.setSpeed(2000.0);

        int rest = session.insert("laptop.insertLaptop", laptop);
        // mybatis中事务默认是开启的，所以在mybatis中不需要显式开启事务，只需要显式提交事务
        // 提交事务
        session.commit();
        System.out.println(rest);
    }

    @Test
    public void testDeleteUserById() throws Exception {
        String resource = "SqlMapConfig.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);

        SqlSession session = factory.openSession();
        session.delete("laptop.deleteLaptopByModel", "3001");
        session.commit();
    }

    @Test
    public void updateLaptopPriceByModel() throws Exception {
        String resource = "SqlMapConfig.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);

        SqlSession session = factory.openSession();


        Laptop laptop = new Laptop();
        laptop.setHd(3.0);
        laptop.setModel("3001");
        laptop.setPrice(3599);
        laptop.setRam(23);
        laptop.setScreen(15.6);
        laptop.setSpeed(2000.0);

        session.update("laptop.updateLaptopPriceByModel", laptop);
        session.commit();
    }
}

```

