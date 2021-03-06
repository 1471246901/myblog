# Redis哨兵模式

#### 概念

##### 哨兵模式是redis`高可用`的实现方式之一
 使用一个或者多个哨兵(Sentinel)实例组成的系统，对redis节点进行监控，在主节点出现故障的情况下，能将从节点中的一个升级为主节点，进行故障转移，保证系统的可用性。

![image-20201214212721027](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20201214212721027.png)

##### 哨兵们是怎么感知整个系统中的所有节点(主节点/从节点/哨兵节点)的


1.  首先主节点的信息是配置在哨兵(Sentinel)的配置文件中
2.  哨兵节点会和配置的主节点建立起两条连接`命令连接`和`订阅连接`
3.  哨兵会通过`命令连接`每10s发送一次`INFO`命令，通过`INFO命令`，主节点会返回自己的run_id和自己的`从节点信息`
4.  哨兵会对这些从节点也建立两条连接`命令连接`和`订阅连接`
5.  哨兵通过`命令连接`向从节点发送`INFO`命令，获取到他的一些信息
     `run_id` , `role` , `从服务器的复制偏移量 offset`  等
6.  因为哨兵对与集群中的其他节点(主从节点)当前都有两条连接，`命令连接`和`订阅连接`
    
     1.  通过`命令连接`向服务器的`_sentinel:hello`频道发送一条消息，内容包括自己的ip端口、run_id、配置纪元(后续投票的时候会用到)等
    
    2.  通过`订阅连接`对服务器的`_sentinel:hello`频道做了监听，所以所有的向该频道发送的哨兵的消息都能被接受到
    3.  解析监听到的消息，进行分析提取，就可以知道还有那些别的哨兵服务节点也在监听这些主从节点了，更新结构体将这些哨兵节点记录下来
    4.  向观察到的其他的哨兵节点建立`命令连接`----没有`订阅连接`

##### 哨兵模式下的故障迁移
-   主观下线

    哨兵(Sentinel)节点会每秒一次的频率向建立了命令连接的实例发送PING命令，如果在`down-after-milliseconds`毫秒内没有做出有效响应包括(PONG/LOADING/MASTERDOWN)以外的响应，哨兵就会将该实例在本结构体中的状态标记为`SRI_S_DOWN`主观下线

-   客观下线

    当一个哨兵节点发现主节点处于主观下线状态是，会向其他的哨兵节点发出询问，该节点是不是已经主观下线了。如果超过配置参数`quorum`个节点认为是主观下线时，该哨兵节点就会将自己维护的结构体中该主节点标记为`SRI_O_DOWN`客观下线        询问命令`SENTINEL is-master-down-by-addr    `

##### leader选举


```
在认为主节点`客观下线`的情况下,哨兵节点节点间会发起一次选举，命令还是上面的命令`SENTINEL is-master-down-by-addr    `,只是`run_id`这次会将`自己的run_id`带进去，希望接受者将自己设置为主节点。如果超过半数以上的节点返回将该节点标记为leader的情况下，会有该leader对故障进行迁移
```

##### 故障迁移步骤

1.  在从节点中挑选出新的主节点
     a. 通讯正常
     b. 优先级排序
     c. 优先级相同是选择offset最大的
2.  将该节点设置成新的主节点 `SLAVEOF no one`,并确保在后续的INGO命令时，该节点返回状态为master
3.  将其他的从节点设置成从新的主节点复制, `SLAVEOF命令`
4.  将旧的主节点变成新的主节点的从节点

##### 哨兵模式优点

高可用,能实现主节点故障时能实现故障的转移

##### 哨兵模式缺点

没有办法做到水平扩展

#### 哨兵模式搭建

-   在主从复制基础上运行哨兵哨兵

-   主节点   127.0.0.1:  6379
    
    ```
    bind [从数据库的ip ....]  #注释后所有ip可访问 
    masterauth [master-password] #从节点连接节点密码
    repl-backlog-ttl 3600   # 在某些时候，master 不再连接 slaves，backlog 将被释放。
                                            # 这里是 3600 释放时间
    repl-backlog-size 1mb   #  设置主从复制容量大小。这个 backlog 是一个用来在 slaves 被断开连接时  backlog 是master向slaves发送的写命令集合,是在slave断开主节点时的写增量
    protected-mode [yes]    # 在没有启用 bind 和 访问密码的时候redis会禁止外网访问reids
    ```
    
    
    
-   从节点    127.0.0.1  6380
    ```
    slaveof [127.0.0.1:  6379] #主节点的位置 
    masterauth #当作为从节点是连接所使用的的密码
    ```
    
-   从节点    127.0.0.1   6381
    
    ```
    slaveof [ 127.0.0.1:  6379]  # 主节点的位置 
    masterauth #当作为从节点是连接所使用的的密码
    ```

-   哨兵  127.0.0.1 26379  26380    26381    三个

    ```
    # 哨兵模式端口
    port 26379
    # 禁止保护模式
    protected-mode no
    # 配置监听的主服务器，这里sentinel monitor代表监控，mymaster代表服务器的名称，可以自定义，192.168.11.128代表监控的主服务器，6379代表端口，2代表只有两个或两个以上的哨兵认为主服务器不可用的时候，才会进行failover操作。
    sentinel monitor mymaster 127.0.0.1 6379 2
    # sentinel author-pass定义服务的密码，mymaster是服务名称，root是Redis服务器密码
    # sentinel auth-pass <master-name> <password>
    sentinel auth-pass mymaster root
    
    # 绑定的主机地址
    # bind 127.0.0.1 192.168.1.1
    
    ```

-   先运行主节点 在运行从节点 最后运行哨兵

    ![image-20200330214200192](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200330214200192.png)

    可以从控制台发现一个哨兵发现了主节点 还有 两个从节点  下面又发现了两个哨兵

-   关闭主节点 30 秒试试

    ```
    ps -aef | grep redis
    kill -9 进程号
    ```

    ![image-20200330215001750](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200330215001750.png)

    此时哨兵把6380 从节点提升到主节点了

    ![image-20200330215250419](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200330215250419.png)

    现在在重新启动一下 6379 节点

    ![image-20200330215543065](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200330215543065.png)

    

