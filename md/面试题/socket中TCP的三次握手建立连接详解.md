# socket中TCP的三次握手建立连接详解

我们知道tcp建立连接要进行“三次握手”，即交换三个分组。大致流程如下：

-   客户端向服务器发送一个SYN J
-   服务器向客户端响应一个SYN K，并对SYN J进行确认ACK J+1
-   客户端再想服务器发一个确认ACK K+1

只有就完了三次握手，但是这个三次握手发生在socket的那几个函数中呢？请看下图：

![image-20201125161439468](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20201125161439468.png)

