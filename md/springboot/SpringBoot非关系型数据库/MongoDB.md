# MongoDB

## 安装MongoDB

ubuntu 安装

### 下载mongodb

```
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.0.tgz
```

### 解压

```
mkdir mongodb
cd mongodb
tar -zxvf mongodb-linux-x86_64-4.0.0.tgz
```

### 创建db\logs 文件夹

```
mkdir db
mkdir logs
```

### 创建配置文件 `mongo.conf`   

在 `bin` 文件夹下创建mongo.conf 

```properties
# 数据库存储目录
dbpath=/home/gushiyu/mongodb/db
# 日志文件目录
logpath=/home/gushiyu/mongodb/logs
# 端口
port=27017
# 守护进程方式启动mongodb
fork=true
```

### 启动mongodb

```shell
./mongod -f mongo.conf --bind_ip_all
```

在`bin` 目录下执行 mongo 命令  查看版本

```
./mongo
db.version()
```

```shell
gushiyu@gushiyumongodb:~/mongodb/mongodb-linux-x86_64-4.0.0/bin$ ./mongo
MongoDB shell version v4.0.0
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 4.0.0
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user
Server has startup warnings: 
2020-09-06T01:19:44.944+0000 I STORAGE  [initandlisten] 
2020-09-06T01:19:44.944+0000 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine
2020-09-06T01:19:44.944+0000 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
2020-09-06T01:19:46.483+0000 I CONTROL  [initandlisten] 
2020-09-06T01:19:46.483+0000 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2020-09-06T01:19:46.483+0000 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2020-09-06T01:19:46.483+0000 I CONTROL  [initandlisten] 
> db.version()
4.0.0

```

## MongoDB 安全管理

启动mongodb后 由于mongodb默认没有密码,所以我们要创建密码,mongodb 每个库都有自己的密码,用户使用哪个库就需要哪一个库中的密码

### 创建密码

```
use admin
db.createUser({user:"gushiyu",pwd:"root",roles:[{role:"readWrite",db:"test"}]})
```

```
> use admin
switched to db admin
> db.createUser({user:"gushiyu",pwd:"root",roles:[{role:"readWrite",db:"test"}]})
Successfully added user: {
	"user" : "gushiyu",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "test"
		}
	]
}

```

### 关闭实例

ps aux|grep mongo

grep --color=auto mongo

```shell
gushiyu@gushiyumongodb:~/mongodb/mongodb-linux-x86_64-4.0.0/bin$ ps aux|grep mongo
gushiyu     3015  0.6  6.0 973412 61200 ?        SLl  01:19   0:03 ./mongod -f mongo.conf --bind_ip_all
gushiyu     3060  0.0  0.0   6432   672 pts/0    S+   01:29   0:00 grep --color=auto mongo
gushiyu@gushiyumongodb:~/mongodb/mongodb-linux-x86_64-4.0.0/bin$ kill -9 3015

```

### 启动并认证

```shell
./mongod -f mongo.conf --auth --bind_ip_all
db.auth("gushiyu","root")
# 若返回1 则代表成功

```

## Spring Boot 整合MongoDB

### 配置

#### 依赖

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

#### application.properties

```properties
# 验证登录信息的库
spring.data.mongodb.authentication-database=admin
# 要连接的库,认证信息并不一定在这个库
spring.data.mongodb.database=test
spring.data.mongodb.host=192.168.31.184
spring.data.mongodb.port=27017
spring.data.mongodb.username=gushiyu
spring.data.mongodb.password=root
```

#### 自动配置

mongodb 的配置信息来源于 `org.springframework.boot.autoconfigure.mongo.MongoProperties ` 前缀 `spring.data.mongodb`

mongodb 的自动配置来源于 `org.springframework.boot.autoconfigure.mongo` 和 `org.springframework.boot.autoconfigure.data.mongo` 

其中 `org.springframework.boot.autoconfigure.mongo` 主要配置mongodb连接 注入了 `MongoClient` 如果不想使用 spring-data-mongodb 配置到这一步即可

`org.springframework.boot.autoconfigure.data.mongo`  主要配置了 spring-data-mongodb 

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ MongoClient.class, MongoTemplate.class })
@EnableConfigurationProperties(MongoProperties.class)
@Import({ MongoDataConfiguration.class, MongoDatabaseFactoryConfiguration.class,
		MongoDatabaseFactoryDependentConfiguration.class })
@AutoConfigureAfter(MongoAutoConfiguration.class)
public class MongoDataAutoConfiguration {

}
```

 主要注入了 `MongoDatabaseFactory` `mongoTemplate` 

### 使用

#### 创建实体类

```java
public class Book {

    private Integer id;
    private String name;
    private String author;
```

#### 创建dao

```java
public interface BookDao extends MongoRepository<Book,Integer> {
    List<Book> findByAuthorContains(String author);
    Book findByNameEquals(String name);
}
```

#### 创建controller

```java
@RestController
public class TestController {


    @Autowired
    BookDao bookDao;
    
    @Autowired
    MongoTemplate mongoTemplate;
    //可以使用 mongoTemplate 操作mogodb

    @RequestMapping("/test")
    public Object test(){

        Book book = new Book();
        book.setId(1);
        book.setName("asdasd");
        book.setAuthor("鲁迅");
        bookDao.save(book);
        List<Book> b = bookDao.findByAuthorContains("鲁迅");
        return b;
    }
}
```

测试

![image-20200906101419047](Untitled.assets/image-20200906101419047.png)