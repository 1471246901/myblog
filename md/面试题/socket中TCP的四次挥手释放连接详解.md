# socket中TCP的四次挥手释放连接详解

![image-20201125161851349](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20201125161851349.png)

socket中发送的TCP四次挥手

图示过程如下：

-   某个应用进程首先调用close主动关闭连接，这时TCP发送一个FIN M；
-   另一端接收到FIN M之后，执行被动关闭，对这个FIN进行确认。它的接收也作为文件结束符传递给应用进程，因为FIN的接收意味着应用进程在相应的连接上再也接收不到额外数据；
-   一段时间之后，接收到文件结束符的应用进程调用close关闭它的socket。这导致它的TCP也发送一个FIN N；
-   接收到这个FIN的源发送端TCP对它进行确认。