# 使用RESTful风格开发Java Web

#### 什么是RESTful风格?

REST是**REpresentational State Transfer**的缩写（一般中文翻译为表述性状态转移），REST 是一种体系结构，而 HTTP 是一种包含了 REST 架构属性的协议，为了便于理解，我们把它的首字母拆分成不同的几个部分：

-   **表述性（REpresentational）：** REST 资源实际上可以用各种形式来进行表述，包括 XML、JSON 甚至 HTML——最适合资源使用者的任意形式；
-   **状态（State）：** 当使用 REST 的时候，我们更关注资源的状态而不是对资源采取的行为；
-   **转义（Transfer）：** REST 涉及到转移资源数据，它以某种表述性形式从一个应用转移到另一个应用。

REST 就是将资源的状态以适合客户端或服务端的形式从服务端转移到客户端（或者反过来）。在 REST 中，资源通过 URL 进行识别和定位，**然后通过行为(即 HTTP 方法)来定义 REST 来完成怎样的功能。**

#### 例如:

在平时的 Web 开发中，method 常用的值是 GET 和 POST，但是实际上，HTTP 方法还有 PATCH、DELETE、PUT 等其他值，这些方法又通常会匹配为如下的 CRUD 动作:

| CRUD 动作 |  HTTP 方法   |
| :-------: | :----------: |
|  Create   |     POST     |
|   Read    |     GET      |
|  Update   | PUT 或 PATCH |
|  Delete   |    DELETE    |

![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/7896890-1273cfad7cda65ce.webp)

RESTful 等级

1.  每一个资源都有对应的url

2.  使用http方法进行不同的操作,使用http状态码来表示不同的结果

3.  使用超媒体,在资源的表达中包含了连接信息

    在返回的结果中有一些相关的api,以提供访问者别的需求