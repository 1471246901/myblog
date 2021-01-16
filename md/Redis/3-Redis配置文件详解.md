# Redis配置文件详解

- 在windows下配置文件 名称为`redis.windows.conf`

- 在Linux下配置文件名称为`redis.conf`

- 配置文件默认放在服务端的同级目录下 `安装时组要赋值文件到该目录`

  ![image-20200316195547811](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200316195547811.png)

**redis 配置文件主要配置**

1. Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程
   
    ```shell
    daemonize yes [守护进程运行]
    ```
    
2. 当Redis以守护进程方式运行时，Redis默认会把pid写入`/var/run/redis.pid`文件，可以通过pidfile指定
   
    ```shell
    pidfile /var/run/redis.pid[相对目录]
    ```
    
3. 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字
   
    ```shell
    port 6379 [端口]
    ```
    
4. 绑定的主机地址
   
     ```shell
       bind 127.0.0.1 [主机地址,当其注释时 所有主机地址都可以连接]
     ```

5. 当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能

     ```shell
       timeout 300 [秒]
     ```

6. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose

     ```shell
     loglevel verbose  
     ```

7. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给`/dev/null`

     ```shell
     logfile stdout
     ```

8. 设置数据库的数量，默认数据库为0，可以使用SELECT <dbid>命令在连接上指定数据库id

     ```
     databases 16 
     ```

9. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合

     ```
     save <seconds> <changes>
       Redis默认配置文件中提供了三个条件：
       save 900 1
       save 300 10
       save 60 10000
      分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。
      都会触发持久化，未触发持久化关闭Redis将会有数据丢失
     ```

10. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大

     ```
     rdbcompression yes
     ```

11. 指定本地数据库文件名，默认值为dump.rdb

      ```
    dbfilename dump.rdb
      ```

12. 指定本地数据库存放目录

      ```
    dir ./  [本地数据库文件的存放目录]
      ```

13. 设置当本机为slav`主从机制`服务时，设置master`主节点`服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步

      ```
    slaveof <masterip> <masterport>  [主节点ip][主节点端口] 
      ```

14. 当master`主节点`服务设置了密码保护时，slave服务连接master的密码

      ```
    masterauth <master-password> [主节点密码]
      ```

15. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password>命令提供密码，默认关闭

      ```
    requirepass foobared [密码]
      ```

16. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回`max number of clients reached`错误信息

      ```
    maxclients 128
      ```

17. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区

      ```
    maxmemory <bytes>  
      ```

18. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no

      ```
    appendonly no
      ```

19. 指定更新日志文件名，默认为appendonly.aof

      ```
    appendfilename appendonly.aof
      ```

20. 指定更新日志条件，共有3个可选值： 

      ```shell
      > no：表示等操作系统进行数据缓存同步到磁盘（快） 
      > always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全） 
      > everysec：表示每秒同步一次（折衷，默认值）
      ```

      ```
    appendfsync everysec
      ```

21. 指定是否启用虚拟内存机制，默认值为no，`VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中`
    
    ```
        vm-enabled no
    ```

22. 虚拟内存文件路径，默认值为`/tmp/redis.swap`，不可多个Redis实例共享
    
    ```
        vm-swap-file /tmp/redis.swap
    ```

23. Redis将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0
    
    ```
    vm-max-memory 0
    ```

24. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享`类似文件系统的单元大小`，vm-page-size是要根据存储的 数据大小来设定的。**如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值**
    
    ```
        vm-page-size 32
    ```

25. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。
    
    ```
    vm-pages 134217728
    ```
    
26. 设置访问swap文件的线程数,**最好不要超过机器的核数**,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4
    
    ```
    vm-max-threads 4
    ```
    
27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启
    
    ```
    glueoutputbuf yes
    ```
    
28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法
    
    ```
hash-max-zipmap-entries 64
    hash-max-zipmap-value 512
    ```
    
29. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）
    
```
    activerehashing yes
```

30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
    
```
    include /path/to/local.conf
```

31. Redis的内存淘汰策略  Redis的内存淘汰策略是指在Redis的用于缓存的**内存不足**时，怎么处理需要新写入且需要申请额外空间的数据。

    ```
    maxmemory-policy noeviction [默认值noeviction]
    
    noeviction：当内存不足以容纳新写入数据时，新写入操作会报错。
    allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key。
    allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个key。
    volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的key。
    volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个key。
    volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除。
    ```

32. Redis的过期策略

    - 定时过期：每个设置过期时间的key都需要创建一个定时器，到过期时间就会立即清除。该策略可以立即清除过期的数据，对内存很友好；但是会占用大量的CPU资源去处理过期的数据，从而影响缓存的响应时间和吞吐量。

    - 惰性过期：只有当访问一个key时，才会判断该key是否已过期，过期则清除。该策略可以最大化地节省CPU资源，却对内存非常不友好。极端情况可能出现大量的过期key没有再次被访问，从而不会被清除，占用大量内存。

    - 定期过期：每隔一定的时间，会扫描一定数量的数据库的expires字典中一定数量的key，并清除其中已过期的key。该策略是前两者的一个折中方案。通过调整定时扫描的时间间隔和每次扫描的限定耗时，可以在不同情况下使得CPU和内存资源达到最优的平衡效果。
       (expires字典会保存所有设置了过期时间的key的过期时间数据，其中，key是指向键空间中的某个键的指针，value是该键的毫秒精度的UNIX时间戳表示的过期时间。键空间是指该Redis集群中保存的所有键。)

    - ```
      第一 第三、配置redis.conf 的hz选项，默认为10 （即1秒执行10次，100ms一次，值越大说明刷新频率越快，最Redis性能损耗也越大） 
      
      第二、配置redis.conf的maxmemory最大值，当已用内存超过maxmemory限定时，就会触发主动清理策略
      ```

      

    