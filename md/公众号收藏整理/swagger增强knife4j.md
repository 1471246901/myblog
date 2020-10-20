# knife4j

----

一、介绍

knife4j是为Java MVC框架集成Swagger生成Api文档的增强解决方案（在非Java项目中也提供了前端UI的增强解决方案），前身是swagger-bootstrap-ui,取名knife4j是希望她能像一把匕首一样小巧,轻量,并且功能强悍!

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/640-1589689642818.webp)

二、开源仓库

-   Github

```
https://github.com/xiaoymin/swagger-bootstrap-ui
```

-   码云

```
https://gitee.com/xiaoym/knife4j
```

# 三、功能特性

-   **简洁**

基于左右菜单式的布局方式,是更符合国人的操作习惯吧.文档更清晰...

-   **个性化配置**

个性化配置项,支持接口地址、接口description属性、UI增强等个性化配置功能...

-   **增强**

接口排序、Swagger资源保护、导出Markdown、参数缓存众多强大功能...

# 四、功能预览

-   在线预览

```
http://knife4j.xiaominfo.com/doc.html
```

-   选择不同接口

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/640-1589689642824.webp)

-   Authorize

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/640-1589689642833.webp)

-   swagger实体

包含了swagger实体的相关信息

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/640-1589689642834.webp)

-   swagger全局设置

全局参数设置

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/640-1589689642835.webp)

-   离线文档导出

Knife4j提供导出4种格式的离线文档(Html\Markdown\Word\Pdf)

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/640-1589689642839.webp)

-   个性化设置

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/640-1589689642845.webp)

-   api文档

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/640-1589689642846.webp)

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/640-1589689642848.webp)

-   搜索功能

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/640-1589689642849.webp)

# 五、使用简介

-   项目结构



![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/640-1589689642859.webp)

-   目前主要的模块

```
目前主要的模块包括：模块名称说明：knife4j为Java MVC框架集成Swagger的增强解决方案knife4j-admin云端Swagger接口文档注册管理中心,集成gateway网关对任意微服务文档进行组合集成knife4j-extensionchrome浏览器的增强swagger接口文档ui,快速渲染swagger资源knife4j-service为swagger服务的一系列接口服务程序knife4j-frontknife4j-spring-ui的纯前端静态版本,用于集成非Java语言使用swagger-bootstrap-uiknife4j的前身,最后发布版本是1.9.6
```

-   单纯皮肤增强

不使用增强功能,纯粹换一个swagger的前端皮肤，这种情况是最简单的,你项目结构下无需变更

可以直接引用swagger-bootstrap-ui的最后一个版本1.9.6或者使用knife4j-spring-ui

老版本引用

```
<dependency>    
  <groupId>com.github.xiaoymin</groupId>    
  <artifactId>swagger-bootstrap-ui</artifactId>    
  <version>1.9.6</version>
</dependency>
```

新版本引用

```
<dependency>    
  <groupId>com.github.xiaoymin</groupId>    
  <artifactId>knife4j-spring-ui</artifactId>    
  <version>${lastVersion}</version>
</dependency>
```



-   Spring Boot项目单体架构使用增强功能

在Spring Boot单体架构下,knife4j提供了starter供开发者快速使用

```
<dependency>    
  <groupId>com.github.xiaoymin</groupId>    
  <artifactId>knife4j-spring-boot-starter</artifactId>    
  <version>${knife4j.version}</version>
</dependency>
```

该包会引用所有的knife4j提供的资源，包括前端Ui的jar包

-   Spring Cloud微服务架构

在Spring Cloud的微服务架构下,每个微服务其实并不需要引入前端的Ui资源,因此在每个微服务的Spring Boot项目下,引入knife4j提供的微服务starter

```
<dependency>    
  <groupId>com.github.xiaoymin</groupId>    
  <artifactId>knife4j-micro-spring-boot-starter</artifactId>    
  <version>${knife4j.version}</version>
</dependency>
```

在网关聚合文档服务下,可以再把前端的ui资源引入

```
<dependency>    
   <groupId>com.github.xiaoymin</groupId>    
   <artifactId>knife4j-spring-boot-starter</artifactId>    
   <version>${knife4j.version}</version>
</dependency>
```





>   作者：最美分享Coder
>
>   来源：http://39sd.cn/9D85F