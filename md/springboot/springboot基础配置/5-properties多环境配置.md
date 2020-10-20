# profile配置文件  多环境配置文件支持

## 多配置文件方式

1.  我们在配置文件编写时，文件名可以是application-环境.properties  或yml

```
dev 开发环境
prod 生产环境
```

​	但是spring boot 默认使用application.properties如和激活不同的配置文件

```
在主配置文件进行激活 使用 spring.profiles.active = 环境
 
```

## yaml 多文档块方式

1.  --- 三个横杠 可以把yml文件分成多个文档块

![spring  8083  spring :  pore 8e84  Spring  les : prod ](5-properties%E5%A4%9A%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE.assets/clip_image001-1586677503612.png)

```
在每个文档快标注上spring.profiles 为什么环境下使用
在第一个配置块用 spring.profiles.active = 环境 指定使用那个文档块
```

## 命令行方式

1.  1.  使用Java-jar方式启动 添加参数  --spring.profiles.active=指定环境

![img](5-properties%E5%A4%9A%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE.assets/clip_image002-1586677503612.png)