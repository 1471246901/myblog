# Mybatis 动态SQL

---



MyBatis通过 OGNL 来进行动态 SQL 的使用的。目前， 动态 SQL 支持以下几种标签：

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/640.png)

## 数据准备

为了后面的演示， 创建了一个 Maven 项目 mybatis-dynamic, 创建了对应的数据库和表

```sql
DROP TABLE IF EXISTS `student`;

CREATE TABLE `student` (
  `student_id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '编号',
  `name` varchar(20) DEFAULT NULL COMMENT '姓名',
  `phone` varchar(20) DEFAULT NULL COMMENT '电话',
  `email` varchar(50) DEFAULT NULL COMMENT '邮箱',
  `sex` tinyint(4) DEFAULT NULL COMMENT '性别',
  `locked` tinyint(4) DEFAULT NULL COMMENT '状态(0:正常,1:锁定)',
  `gmt_created` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '存入数据库的时间',
  `gmt_modified` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改的时间',
  `delete` int(11) DEFAULT NULL,
  PRIMARY KEY (`student_id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='学生表';
```

对应的项目结构

![](https://raw.githubusercontent.com/1471246901/myblog/master/img/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20200724095820.png)

## if标签

### 在 WHERE 条件中使用 if 标签

这是常见的一种现象， 我们在进行按条件查询的时候， 可能会有多种情况。

#### 查询条件

根据输入的学生信息进行条件检索

-   当只输入用户名时， 使用用户名进行模糊检索；
-   当只输入性别时， 使用性别进行完全匹配
-   当用户名和性别都存在时， 用这两个条件进行查询匹配查询

####  动态 SQL

接口函数

```java
    /**
     * 根据输入的学生信息进行条件检索
     * 1. 当只输入用户名时， 使用用户名进行模糊检索；
     * 2. 当只输入邮箱时， 使用性别进行完全匹配
     * 3. 当用户名和性别都存在时， 用这两个条件进行查询匹配的用
     * @param student
     * @return
     */
        List<Student> selectByStudentSelective(Student student);
```

对应的动态 SQL

```xml
  <select id="selectByStudentSelective" resultMap="BaseResultMap" parameterType="com.homejim.mybatis.entity.Student">
    select
    <include refid="Base_Column_List" />
    from student
    where 1=1
    <if test="name != null and name !=''">
      and name like concat('%', #{name}, '%')
    </if>
    <if test="sex != null">
      and sex=#{sex}
    </if>
  </select>
```

在此 SQL 语句中， where 1=1 是多条件拼接时的小技巧， 后面的条件查询就可以都用 and 了。

同时， 我们添加了 if 标签来处理动态 SQL

```xml
    <if test="name != null and name !=''">
      and name like concat('%', #{name}, '%')
    </if>
    <if test="sex != null">
      and sex=#{sex}
    </if>
```

此 if 标签的 test 属性值是一个符合 OGNL 的表达式， 表达式可以是 true 或 false。如果表达式返回的是数值， 则0为 false, 非 0 为 true;

### 在 UPDATE 更新列中使用 if 标签

在一个实体(一条记录)中,有时候我们不希望更新所有的字段， 只更新有变化的字段。

#### 查询条件

更新的时候只更新有变化的字段， 空值不更新

#### 动态 SQL

接口方法

```
    /**
     * 更新非空属性
     */
    int updateByPrimaryKeySelective(Student record);
```

对应的 SQL

```xml
  <update id="updateByPrimaryKeySelective" parameterType="com.homejim.mybatis.entity.Student">
    update student
    <set>
      <if test="name != null">
        `name` = #{name,jdbcType=VARCHAR},
      </if>
      <if test="phone != null">
        phone = #{phone,jdbcType=VARCHAR},
      </if>
      <if test="email != null">
        email = #{email,jdbcType=VARCHAR},
      </if>
      <if test="sex != null">
        sex = #{sex,jdbcType=TINYINT},
      </if>
      <if test="locked != null">
        locked = #{locked,jdbcType=TINYINT},
      </if>
      <if test="gmtCreated != null">
        gmt_created = #{gmtCreated,jdbcType=TIMESTAMP},
      </if>
      <if test="gmtModified != null">
        gmt_modified = #{gmtModified,jdbcType=TIMESTAMP},
      </if>
    </set>
    where student_id = #{studentId,jdbcType=INTEGER}
```

### 在 INSERT 动态插入中使用 if 标签

我们插入数据库中的一条记录， 不是每一个字段都有值的， 而是动态变化的。在这时候使用 if 标签， 可帮我们解决这个问题。

#### 插入条件

只有非空属性才插入。

### 2.3.2 动态SQL

接口方法

```java
    /**
     * 非空字段才进行插入
     */
    int insertSelective(Student record);
```

对应的SQL

```xml
<insert id="insertSelective" parameterType="com.homejim.mybatis.entity.Student">
    insert into student
    <trim prefix="(" suffix=")" suffixOverrides=",">
      <if test="studentId != null">
        student_id,
      </if>
      <if test="name != null">
        `name`,
      </if>
      <if test="phone != null">
        phone,
      </if>
      <if test="email != null">
        email,
      </if>
      <if test="sex != null">
        sex,
      </if>
      <if test="locked != null">
        locked,
      </if>
      <if test="gmtCreated != null">
        gmt_created,
      </if>
      <if test="gmtModified != null">
        gmt_modified,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides=",">
      <if test="studentId != null">
        #{studentId,jdbcType=INTEGER},
      </if>
      <if test="name != null">
        #{name,jdbcType=VARCHAR},
      </if>
      <if test="phone != null">
        #{phone,jdbcType=VARCHAR},
      </if>
      <if test="email != null">
        #{email,jdbcType=VARCHAR},
      </if>
      <if test="sex != null">
        #{sex,jdbcType=TINYINT},
      </if>
      <if test="locked != null">
        #{locked,jdbcType=TINYINT},
      </if>
      <if test="gmtCreated != null">
        #{gmtCreated,jdbcType=TIMESTAMP},
      </if>
      <if test="gmtModified != null">
        #{gmtModified,jdbcType=TIMESTAMP},
      </if>
    </trim>
  </insert>
```

## choose 标签

choose when otherwise 标签可以帮我们实现 if else 的逻辑。一个 choose 标签至少有一个 when, 最多一个otherwise。

### 查询条件

假设 name 具有唯一性， 查询一个学生

-   当 studen_id 有值时， 使用 studen_id 进行查询；
-   当 studen_id 没有值时， 使用 name 进行查询；
-   否则返回空

### 动态SQL

接口方法

```java
    /**
     * - 当 studen_id 有值时， 使用 studen_id 进行查询；
     * - 当 studen_id 没有值时， 使用 name 进行查询；
     * - 否则返回空
     */
    Student selectByIdOrName(Student record);
```

对应的SQL

```xml
  <select id="selectByIdOrName" resultMap="BaseResultMap" parameterType="com.homejim.mybatis.entity.Student">
    select
    <include refid="Base_Column_List" />
    from student
    where 1=1
    <choose>
      <when test="studentId != null">
        and student_id=#{studentId}
      </when>
      <when test="name != null and name != ''">
        and name=#{name}
      </when>
      <otherwise>
        and 1=2
      </otherwise>
    </choose>
  </select>
```

## trim(set、where)标签

这三个其实解决的是类似的问题。如我们在写前面的[在 WHERE 条件中使用 if 标签] SQL 的时候， where 1=1 这个条件我们是不希望存在的。

### 不使用where 1=1

#### 查询条件

根据输入的学生信息进行条件检索。

-   当只输入用户名时， 使用用户名进行模糊检索；
-   当只输入性别时， 使用性别进行完全匹配
-   当用户名和性别都存在时， 用这两个条件进行查询匹配查询

不使用 where 1=1。

#### 动态 SQL

很显然， 我们要解决这几个问题

当条件都不满足时：此时 SQL 中应该要不能有 where ， 否则导致出错 当 if 有条件满足时：SQL 中需要有 where， 且第一个成立的 if 标签下的 and | or 等要去掉 这时候， 我们可以使用 `where` 标签。

接口方法

```java
    /**
     * 根据输入的学生信息进行条件检索
     * 1. 当只输入用户名时， 使用用户名进行模糊检索；
     * 2. 当只输入邮箱时， 使用性别进行完全匹配
     * 3. 当用户名和性别都存在时， 用这两个条件进行查询匹配的用
     */
    List<Student> selectByStudentSelectiveWhereTag(Student student);
```

对应的 SQL

```xml
  <select id="selectByStudentSelectiveWhereTag" resultMap="BaseResultMap" parameterType="com.homejim.mybatis.entity.Student">
    select
    <include refid="Base_Column_List" />
    from student
   <where>
    <if test="name != null and name !=''">
      and name like concat('%', #{name}, '%')
    </if>
    <if test="sex != null">
      and sex=#{sex}
    </if>
   </where>
  </select>
```

### 去除set

set 标签也类似 ,直接使用   <set></set> 就可以

### trim 标签

set 和 where 其实都是 trim 标签的一种类型， 该两种功能都可以使用 trim 标签进行实现。

#### 用trim来表示 where 

如以上的 where 标签， 我们也可以写成

```xml
<trim prefix="where" prefixOverrides="AND |OR">
</trim>
```

表示当 trim 中含有内容时， 添加 where('prefix' 属性)， 且第一个为 and 或 or 时(`prefixOverrides`属性)， 会将其去掉。而如果没有内容， 则不添加 where。

相同可以用trim表示set标签

#### trim标签的属性

-   prefix: 当 trim 元素包含有内容时， 增加 prefix 所指定的前缀
-   prefixOverrides: 当 trim 元素包含有内容时， 去除 prefixOverrides 指定的 前缀
-   suffix: 当 trim 元素包含有内容时， 增加 suffix 所指定的后缀
-   suffixOverrides：当 trim 元素包含有内容时， 去除 suffixOverrides 指定的后缀



## foreach标签

foreach 标签可以对数组， Map 或实现 Iterable 接口。

foreach 中有以下几个属性：

-   collection: 必填， 集合/数组/Map的名称.
-   item: 变量名。即从迭代的对象中取出的每一个值
-   index: 索引的属性名。当迭代的对象为 Map 时， 该值为 Map 中的 Key.
-   open: 循环开头的字符串
-   close: 循环结束的字符串
-   separator: 每次循环的分隔符

其他的比较好理解， collection 中的值应该怎么设定呢？

跟接口方法中的参数相关。

1. **只有一个数组参数或集合参数**

默认情况：集合collection=list， 数组是collection=array

推荐：使用 @Param 来指定参数的名称， 如我们在参数前@Param("ids")， 则就填写 collection=ids

2. **多参数**

多参数请使用 @Param 来指定， 否则SQL中会很不方便

3. **参数是Map**

指定为 Map 中的对应的 Key 即可。其实上面的 @Param 最后也是转化为 Map 的。

4. **参数是对象**

使用属性.属性即可。

### where中使用foreach

在 where条件中使用， 如按id集合查询， 按id集合删除等。

#### 查询条件

我们希望查询用户 id 集合中的所有用户信息。

#### 动态 SQL

函数接口

```java
    /**
     * 获取 id 集合中的用户信息
     * @param ids
     * @return
     */
    List<Student> selectByStudentIdList(List<Integer> ids);
```

对应 SQL

```xml
  <select id="selectByStudentIdList" resultMap="BaseResultMap">
    select
    <include refid="Base_Column_List" />
    from student
    where student_id in
    <foreach collection="list" item="id" open="(" close=")" separator="," index="i">
      #{id}
    </foreach>
  </select>
```

### foreach 实现批量插入

#### 动态SQL

接口方法

```java
    /**
     * 批量插入学生
     */
    int insertList(List<Student> students);
```

对应的SQL

```xml
  <insert id="insertList">
    insert into student(name, phone, email, sex, locked)
    values
    <foreach collection="list" item="student" separator=",">
      (
      #{student.name}, #{student.phone},#{student.email},
      #{student.sex},#{student.locked}
      )
    </foreach>
  </insert>
```



## bind 标签

bind 标签是通过 OGNL 表达式去定义一个上下文的变量， 这样方便我们使用。

如在 selectByStudentSelective 方法中， 有如下

```xml
<if test="name != null and name !=''">
      and name like concat('%', #{name}, '%')
    </if>
```

在 MySQL 中， 该函数支持多参数， 但在 Oracle 中只支持两个参数。那么我们可以使用 bind 来让该 SQL 达到支持两个数据库的作用

```xml
<if test="name != null and name !=''">
     <bind name="nameLike" value="'%'+name+'%'"/>
     and name like #{nameLike}
</if>
```

