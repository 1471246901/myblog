# Sentinel热点key

何为热点？热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：

-   商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
-   用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制

热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。

![Sentinel Parameter Flow Control](https://raw.githubusercontent.com/1471246901/myblog/master/img/sentinel-hot-param-overview-1.png)

Sentinel 利用 LRU 策略统计最近最常访问的热点参数，结合令牌桶算法来进行参数级别的流控。热点参数限流支持集群模式。

就是对某一热点进行限流

实现源码 com.alibaba.csp.sentinel.slots.block.BlockException

## 实现

### controller添加一个带参数的请求

```java
@GetMapping("/test/hotkey")
    @SentinelResource(value = "testHotKey",blockHandler = "testHotKeyBlockHandler")
    public String testHotKey(@RequestParam(value = "key1",required = false) String key1){
        return "testHotKeySuccess :"+key1;
    }

    public String testHotKeyBlockHandler(String key1, BlockException e){

        return "testHotKeyBlockHandler :"+key1+"/n"+e.getMessage();
    }
```

 **@SentinelResource(value = "testHotKey",blockHandler = "testHotKeyBlockHandler")**

**可以标识资源并且设置自定义的失败方法** 

**blockHandler**  这里的失败方法处理的是Sentinel 出错时调用的方法,所以出现运行时异常的话那么不会跳转到blockHandler  指定的方法  若想实现出现运行时异常那麽通过**fallback** 指定即可  

### 启动相关服务,添加热点key规则

这时可以通过 @SentinelResource 中表示的资源对相关限流做出设置

![image-20210221191554157](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210221191554157.png)

这时访问 `http://localhost:8001/test/hotkey` 无论怎么访问均不会限流

在访问 `http://localhost:8001/test/hotkey?key1=1` 多次访问之后就限流了

## 参数例外项

当我们希望这个限流的参数为某个值的时候,他的限流和平时指定的不同

例如 当 key1等与3时要支持 一秒一百下(QPS 达到 100)

![image-20210221192538885](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210221192538885.png)

访问 `http://localhost:8001/test/hotkey?key1=3` 的时候没有被限流

参数必须时基本类型和string