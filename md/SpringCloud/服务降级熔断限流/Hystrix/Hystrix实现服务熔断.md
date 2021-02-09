# Hystrix实现服务熔断

服务熔断我们已经在Hystrix 介绍内讲过，而且 服务熔断属于服务降级的一种，那么自然服务熔断也是使用`@HystrixCommand` 进行服务熔断没使用 `@HystrixProperty` 来制定熔断的规则

![image-20210205135932167](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210205135932167.png)

## 熔断的主要逻辑

![image-20210205140011472](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210205140011472.png)

细想以下一个超负荷工作的餐厅，在人数过多时已经提供不了服务，这个时候应该关店不让新的顾客在进店，再等些时间试着再让新顾客进店如果餐厅能正常服务，那么就会放开店门让更多人进来，如果新顾客依然无法服务，那么继续关闭店门等待餐厅能正常接受新顾客（另外在微服务中，这样的自我恢复应是自动的）

## 实现服务熔断

首先按照Hystrix 实现服务降级

### 在provider 提供者的 service中 添加服务熔断相关规则

```java
@Service
public class PaymentService {

    //开启服务熔断，在10秒内超过10 次请求 有60% 不能正常或及时响应，那么就进行服务熔断
    @HystrixCommand(fallbackMethod = "test_TimeOutHandler", commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"),
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60")
        }
    )
    public String test(int id){
        
        if(id == 1){
            throw new RuntimeException("eee");
        }

        return "this is cloud-provider-hystrix-payment8003/test id : "+id;
    }
```

## 测试

首先访问 `localhost:8003/test/1`  这时会有错误抛出，在10秒内访问10次，以此来触发熔断

![image-20210205142710851](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210205142710851.png)

这个时候在访问 `localhost:8003/test/0` 依旧会触发熔断

![image-20210205142727169](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210205142727169.png)

等5秒钟时间 在请求 `localhost:8003/test/0`

![image-20210205142742962](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210205142742962.png)

### 