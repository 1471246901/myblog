# 创建spring_boot项目

#### 1 下载IDEA开发工具

访问JetBrains[官网](https://www.jetbrains.com/) 下载IEDA,安装即可

#### 2 IDEA激活

1.  如果有教育邮箱可以申请教育许可证`或者使用学生证,教师证申请`
2.  参考网上破解教程

#### 3 创建spring boot 项目 

1.  创建spring boot 分为3步 ,但是通过spring boot 提供的工具可以直接完成这三部

    -   创建MAVEN项目
    -   引入starters依赖
    -   创建主程序

2.  IDEA中点击`File` -`new`-`project` 

    <img src="https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200412142122262.png" alt="image-20200412142122262" style="zoom: 80%;" />

3.  选择 `spring initializr`项目<img src="https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200412142248227.png" alt="image-20200412142248227" style="zoom:67%;" />

4.  图中   `initializr service URL` 是构建项目服务器地址 默认就可以了 

5.  填写好 `groupid`组织名 , `artifact`项目名,`java version` java版本 点击下一步<img src="https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200412143850633.png" alt="image-20200412143850633" style="zoom:80%;" />

6.  接下来选择需要的场景启动器(starters) 并点击下一步

7.  选择项目存放路径即可

8.  创建成功后maven会在后台下载所需的包和依赖,项目目录如下<img src="https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200412144817014.png" alt="image-20200412144817014" style="zoom: 80%;" />

9.  主程序就在类路径根目录下   `ScaffoldingApplication`  所构建的程序就是一个标准的java程序 可以直接java -jar运行

#### 4 输出helloworld

1.  在`com.gushiyu.scaffolding` 下创建一个包 名为`controller`  ,`com.gushiyu.scaffolding.controller`

2.  在该包中新建一个测试类TestController    并标注注解 `@RestController`

3.  再类中实现一个方法 String Hello()  并添加注解 `@RequestMapping("/hello")`  ,"/hello" 为浏览器的url映射![image-20200412150146737](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200412150146737.png)

4.  启动项目,启动 `ScaffoldingApplication`  类下的main方法即可![image-20200412150340499](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200412150340499.png)

5.  浏览器中输入`127.0.0.1:8080/hello ` 查看返回的结果![image-20200412150516136](D:\Typora\data\image\image-20200412150516136.png)

    #### 自此springboot项目创建完成

    