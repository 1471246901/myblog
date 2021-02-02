# curator简介

zookeeper api 有很多不足，所以Netflix （网飞）自己封装了一下，然后就起名curator，后来就开源给 apache 基金会，curator 实现了好多工具，并且能够更灵活的操作zookeeper中的节点和数据(比如：分布式锁服务、集群领导选举、 共享计数器、缓存机制、分布式队列等)，并且是Fluent风格的API

## 原生zookeeperAPI的不足

连接对象异步创建，需要开发人员自行编码等待 

连接没有自动重连超时机制 

watcher一次注册生效一次 

不支持递归创建树形节点

## curator解决了那些问题

解决session会话超时重连

解决watcher需要反复注册才能一直监听

简化了开发api 

遵循Fluent风格的API 

提供了分布式锁服务、共享计数器、缓存机制等机制



## curator maven依赖

```xml
<dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.6.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>5.0.0</version>
            <type>jar</type>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>5.0.0</version>
            <type>jar</type>
        </dependency>
```

