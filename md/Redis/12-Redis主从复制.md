# Reids主从复制

-   #### 概述

    -   redis 的复制功能是支持多个数据库之间的同步.一类是主数据库`master` ,一类是从数据库`slave`.主数据库可以进行读写操作,当发生写操作的时候自动将数据同步到从数据库.从数据库一般是只读的,并接受主数据库同步过来的数据,一个主数据库可以有多个 从数据库,但是一个从数据库只能有一个主数据库.

    -   通过主从复制可以很好的实现读写分离,提高服务器负载能力.主数据库主要进行写操作,读操作一般在从数据库中进行

    -   主从复制过程

        <img src="D:\Typora\data\image\20160503192249524" alt="img" style="zoom: 67%;" />

        -   当一个从数据库中启动的时候,会向主数据库发送sync(同步)命令
        -   主数据库接收到sync命令之后会开始在后台保存快照,(rdb操作) 并将保存期间的命令缓存起来一并发送给从数据库.
        -   从数据库接收到快照和缓存文件后将数据恢复到从数据库.
        -   主数据库在写操作的时候,会发出写命令给从数据库

    -   redis 主从同步策略

        -   主从刚连接的时候,进行全量同步,全同步结束后,进行增量同步, 如有需要从节点也可以进行全量同步.redis的策略是无论如何,首先会尝试进行增量同步,如果不成功,从数据库进行全量同步
        -   注意: 如果多个slave 断线了,需要重启的时候,因为只要slave 启动就会进行全同步,如果过多和从节点同时进行全同步可能导致主节点宕机.

    -   reids 主从复制环境配置

        -   主节点   127.0.0.1:  6379

            要修改的配置

            ```properties
            bind [从数据库的ip ....]  #注释后所有ip可访问 
            masterauth [master-password] #从节点连接节点密码
            repl-backlog-ttl 3600   # 在某些时候，master 不再连接 slaves，backlog 将被释放。
                                    # 这里是 3600 释放时间
             repl-backlog-size 1mb   #  设置主从复制容量大小。这个 backlog 是一个用来在 slaves 被断开连接时  backlog 是master向slaves发送的写命令集合,是在slave断开主节点时的写增量
            protected-mode [yes]    # 在没有启用 bind 和 访问密码的时候redis会禁止外网访问reids
            
           ```
           
        
           
        -   从节点    127.0.0.1  6380
        
            配置
        
            ```
            slaveof [127.0.0.1:  6379] #主节点的位置 
            masterauth #当作为从节点时连接所使用的的密码
            ```
        
        -   从节点    127.0.0.1   6381
        
            配置
            
            ```
            slaveof [ 127.0.0.1:  6379]  # 主节点的位置 
            masterauth #当作为从节点是连接所使用的的密码
            ```
            
        -   先启动主节点 后启动从节点
        
        -   从节点写入测试
        
            ![image-20200329194601271](D:\Typora\data\image\image-20200329194601271.png)
        
        -   主节点同步测试
        
            ![image-20200329194801604](D:\Typora\data\image\image-20200329194801604.png)
        
            ![image-20200329194834395](D:\Typora\data\image\image-20200329194834395.png)
        
            现在主节点插入数据,再在从节点查询,可以查询到
        
        -   使用 info replication查看节点信息 
        
            ![image-20200329195319191](D:\Typora\data\image\image-20200329195319191.png)
        
            ```
            从节点显示的信息
            role:slave #实例的角色，是master or slave
            master_host:192.168.64.102 #此节点对应的master的ip
            master_port:9021 #此节点对应的master的port
            master_link_status:up #slave端可查看它与master之间同步状态,当复制断开后表示down
            master_last_io_seconds_ago:0 #主库多少秒未发送数据到从库?
            master_sync_in_progress:0 #从服务器是否在与主服务器进行同步
            slave_repl_offset:6713173818 #slave复制偏移量
            slave_priority:100 #slave优先级
            slave_read_only:1 #从库是否设置只读
            connected_slaves:0 #连接的slave实例个数
            master_repl_offset:0
            repl_backlog_active:0 #复制积压缓冲区是否开启
            repl_backlog_size:134217728 #复制积压缓冲大小
            repl_backlog_first_byte_offset:0 #复制缓冲区里偏移量的大小
            repl_backlog_histlen:0 #此值等于 master_repl_offset - repl_backlog_first_byte_offset,该值不会超过repl_backlog_size的大小
            ```
        
            ![image-20200329195444735](D:\Typora\data\image\image-20200329195444735.png)
        
            ```
            主节点显示的信息
            role:master #实例的角色，是master or slave
            connected_slaves:1 #连接的slave实例个数
            slave0:ip=192.168.64.104,port=9021,state=online,offset=6713173004,lag=0 #lag从库多少秒未向主库发送REPLCONF命令
            master_repl_offset:6713173145 #主从同步偏移量,此值如果和上面的offset相同说明主从一致没延迟
            repl_backlog_active:1 #复制积压缓冲区是否开启
            repl_backlog_size:134217728 #复制积压缓冲大小
            repl_backlog_first_byte_offset:6578955418 #复制缓冲区里偏移量的大小
            repl_backlog_histlen:134217728 #此值等于 master_repl_offset - repl_backlog_first_byte_offset,该值不会超过repl_backlog_size的大小
            ```
    

>   [Redis 集群规范](http://doc.redisfans.com/topic/cluster-spec.html)
>
>   [Redis主从复制和集群配置](https://blog.csdn.net/u011204847/article/details/51307044)
>
>   [Redis 的主从同步，及两种高可用方式](https://blog.csdn.net/weixin_42711549/article/details/83061052)

**错误**:因为6381的从节点没有配置slaveof 所以才会显示只有一个从节点