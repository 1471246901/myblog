# Redis常用命令

redis 所有命令可以在 [redis命令](http://www.redis.cn/commands.html) 查看

----

1. ##### **Redis 键命令**

   - DEL `key`  

     该命令用于在key存在时删除key<u>不存在时不可以删除</u>

   - DUMP `key`

     序列化给定的key, 并返回被系列化的值，使用 [RESTORE](http://www.redis.cn/commands/restore) 命令可以将这个值反序列化为 Redis 键。

     序列化生成的值有以下几个特点：

     - 它带有 64 位的校验和，用于检测错误，[RESTORE](http://www.redis.cn/commands/restore) 在进行反序列化之前会先检查校验和。

     - 值的编码格式和 RDB 文件保持一致。

     - RDB 版本会被编码在序列化值当中，如果因为 Redis 的版本不同造成 RDB 格式不兼容，那么 Redis 会拒绝对这个值进行反序列化操作。

       序列化的值不包括任何生存时间信息。

   - EXISTS  `key` 

     检查给定的key是否存在 

   - **EXPIRE** `key`  `second秒`

     为给定的key 设置过期时间(以秒计)

     ```
              应用场景
          
               - 限时的优惠活动信息
               - 网站数据缓存(对于一些需要定时更新的数据,例如:日排行榜)
               - 手机验证码
               - 限制网站访客访问频率
     ```

   - PEXPIRE `key` `毫秒`

     为给定的key 设置过期时间(毫秒计)

   - TTL `key` 

     以秒为单位 ,返回给定key 的剩余生存周期

   - PTTL `key`

     以毫秒为单位,返回给定key 的剩余生命周期

   - PERSIST `key`

     移除key的过期时间,key将永久保存

   - KEYS `pattern`

     查找所有符合给定模式`pattern`的key

     通配符

     - `*`代表所有
     - `?`代表一个字符

   - RANDOMKEY

     从当前数据库中随即返回一个key

   - RENAME `key`

     修改key的名称

   - MOVE `key` `db`

     将当前数据库中的key移动到给定的数据库db当中

   - TYPE `key`

     返回key所储存的值的类型

   - SELECT `db`

     设置当前操作的数据库

2.  ##### **Redis string 命令**

    - SET `keyname` `keyvalue`

      设置给定的值,如果key已经储存值,则覆盖

      如果值是其他类型的则强制覆盖为string 类型

    - SETNX `key` `value`

      只有在key不存在时才会设定key的键值(**SET** IF **N**OT E**X**ISTS)

    - MSET `key` `value` `[key value...]`

      同时指定一个及以上的key - value 键值对

    - GET `keyname`

      Reids `get` 获取指定`key` 的值.如果`key` 不存在 ,返回`nil` .如果是其他类型的(非string 类型) ,返回错误

    - GETRANGE `key` `start` `end`

      获取字符串截取的一部分 按照下标索引的方式 包含`end` 

      负数偏移量表示从字符串最后开始计数， `-1` 表示最后一个字符， `-2` 表示倒数第二个，以此类推。

      `GETRANGE` 通过保证子字符串的值域(range)不超过实际字符串的值域来处理超出范围的值域请求。

    - GETBIT `key` `offset`

      对key 所储存的字符串的值,获取指定偏移量上的位(bit)

    - MGET `key` `[key ...]`

      获取一个及以上的key 的值

    - GETSET `key` `value`

      设置指定key的值 并返回key的旧值,key 不存在时 返回 `nil`

    - STRLEN `key` 

         返回key 所储存的字符串的长度

    - INCR `key`     INCR `key`  `增量`

      将key中的数字值增加1 或增量,如果key不存在则先被初始化为0 然后再增加1 或增量

    - DECR `key`    DECR `key`  `减值` 

      将key中的数字值减1 或减值,如果key不存在则先被初始化为0 然后再减1 或减值

    - **APPEND** `key` `appendstr`

      将`appendstr` 追加到指定key 的值的末尾 ,如不存在,为其赋值
   
3. ##### **Hash 命令**

   - HSET `key` `field` `value`  `[field  value....]`
     为指定的key 设定 FILD 和 VALUE ,可以同时设置多个
     
   - HGET `key` `field` 
     
     获取存储在HASH 中的fieid 中指定的值
   
   -   HMGET `key` `fieid` `[field....]`
   
       获取key所有给定字段的值
   
   -   HGETALL `key`
   
       获取key 中所有的**字段和值**
   
   -   HKEYS `key` 
   
       获取hash中所有的字段
   
   -   HLEN `key`
   
       获取hash中字段的数量
   
   -   HDEL `key` `field` `[fieid ....]`
   
       删除一个或多个hash中的字段
   
   -   HSETNX `key` `field` `value`
   
       只有当field 字段不存在时,设置哈希表的字段值
   
   -   HINCRBY `key` `field` `增量`
   
       为key中field 字段的整数值加上增量 
   
   -   HINCRBYFLOAT `key` `field` `增量`
   
       为key中field 字段的浮点数值加上增量
   
   -   HEXISTS `key` `fieid` 
   
       查看key中 field是否存在

4.  ##### **List 命令**

    -   BLPOP `key` `[key]` `timeout` 删

        移除列表的第一个或多个元素,如果列表没有元素会阻塞列表知道等待超时或发现可弹出元素为止

    -   BRPOP `key` `[key]` `timeout` 删

        移除列表的最后一个或多个元素,如果列表没有元素会阻塞列表知道等待超时或发现可弹出元素为止

    -   BRPOPLPUSH `列表一` `列表二` `timeout` 增 删 查 

        从列表中**取出**最后一个元素，并插入到另外一个列表的头部； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 

    -   LINDEX `key` `index` 查

        通过索引获取列表中的元素

    -   LINSERT `key`  `(BEFOR|AFTER)` `元素` `要插入的元素` 增

        在列表的元素前或者后插入元素。当指定元素不存在于列表中时，不执行任何操作。

    -   **LLEN `key`  查**

        获取列表长度

    -   LPOP `key`  删 查

        移出并获取列表的第一个元素,不会阻塞

    -   LPUSH `key` `value` `[value]`增

        将一个或者多个值插入到列表的头部,如果 key 不存在，一个空列表会被创建并执行 LPUSH 操作。 当 key 存在但不是列表类型时，返回一个错误。

    -   **LPUSHX `key` `value` `[value]` 增**

        将一个或者多个值插入到列表的头部,当key 存在且是list 类型是才会插入

    -   **LRANGE `key` `start` `stop`  查**

        获取列表start 开始到 stop 结束的元素

    -   **LREM `key` `count` `value`  删** 

        移除列表元素, 根据参数 COUNT 的值，移除列表中与参数 VALUE 相等的元素。

        COUNT 的值可以是以下几种：

        -   count > 0 : 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 COUNT 。
        -   count < 0 : 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 COUNT 的绝对值。
        -   count = 0 : 移除表中所有与 VALUE 相等的值。

    -   LTRIM `key` `start` `stop` 删

        通过start 和 stop 进行修剪, 只保留指定区域的元素 ,不在指定区域的元素都将被删除.

        也可以使用负数下标

    -   **RPOP `key` 删  查**

        移除并返回列表的最后一个元素,不会阻塞

    -   RPOPLPUSH  `列表一` `列表二` 删 查 增

        从列表一中取出最后一个元素,并将元素添加到另一个列表并返回,不会阻塞

    -   **RPUSH `key` `value` `[value ....]`  增**

        在列表中添加一个或多个值,插入末尾,如果列表不存在，一个空列表会被创建并执行 RPUSH 操作。 当列表存在但不是列表类型时，返回一个错误。

    -   **RPUSHX `key` `value` 增**

        在已存在的列表中添加值.

    -   **LSET `key` `index` `value`  改** 

        设置列表指定索引的值,如果指定索引不存在则报错

5.  ##### **Set 命令**

    -   SADD `key` `[value ...]`

        向集合添加一个或多个成员

    -   SCARD `key` 

        获取集合的成员数

    -   SDIFF `key1` `[otherkey ...]`

        返回给定集合的差集 如`key1` except(差运算) `[otherkey....]`

        不存在的集合 key 将视为空集。

    -   SDIFFSTORE `insertkey`  `key1` `[otherkey ...]`

        将给定集合之间的差集存储在指定的集合中。如果指定的集合 key 已存在，则会被覆盖。

        `{insertkey}` = `key1` except(差运算) `[otherkey ....]`

    -   SISMEMBER `key` `value` 

        判断value 是否在key 集合中

    -   SMEMBERS `key`

        返回集合中的所有成员

    -   SMOVE `fromkey` `insertkey` `value`

        将 `fromkey` 中的 `value` 移动到 `insertkey`

    -   SPOP `key`

        移除并返回 key 集合中的一个随机元素

    -   SINTER `key1` `[key2 ...]`

        返回`key1` 和`[key2 ...]` 的交集,`key1` `key2 ...` 中共同的

    -   SINTERSTORE `insertkey` `key1` `[key2 ...]`
    
    将 `key1` 和`[key2 ...]` 的交集 存入 `insertkey`  中
    
-   SRANDMEMBER `key` `[count]`
    
    返回集合中一个或者多个随机元素 并不会移除 
    
    -   count 是正数且小于集合基数，那么命令返回一个包含 count 个元素的数组，数组中的元素各不相同。如果 count 大于等于集合基数，那么返回整个集合
    
-   如果 count 为负数，那么命令返回一个数组，数组中的元素可能会重复出现多次，而数组的长度为 count 的绝对值
    
-   SUNION `key1` `[key ...]`
    
    返回所有key 的并集
    
    -   SUNIONSTORE `insertkey` `key1` `[key ...]`
    
    将`key1` 及所有集合 的并集存储在 `insertkey`中 `insertkey` 不存在将创建
    
-   SSCAN `key` `[cursor 游标]` `[MATCH pattern 过滤器]` `[COUNT count 最大返回数量]`
    
        该命令以及`SCAN类的命令`返回一个数组 第一个元素为迭代到的位置(执行一次 SCAN 类型的命令不是迭代整个key ,当第一个元素为0时才证明整个key 集合迭代完成 ) 返回的第二个元素是一个数组 代表迭代到的值的数组
    
    -   COUNT 参数的默认值为 10 。
        
        -   数据集比较大时，如果没有使用MATCH 选项, 那么命令返回的元素数量通常和 COUNT 选项指定的一样， 或者比 COUNT 选项指定的数量稍多一些。
    -   在迭代一个编码为整数集合（intset，一个只由整数值构成的小集合）、 或者编码为压缩列表（ziplist，由不同值构成的一个小哈希或者一个小有序集合）时， 增量式迭代命令通常会无视 COUNT 选项指定的值， 在第一次迭代就将数据集包含的所有元素都返回给用户。

          **SCAN命令的保证**
    
        -   从完整遍历开始直到完整遍历结束期间， 一直存在于数据集内的所有元素都会被完整遍历返回； 这意味着， 如果有一个元素， 它从遍历开始直到遍历结束期间都存在于被遍历的数据集当中， 那么 SCAN 命令总会在某次迭代中将这个元素返回给用户。
        -   同样，如果一个元素在开始遍历之前被移出集合，并且在遍历开始直到遍历结束期间都没有再加入，那么在遍历返回的元素集中就不会出现该元素。
    
        然而因为增量式命令仅仅使用游标来记录迭代状态， 所以这些命令带有以下缺点：
    
        -   **同一个元素可能会被返回多次。 处理重复元素的工作交由应用程序负责， 比如说， 可以考虑将迭代返回的元素仅仅用于可以安全地重复执行多次的操作上。**
        -   **如果一个元素是在迭代过程中被添加到数据集的， 又或者是在迭代过程中从数据集中被删除的， 那么这个元素可能会被返回， 也可能不会。**
    
        >   具体细节参照[SCAN命令](http://www.redis.cn/commands/scan.html)
    
6.  ##### **Sorted set 命令**   **Sorted set 的一个成员 含有 `value 值` 和 `number 分数`两个值，有序性是对于 `number 分数` 而说的。**

    -   ZADD `key` ` number value ` `[number value  ...]`

        向key 有序列表中插入一个或多个成员 ,已经存在的进行更新

    -   ZCARD `key` 

        获取key 中成员的数量

    -   ZCOUNT `key` `min` `max`  **根据分数**

        获取`key`中 从`min`到`max` 成员的数量,`min` `max` 为有序集合的分数

    -   ZINCRBY `key` `增量` `value`

        让`key` 中的成员`value` 加上增量 ,增量可以是负数

    -   ZINTERSTORE `insertkey`  `要运算的key的数量` `key`  `[key ...]` `[WEIGHTS weight [weight ...]]` `[AGGREGATE SUM|MIN|MAX]`

        计算给定的一个或多个有序集的交集,将该交集(结果集)储存到`insertkey`  ,

        -   对于要`运算的key的数量`  是key 的数量

        -   对于 `WEIGHTS`  选项 是进行交集运算时  `number` 值是传入聚合函数之前要乘以的因子，不同 `key`  可以有不同的因子，因子不指定默认为 1

        -   对于`AGGREGATE`  选项是聚合函数的选择，指多个key 之间相同的成员的number 之间如何运算 ，默认为和

    -   ZLEXCOUNT `key` `[minvalue` `[maxvalue`  **根据值**

        返回key集合中值从`minvalue`  到 `maxvalue ` 中的集合 

        -   **对于ZCOUNT  是比较 `number分数`  的值，  ZLEXCOUNT 比较`value内容`值**

        -   成员名称前需要加 `[` 符号作为开头, `[` 符号与成员之间不能有空格
        -   可以使用 `-` 和 `+` 表示得分最小值和最大值
        -   `min` 和 `max` 不能反, `max` 放前面 `min`放后面会导致返回结果为0
        -   计算成员之间的成员数量时,参数 `min` 和 `max` 的位置也计算在内。
        -   `min` 和 `max` 参数的含义与 `zrangebylex` 命令中所描述的相同

    -   ZRANGE `key` `start` `stop`  `WITHSCORES`  **根据排名**

        返回有序集中，指定区间内的成员。其中成员的位置按分数值递增**(从小到大)**来排序。

        -   具有相同分数值的成员按字典序(lexicographical order )来排列。

        -   如果你需要成员按值递减(从大到小)来排列,使用`ZREVRANGE`命令。

        -   下标参数 start 和 stop 都以 0 为底，也就是说，以 0 表示有序集第一个成员，以 1 表示有序集第二个成员，以此类推。

        -   你也可以使用负数下标，以 -1 表示最后一个成员， -2 表示倒数第二个成员，以此类推。
        -   可以传递`WITHSCORES`选项，以便将元素的分数与元素一起返回。这样，返回的列表将包含`value1,score1,...,valueN,scoreN`，而不是`value1,...,valueN`。 客户端类库可以自由地返回更合适的数据类型（建议：具有值和得分的数组或记录）。

    -   ZREVRANGE `key` `start` `stop`  `WITHSCORES`   **根据排名**

        返回有序集中指定区间内的成员（大到小 ） ，除了从大到小 ，其余的跟 `ZRANGE` 一样  

    -   ZREM `key` `value` `[value ...]`

        移除key 中一个或多个 成员，忽略未存在的成员 **值**

    -   ZREMRANGEBYLEX `key` `[minvalue` `[maxvalue`

        移除key集合中值从`minvalue`  到 `maxvalue ` 中的集合  **值**

        -   `min` 和 `max` 参数的含义与  `ZLEXCOUNT` ,  `zrangebylex`  命令中所描述的相同

    -   ZREMRANGEBYRANK `key` `start` `stop`

        移除成员，根据有序集合的**排名**从`start` 到`stop` 

    -   ZREMRANGEBYSCORE `key` `min`   `max`

        移除成员，根据有序集合的**分数**从`min`   `max`

    -   ZRANK  `key` `value`

        按照分数从小到大排名 返回`value`成员的排名

        从0开始

    -   ZREVRANK `key` `value`

        按照分数从大到小排名 返回`value`成员的排名

        从0开始

    -   ZSCORE `key` `value`

        在有序集中返回 `value` 成员的分数

    -   ZSCAN `key` `[cursor 游标]` `[MATCH pattern 过滤器]` `[COUNT count 最大返回数量]`

        类似 `SSCAN` 和 `SCAN` 迭代有序集合中的元素（包括元素成员和元素分值）

7. **HyperLogLog 命令 只关注数量不关注内容**

    -   PFADD `key` `元素` `[元素 ...]`

        添加 元素到 HyperLogLog key中

    -   PFCOUNT `key` 

        返回 key 的基数估算值

    -   PFMERGE `新的key` `key` `[key ...]`

        将多个 HyperLogLog key 合并

8.  其他命令 

    -   INFO [选项 | all ]

        显示数据库信息

        [常用的选项](https://www.cnblogs.com/web369/articles/8662036.html)

    -   exit  

        退出客户端