# zookeeper集群的广播协议

zab协议 的全称是 **Zookeeper Atomic Broadcast** （zookeeper原子广播）。 zookeeper 是通过 zab协议来保证分布式事务的最终一致性

基于zab协议，zookeeper集群中的角色主要有以下三类

如下表所示： zab广播模式工作原理，通过类似两阶段提交协议的方式解决数据一致性：

1.  leader从客户端收到一个写请求 
2.  leader生成一个新的事务并为这个事务生成一个唯一的ZXID 
3.  leader将这个事务提议(propose)发送给所有的follows节点 
4.   follower节点将收到的事务请求加入到历史队列(history queue)中
5.  follower节点发送ack给 leader 
6.  当leader收到大多数follower（半数以上节点）的ack消息，leader会发送commit请 求 
7.  当follower收到commit请求时，从历史队列中将事务请求commit

![image-20210127194714506](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210127194714506.png)

