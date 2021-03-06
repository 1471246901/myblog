# MybatisCodeHelperPro

插件要钱,仅需 29 元 1 年，用过的都说好用

购买地址:http://brucege.com/

用邮箱可以7天试用

## 使用配置

### 配置数据库

使用IDEA连接数据库

<img src="MybatisCodeHelperPro%E6%8F%92%E4%BB%B6.assets/image-20200819103629604.png" alt="image-20200819103629604" style="zoom:50%;" />

填写数据库位置,用户,密码, 数据库名称,serverTimezone=UTC

## 功能

### 通过javaBean生成建表语句

创建javabean

```java
@Data
public class TestBean {
    File icon;
    String name;
    BigDecimal money;
    Double height;
    Integer id;
}
```

选择 `alt`+ `insert` 或者右键 Generate 就会见到 `generate mybatis file` 选项

<img src="MybatisCodeHelperPro%E6%8F%92%E4%BB%B6.assets/image-20200819104831867.png" alt="image-20200819104831867" style="zoom:50%;" />

点击OK 得到建表SQL语句

```sql
-- auto Generated on 2020-08-19
-- DROP TABLE IF EXISTS test_bean;
CREATE TABLE test_bean(
	`name` VARCHAR (50) NOT NULL DEFAULT '' COMMENT 'name',
	money DECIMAL (13,4) NOT NULL DEFAULT -1 COMMENT 'money',
	height DOUBLE (16,4) NOT NULL DEFAULT -1.0 COMMENT 'height',
	id INT (11) NOT NULL AUTO_INCREMENT COMMENT 'id',
INDEX `ix_id_height_money`(id,height,money),
INDEX `ix_id_height_money`(id,height,money),
	PRIMARY KEY (id)
)ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT 'test_bean';
```

运行即可得到数据库表

![image-20200819105330293](MybatisCodeHelperPro%E6%8F%92%E4%BB%B6.assets/image-20200819105330293.png)

### 通过表生成javabean

连接好数据库后,在表上右击 会出现 ![image-20200819105612245](MybatisCodeHelperPro%E6%8F%92%E4%BB%B6.assets/image-20200819105612245.png)

generate java class 作用是生成javabean ,点击它 选择选项及位置生成

<img src="MybatisCodeHelperPro%E6%8F%92%E4%BB%B6.assets/image-20200819105729823.png" alt="image-20200819105729823" style="zoom: 80%;" />

### 快速获取列名

连接好数据库后,在表上右击,并点击`generate all column sql` 选择对应sql复制

### Mybatis Generator 生成单表的 xml mapper bean

连接好数据库后,在相应表上右击,并点击`Mybatis Generator`

![image-20200819111032367](MybatisCodeHelperPro%E6%8F%92%E4%BB%B6.assets/image-20200819111032367.png)

选择对应的选项 确定,然后生成相应的文件

### Mybatis Generator 生成多表的 xml mapper bean

连接好数据库后,在表上右击,并点击`Mybatis multiple table Generate`  像单表操作即可

### 特定方法名自动生成SQL

在mapper 书写符合规范(JPA)的方法名,会自动生成想用的xml ,选项中 `Generate mybatis sql with if test` 用于测试传入的数据是否有效

![image-20200819112342120](MybatisCodeHelperPro%E6%8F%92%E4%BB%B6.assets/image-20200819112342120.png)

![image-20200819112541244](MybatisCodeHelperPro%E6%8F%92%E4%BB%B6.assets/image-20200819112541244.png)

生成的接口

```java
List<TestBean> findByHeightGreaterThan(@Param("minHeight")Double minHeight);
```

生成的 xml 

```xml
<!--auto generated by MybatisCodeHelper on 2020-08-19-->
  <select id="findByHeightGreaterThan" resultMap="BaseResultMap">
    select
    <include refid="Base_Column_List"/>
    from test_bean
    where height <![CDATA[>]]> #{minHeight,jdbcType=DOUBLE}
  </select>
```

with if test 选项生成的xml 中if 判断了height 字段

```xml
<!--auto generated by MybatisCodeHelper on 2020-08-19-->
  <select id="findByHeight" resultMap="BaseResultMap">
    select
    <include refid="Base_Column_List"/>
    from test_bean
    <where>
      <if test="height != null">
        and height=#{height,jdbcType=DOUBLE}
      </if>
    </where>
  </select>
```

### 数据库变动更新代码

数据库变动后使用默认方法生成xml 及java代码 ,自己添加的代码不会被改动,自动生成的代码已改动完成

### xml 提示功能

编写xml 的时候回有相应的字段提示和java变量提示

### @mapperscan 注解识别 ,mapper和xml 对应

识别@mapperscan注解指明的地方为mapper类 并和xml 对应,否则报错

### 一键生成测试样例

在mapper 类下 提示下选择`Generate mybatis testcase` 

![image-20200819115338799](MybatisCodeHelperPro%E6%8F%92%E4%BB%B6.assets/image-20200819115338799.png)

会在test目录生成 `测试类`文件 和 `TestConfiguration.xml` 配置文件

**另外三个选项是将注解匹配到参数和对应service类生成相应的service 方法以及分页查询**



### 可视化编辑mybatisXML文件

点击xml文件左侧小三角编辑

![image-20200819120138777](MybatisCodeHelperPro%E6%8F%92%E4%BB%B6.assets/image-20200819120138777.png)

