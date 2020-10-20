# **Redis持久化**

>   数据存放于内存中,使用会非常高效,但是断电数据就会丢失,且内存相比硬盘总是很小的
>
>   数据存放于硬盘中,使用时速度慢与内存,但是硬盘存储数据量大,且断电不会丢失

**redis的两种持久化模式**

1.  RDB模式 (Redis默认的持久化模式)(整体快照)

    -   RDB相当于快照,保存的是一种状态.

    -   这种方式就是将内存中的数据以快照的方式写入到二进制文件中,默认的文件名为dump.rdb

    -   优点: 保存数据极快,还原数据极快

    -   优点: 适用于[灾难备份]([https://baike.baidu.com/item/%E7%81%BE%E9%9A%BE%E5%A4%87%E4%BB%BD%E6%8A%80%E6%9C%AF/8247757](https://baike.baidu.com/item/灾难备份技术/8247757)),保存关键的数据

    -   缺点: 小内存的机器不适合使用,RDB在符合机制的情况下就会快照

        -   当服务器正常关闭的时候,会进行快照

            shutdown 和flushall 都会触发快照

        -   当KEY满足一定条件的时候   

            如 :conf中的save 配置`save 900 1` `每900秒当一个key发生变化了`就会进行快照

2.  AOF模式(增量方式)

    -   由于快照方式是在一定间隔时间做一次的,如果reids意外关闭的话,就会损失最后一次快照后做的修改.如果应用要求不能丢失任何修改的话,可以采用AOF方式
    -   AOF 相当于日志方式 ,每当redis收到一个写命令时,都会将修改的操作通过write函数写入到文件中.当redis重启时会通过重新执行文件内的命令来在内存中重新构建数据库的内容.
    -   AOF 默认的文件名是appendonly.aof
    -   aof的三种方式  在 conf 中  appendonly yes // 启用aof持久化方式
        -   conf 中 appendfsync `always`  收到命令就写入磁盘 ,速度最慢,但是保证最完整的持久化
        -   conf 中 appendfsync `everysec` 每秒写入一次 ,在性能和持久化中保持折中
        -   conf 中 appendfsync `no` 完全依赖os,性能最好,持久化没有保证
    -   AOF模式的问题
        -   持久化文件会变得越来大(多次的修改操作,记录的是过程不是结果)

>   [redis持久化的几种方式](https://www.cnblogs.com/chenliangcl/p/7240350.html)
>
>   [redis持久化存储 与主从同步](https://blog.csdn.net/wq962464/article/details/90578484)

