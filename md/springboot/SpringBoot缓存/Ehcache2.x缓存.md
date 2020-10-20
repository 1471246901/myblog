# Ehcache 2.x 缓存

Ehcache 只需一个配置文件即可集成到springboot 中

## 配置

### maven 依赖

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <dependency>
            <groupId>net.sf.ehcache</groupId>
            <artifactId>ehcache</artifactId>
        </dependency>
```

### 添加配置文件

需要在class path路径下创建 `ehcache.xml` 配置文件

```java
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
         updateCheck="false">
    <!--
       diskStore：为缓存路径，ehcache分为内存和磁盘两级，此属性定义磁盘的缓存位置。参数解释如下：
       user.home – 用户主目录
       user.dir  – 用户当前工作目录
       java.io.tmpdir – 默认临时文件路径
     -->
    <diskStore path="java.io.tmpdir/Tmp_EhCache"/>
    <!--
       defaultCache：默认缓存策略，当ehcache找不到定义的缓存时，则使用这个缓存策略。只能定义一个。
     -->
    <!--
      name:缓存名称。
      maxElementsInMemory:缓存最大数目
      maxElementsOnDisk：硬盘最大缓存个数。
      eternal:对象是否永久有效，一但设置了，timeout将不起作用。
      overflowToDisk:是否保存到磁盘，当系统当机时
      timeToIdleSeconds:设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
      timeToLiveSeconds:设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
      diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.
      diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
      diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
      memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
      clearOnFlush：内存数量最大时是否清除。
      memoryStoreEvictionPolicy:可选策略有：LRU（最近最少使用，默认策略）、FIFO（先进先出）、LFU（最少访问次数）。
      FIFO，first in first out，这个是大家最熟的，先进先出。
      LFU， Less Frequently Used，就是上面例子中使用的策略，直白一点就是讲一直以来最少被使用的。如上面所讲，缓存的元素有一个hit属性，hit值最小的将会被清出缓存。
      LRU，Least Recently Used，最近最少使用的，缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存。
   -->
    <defaultCache
            eternal="false"
            maxElementsInMemory="10000"
            overflowToDisk="false"
            diskPersistent="false"
            timeToIdleSeconds="1800"
            timeToLiveSeconds="259200"
            memoryStoreEvictionPolicy="LRU"/>

    <cache
            name="cloud_user"
            eternal="false"
            maxElementsInMemory="5000"
            overflowToDisk="false"
            diskPersistent="false"
            timeToIdleSeconds="1800"
            timeToLiveSeconds="1800"
            memoryStoreEvictionPolicy="LRU"/>

</ehcache>
```

### 指定配置文件位置

application.yml   默认位置 `类路径下`

```yml
spring:
  cache:
    ehcache:
      config: classpath:config/ehcache.xml
```

### 开启缓存

启动类上标注 `@EnableCaching`

```
@SpringBootApplication
@EnableCaching
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

## 使用

创建实体类和BookService  `缓存是以Repository为单位的`

```java
public class Book implements Serializable {   //要实现序列化接口
    private Integer id;
    private String name;
    private String author;

```

```java
//service
public Book getBook(Integer id){
        return bookDao.getAll(id);
    }
```

创建dao

```java
@Repository
@CacheConfig(cacheNames = "cloud_user")  //配置文件定义的规则的名称
public class BookDao {


    @Cacheable
    public Book getAll(Integer id){
        Book book = new Book();
        book.setId(id);
        book.setName("aaa");
        book.setAuthor("bbb");
        return book;
    }


}
```

**解释**

**`@CacheConfig`** 在类上指明要使用规则的名字,这个配置可选,也可以在方法上通过 `@Cacheable` 指明缓存名称 ,不注明使用默认缓存

**`@Cacheable`** 指明对此方法加入缓存,默认情况下缓存的名称是 [类名方法名参数] 缓存的value 是方法的返回值 ,它还可以控制缓存的执行时机 例如 `@Cacheable(condition="#id%2==0")` 表示当id为2的时候才进行缓存

<span style="color:red">注意</span> 在请求过程中如果方法的参数信息匹配上则直接返回,不匹配才会运行,但是在本类中调用这个方法不会触发缓存 `缓存不会生效`

**`@CachePut`**  具有和`@Cacheable` 相同的属性 ,不同的是在每次执行该方法时,都会去执行方法,并将方法执行结果缓存起来,这样可以避免查询到脏数据

**`@CacheEvict`**  和 `@Cacheable` 一样 ,不同的是 一般用于删除方法上表示移除一个key 所对应的缓存 其有两个特殊的属性  `allEntries` 表示将所有缓存数据都移除 默认为 false , `beforeInvocation` 表示在方法执行前将缓存移除 默认为false

### 自定义缓存key

如果想对key进行控制 ,如

`@Cacheable(key= "#book.id")`  表示缓存的key 为这个方法参数对象的id值 例如 方法为 `public int save(Book book)`

`@Cacheable(key = "#id")`  表示缓存的key为这个方法参数的id 值  例如方法为  `public Book get(int id)`

除了上述方法生成key springboot 还提供了 `root`对象生成key

| 属性名称    | 属性描述                | 用法示例             |
| ----------- | ----------------------- | -------------------- |
| methodName  | 当前方法名              | #root.methodName     |
| method      | 当前方法对象            | #root.method.name    |
| caches      | 当前方法使用的缓存      | #root.caches[0].name |
| target      | 当前被调用的对象        | #root.target         |
| targetClass | 当前被调用的对象的class | #root.targetClass    |
| args        | 当前方法参数数组        | #root.args[0] ..     |



## 测试

创建测试用例

```java
@Autowired
    BookDao bookDao;

    @Test
    public void testCache(){
        System.out.println("执行第一次");
        bookDao.getAll(1);

        System.out.println("执行第二次");
        bookDao.getAll(1);

        System.out.println("另一个id执行第一次");
        bookDao.getAll(2);


    }
```



结果

```java
执行第一次
book Dao 被执行 id:1
执行第二次
另一个id执行第一次
book Dao 被执行 id:2
//cache生效
```

