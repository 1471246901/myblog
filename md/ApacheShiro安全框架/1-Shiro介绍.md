# Shiro介绍

安全是企业应用中不可缺少的功能，在众多权限框架中，Shiro（其前身是[JSecurity](http://www.oschina.net/p/jsecurity)）因其简单而又不失强大的特点引起了不少开发者的注 意。随着Grails的关注度越来越高，在Grails社区也出现了Shiro的插件。

Shiro最早的名字是JSecurity，后来更名为Shiro并成为Apache的孵化项目。这次改名也同样影响了Grails Shiro  Plugin。它最早在Shiro还未改名之前就已经存在了，后来因为Shiro的名字变更，也就一道跟着&ldquo;改名换姓&rdquo;。由于Grails Shiro  Plugin中已经包含了Shiro相关的Jar，因此对于插件的使用者而言，不必专门下载Shiro。

#### Shiro架构理解

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/12154206_OzSF.png)

##### **Subject**：

主体，可以看到主体可以是任何可以与应用交互的“用户”；

##### **SecurityManager** ：

 相 当 于 SpringMVC 中 的 DispatcherServlet 或 者 Struts2 中的

##### **FilterDispatcher**；

是 Shiro 的心脏；所有具体的交互都通过 SecurityManager 进行控制；它管理着所有 Subject、且负责进行认证和授权、及会话、缓存的管理。

##### **Authenticator**：

认证器，负责主体认证的，这是一个扩展点，如果用户觉得 Shiro 默认的不好，可以自定义实现；其需要认证策略（Authentication Strategy），即什么情况下算用户认证通过了；

##### **Authrizer**：

授权器，或者访问控制器，用来决定主体是否有权限进行相应的操作；即控制着用户能访问应用中的哪些功能；

##### **Realm**：

可以有 1 个或多个 Realm，可以认为是安全实体数据源，即用于获取安全实体的；可以是 JDBC 实现，也可以是 LDAP 实现，或者内存实现等等；由用户提供；注意：Shiro不知道你的用户/权限存储在哪及以何种格式存储；所以我们一般在应用中都需要实现自己的 Realm；

##### **SessionManager**：

如果写过 Servlet 就应该知道 Session 的概念，Session 呢需要有人去管理它的生命周期，这个组件就是 SessionManager；而 Shiro 并不仅仅可以用在 Web 环境，也可以用在如普通的 JavaSE 环境、EJB 等环境；所有呢，Shiro 就抽象了一个自己的 Session来管理主体与应用之间交互的数据；这样的话，比如我们在 Web 环境用，刚开始是一台Web 服务器；接着又上了台 EJB 服务器；这时想把两台服务器的会话数据放到一个地方，
这个时候就可以实现自己的分布式会话（如把数据放到 Memcached 服务器）；

##### **SessionDAO**：

DAO 大家都用过，数据访问对象，用于会话的 CRUD，比如我们想把 Session保存到数据库，那么可以实现自己的 SessionDAO，通过如 JDBC 写到数据库；比如想把Session 放到 Memcached 中，可以实现自己的 Memcached SessionDAO；另外 SessionDAO中可以使用 Cache 进行缓存，以提高性能；

##### **CacheManager**：

缓存控制器，来管理如用户、角色、权限等的缓存的；因为这些数据基本
上很少去改变，放到缓存中后可以提高访问的性能

##### **Cryptography**：

密码模块，Shiro 提高了一些常见的加密组件用于如密码加密

#### Shiro 图解

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/20181205205704182.png)

##### **Authentication**：

身份认证/登录，验证用户是不是拥有相应的身份；

##### **Authorization**：

授权，即权限验证，验证某个已认证的用户是否拥有某个权限；即判断用
户是否能做事情，常见的如：验证某个用户是否拥有某个角色。或者细粒度的验证某个用
户对某个资源是否具有某个权限；

##### **Session Manager**：

会话管理，即用户登录后就是一次会话，在没有退出之前，它的所有信
息都在会话中；会话可以是普通 JavaSE 环境的，也可以是如 Web 环境的；

##### **Cryptography**：

加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存储；

##### **Web Support**：

Web 支持，可以非常容易的集成到 Web 环境；

##### **Caching**：

缓存，比如用户登录后，其用户信息、拥有的角色/权限不必每次去查，这样可以提高效率；

##### **Concurrency**：

shiro 支持多线程应用的并发验证，即如在一个线程中开启另一个线程，能
把权限自动传播过去；

##### **Testing**：

提供测试支持；

##### **Run As**：

允许一个用户假装为另一个用户（如果他们允许）的身份进行访问；

##### **Remember Me**：

记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用登录

**记住一点，Shiro 不会去维护用户、维护权限；这些需要我们自己去设计/提供；然后通过**
**相应的接口注入给 Shiro 即可。**



