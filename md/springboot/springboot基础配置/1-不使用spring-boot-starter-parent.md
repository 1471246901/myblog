# 不使用spring-boot-stater-parent

创建spring-boot项目在添加依赖之前需要先添加spring-boot-starter-parent,spring-boot-starter-parent主要提供一下配置

-   java版本默认使用1.8
-   编码格式默认使用utf-8
-   提供<a href="../dependencyManagement使用简介.md"> Dependency Management </a>进行项目依赖的版本管理
-   默认的资源过滤与插件配置

默认创建springboot项目会带有

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
    <!-- 注意这里spring-boot-starter-parent-->
    <!--                                  -->
    <parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.3.1.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
    <!--                                  -->
    <!-- 注意这里spring-boot-starter-parent-->
	<groupId>org.gushiyu</groupId>
	<artifactId>springdemo1</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>springdemo1</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>11</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```



>   Dependency Management
>
>   Maven中的dependencyManagement元素提供了一种管理依赖版本号的方式。在dependencyManagement元素中声明所依赖的jar包的版本号等信息，那么所有子项目再次引入此依赖jar包时则无需显式的列出版本号。Maven会沿着父子层级向上寻找拥有dependencyManagement 元素的项目，然后使用它指定的版本号。
>
>   <span style="color:red;">注意</span> 虽然声明了版本号, 但是不是引入,

spring-boot-stater-parent 虽然方便,但是很多公司在开发微服务项目或者多模块项目时一般需要使用公司自己的parent,这个时候如果还想进行项目依赖版本的统一管理,就需要使用 Dependency Management 流程如下.

先删除`parent  `再添加如下代码到pom.xml中

```xml
<!-- 添加到 dependencies 标签之前,根目录中  -->
<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.1.RELEASE</version>  <!-- 这里填写版本号 -->
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
</dependencyManagement>
```

然后配置java 版本和编码

```xml
<!-- 添加到根目录中 -->
<properties>
        <java.version>11</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
</properties>
    <!-- 添加到plugins之中 -->
<plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source >${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
</plugin>
```





最后完整的pom.xml文件

```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.gushiyu</groupId>
    <artifactId>springdemo1</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springdemo1</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>11</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.1.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
    
    <!-- 这里添加springboot 的stater -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source >${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>

        </plugins>
    </build>

</project>

```

