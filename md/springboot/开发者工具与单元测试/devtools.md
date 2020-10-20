# devtools

spring boot 提供了一组开发工具 `spring-boot-devtools` 可以很方便的提高开发者的工作效率

## 配置

想要在项目中使用`devtools`模块 

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        <!-- optional 防止devtools传递到其他项目中 ,并在打包后devtools将自动禁止使用 -->
        </dependency>
```

## 使用

开发者使用了`devtools `之后,只要classpath 下的文件发生了改变,系统就会自动处理,如果开发者使用eclipse 项目会自动重启,如或是 IDEA 则需配置手动编译才会触发重启

禁用自动重启配置 `spring.devtools.restart.enable=false`

*手动编译: Build_Build Project* 

### 开启自动编译

`file` -  `setting  `  -  `build,execution,deployment`  `compiler`  下的 `build project automatically`

![image-20200911164848424](Untitled.assets/image-20200911164848424.png)

下一步

按 Ctrl + Shift + Alt + /  快捷键调处  `Maintenance` 页面  选择 Registry  选择 `compiler.automake.allow.when.app.running`

开发者再次在IDEA 中修改代码,项目会自动重启



## 基本原理

通过两个类加载器 `baseclassloader` 加载没有变化的类  还有用`restartclassloader` 加载变化的类  因为`baseclassloader`  已经存在并已经加载好

## 自定义监控资源

默认情况下 `/META-INF/maven` , `META-INF/resources` , `/resources`  , `/static` 

, `/public` ,  `templates` 位置下的资源变化不会触发重启 若要重新定义, 配置为 `spring.devtools.restart.exclude = 数组` 不会触发重启的文件夹 

相应的用户也可以配置要检测的文件夹 `spring.devtools.restart.additional-paths= [检测的文件夹] src/main/resources/static` 

用户也可以配置一个触发文件,当文件变化的时候会自动触发重启  `spring.devtools.restart.trigger-file=.triggerfile`  .triggerfile 文件在 `resources` 文件夹下创建

## 静态资源监控

devtools 默认嵌入了LiveReload 服务器可以做到静态文件的热部署 , LiveReload 在静态资源变化的时候自动触发更新 ,使用LiveReload 可以在浏览器应用市场下载

![image-20200912102310484](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200912102310484.png)

安装完成之后 点击 ![image-20200912102347883](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200912102347883.png) 图标完成更新 ,如果不想使用 使用配置 `spring.devtools.livereload.enable=false` 来关闭 

>   建议使用 LiveReload 来重载静态资源 因为他比项目重启的 时间要快

## 多模块管理

如果遇到多模块项目想要统一配置 在个人用户目录下创建 `spring-boot-devtools.properties`  这个文件适用于计算机上的所有devtools 项目













