# Spring Boot打包插件MavenPlugin

---

Spring Boot 对 Maven 一直支持很友好，栈长也一直在用 Maven 进行依赖和项目管理，那么今天就讲一下这个插件的作用，非常有用！

有了 `Spring Boot Maven Plugin` 这个插件，我们可以将项目打成可执行的 jar 包（*.jar）以及 war 包（*.war），可以帮助我们很方便的运行 Spring Boot 应用。

官方地址：

>   https://docs.spring.io/spring-boot/docs/current/maven-plugin/index.html

#### 主要包括以下几个目标（goals）

-   spring-boot:run

可以不用打包，直接运行 Spring Boot 应用。

-   spring-boot:repackage

可以打成可执行的运行包（*.jar/*.war）

-   spring-boot:start/ spring-boot:stop

用于管理 Spring 应用程序的生命周期(例如：用于集成测试)。

-   spring-boot:build-info

用于生成构建信息，用于 Spring Boot Actuator。

#### 如何使用

下面主要讲一下前面两种 `goal` 的使用，后面两种用的比较少。

集成 Spring Boot Maven Plugin 插件：

```xml
<build>
  ...
  <plugins>
    ...
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <version>2.2.6.RELEASE</version>
    </plugin>
    ...
  </plugins>
  ...
</build>
```

**1、直接运行 Spring Boot 应用**

在 Maven 命令行使用：

```
mvn spring-boot:run
```

如果在 IDE 开发工具中，可以省去 `mvn` 命令：

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/640-1589686847918.png)

当然，我们可以直接运行 main class，但使用 Maven 插件可以有更多的功能特性，比如：切换不同环境配置（Profile）, 资源替换 Maven Resource 插件的结合使用。

默认情况下，插件运行在一个新进程中，命令行设置的 JVM 参数是不生效的，需要单独指定：

```
-Dspring-boot.run.jvmArguments="-Dspring.profiles.active=dev"
```

你也还可以指定其他参数：

-   systemPropertyVariables：系统属性变量
-   environmentVariables：环境变量

除此之外，其他 JVM 参数，都可以在命令后面指定。

**2、打成可执行包**

来看一个完整示例：

```xml
<build>
  ...
  <plugins>
    ...
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <version>2.2.6.RELEASE</version>
      <configuration>
        <mainClass>${start-class}</mainClass>
        <layout>jar</layout>
      </configuration>
      <executions>
        <execution>
          <goals>
            <goal>repackage</goal>
          </goals>
          <configuration>
        　  <classifier>exec</classifier>
        　</configuration>
        </execution>
      </executions>
    </plugin>
    ...
  </plugins>
  ...
</build>
```

以下几个参数都是可选的。

**repackage：**

最主要的是要添加 `repackage` goal，用来重新打包。

**layout：**

layout 属性根据项目类型默认是：jar/war，具体可以设置以下几种：

-   **JAR**：可执行 jar 包；
-   **WAR**：可执行 war 包；
-   **ZIP**（别名：DIR）：和 jar 包相似，使用的是：PropertiesLauncher；
-   **NONE**：打包所有依赖项和项目资源，不绑定任何启动加载器；

**classifier：**

默认情况下只会打一个包，但是如果这个模块既是其他模板的依赖，自身又需要打成可执行的运行包，那就需要用这个标签另外指定一个别名包，如：

-   xxx.jar
-   xxx-exec-jar

具体参考:

>   https://docs.spring.io/spring-boot/docs/current/maven-plugin/examples/repackage-classifier.html

Spring Boot 打包这个插件经常会用到，大家还是要学会使用它，不然出去面试，面试官一问 Spring Boot 项目如何打包，你就一脸 MB 了。



 [spring boot 怎么打包为一个可执行的jar包](https://mp.weixin.qq.com/s/iViefaB5Q-RmD5SBRL4jeA)



>   公众号  java技术栈