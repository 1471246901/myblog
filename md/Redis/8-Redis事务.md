# Redis 事务

-   #### Redis事务命令

    -   DISCARD 
        -   取消事务，放弃执行事务块中的所有事务
    -   EXEC 
        -   执行所有事务块内的命令
    -   MULTI 
        -   标记一个事务块的开始
    -   WATCH `key` `[key]`
        -   监视一个(或者多个)key , 如果在事务执行之前这个key被其他命令所改动,那么事务将被打断.
    -   UNWATCH
        -   取消`WATCH` 命令对所有的key的监视.

    

-   #### **Redis事务概念**

    -   可以一次执行多个命令,中途不被其他命令插入 ,能一次性,顺序性,排他性的执行一系列的命令.

-   #### **使用方法**

    -   正常进行Redis事务
        -   先使用`MULTI` 声明事务,然后在插入其他操作命令 ,最后在使用`EXEC` 提交命令
    -   放弃执行事务
        -   前使用`NULTI` 声明事务, 插入操作命令时如果想要放弃,则不执行`EXEC`命令,执行 `DISCARD`命令即可
    -   再事务提交`EXEC` 之前检查到错误 ,`整个事务的操作都不会执行`
        -   如果在插入操作命令时检查出错误则整个事务都不会执行
    -   在EXEC之后的执行阶段出现错误 , `整个操作会继续执行下去不会中断,也不会回滚`
    -   WATCH监控
        -   在声明事务前可以对某个(些)key 进行监控 ,如果值被改动则导致整个事务无法执行
        -   一旦在`WATCH` 之后执行`EXEC` 会导致所有WATCH 失效,导致之前加的监控锁都会失效
        -   watch 指令类似乐观锁 ,事务提交时如果key值已经被别的客户端修改则不会被执行.

**乐观锁和悲观锁**

>   实质上都是锁,一种侧重于主动获得资源的控制权以保证操作的成功,另一种则是未获得锁则不会进行本次的操作
>
>   **乐观锁**: 每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量
>
>   **悲观锁**: 每次去拿数据的时候都认为别人修改，所以每次在拿数据的时候都会上锁，这样如果中间有人想拿数据就会一直阻塞除非锁被释放获取到锁。传统的关系型数据库里，用到了很多种这种锁机制，比如行锁，表锁，写锁等

**Redis 中事务的特性**

-   redis中的事务不保证原子性:如果事务其中一条命令执行失败,其后的命令仍然会执行.
-   没有隔离级别的概念: 事务在没有提交之前实际上是不会被执行的.
-   单独的隔离操作: 事务中的所有命令都会被系列化,按顺序的执行,事务执行过程中不会被其他客户端的命令所打断

**应用场景**

-   商品秒杀情况,转账.



