# Ribbon的负载均衡规则

## IRule

Ribbon使用 IRule接口实现负载均衡

下图是Ribbon 自带类的继承关系

![image-20210202142526041](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210202142526041.png)



## Ribbon中自带的 IRule

| 类名                                    | 简介                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| com.netflix.loadbalancer.RoundRobinRule | 轮询                                                         |
| com.netflix.loadbalancer.RandomRule     | 随机                                                         |
| com.netflix.loadbalancer.RetryRule      | 先按照RoundRobinRule的策略获取服务,如果获取服务失败则在指定时间内进行重试,获取可用的服务 |
| WeightedResponseTimeRule                | 对RoundRobinRule的扩展,响应速度越快的实例选择权重越多大,越容易被选择 |
| BestAvailableRule                       | 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务,然后选择一个并发量最小的服务 |
| AvailabilityFilteringRule               | 先过滤掉故障实例,再选择并发较小的实例                        |
| ZoneAvoidanceRule                       | 默认规则,复合判断server所在区域的性能和server的可用性选择服务器 |

## 使用其他负载均衡规则

### 添加配置类`MyRuleConfig`

位于想要实现负载均衡项目的config包内

```java
@Configuration
public class MyRuleConfig {

    @Bean
    public IRule myRule(){
        return new RandomRule();
    }
}
```

### 将`MyRuleConfig` 类排除到扫描之外

因为

![image-20210202144051389](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210202144051389.png)

如果被扫描到,那么全局则都是用此规则

将注解加入启动类上

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient  

//表示excludeFilters 排除 ASSIGNABLE_TYPE 规则 的MyRuleConfig 类
@ComponentScan(excludeFilters = {@ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {MyRuleConfig.class})})
public class OrderMain8080 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain8080.class,args);
    }
}
```

此时可以验证 `MyRuleConfig` 不被扫描

### 将规则绑定到某一服务上

在启动类添加注解

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
@ComponentScan(excludeFilters = {@ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {MyRuleConfig.class})})

//表示对CLOUD-PAYMENT-SERVICE 服务,应用 MyRuleConfig中注入的规则
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE", configuration = MyRuleConfig.class)
public class OrderMain8080 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain8080.class,args);
    }
}
```

### 测试

访问 服务,查看是否按照自定的规则



## 自己编写负载均衡规则

![image-20210202142526041](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210202142526041.png)

由此可见我们自定义的规则需要集成自 IRule 或者其实现类

### 查看IRule接口

```java
public interface IRule {
    Server choose(Object var1);

    void setLoadBalancer(ILoadBalancer var1);

    ILoadBalancer getLoadBalancer();
}
```

其中有 `ILoadBalancer ` 实例, 通过 `choose` 方法实现选取某一server

Ribbon默认的轮询算法 `RoundRobinRule`

```java
public class RoundRobinRule extends AbstractLoadBalancerRule {

    private AtomicInteger nextServerCyclicCounter;
    private static final boolean AVAILABLE_ONLY_SERVERS = true;
    private static final boolean ALL_SERVERS = false;

    private static Logger log = LoggerFactory.getLogger(RoundRobinRule.class);

    public RoundRobinRule() {
        nextServerCyclicCounter = new AtomicInteger(0);
    }

    public RoundRobinRule(ILoadBalancer lb) {
        this();
        setLoadBalancer(lb);
    }

    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            log.warn("no load balancer");
            return null;
        }

        Server server = null;
        int count = 0;
        while (server == null && count++ < 10) {
            List<Server> reachableServers = lb.getReachableServers();
            List<Server> allServers = lb.getAllServers();
            int upCount = reachableServers.size();
            int serverCount = allServers.size();

            if ((upCount == 0) || (serverCount == 0)) {
                log.warn("No up servers available from load balancer: " + lb);
                return null;
            }

            int nextServerIndex = incrementAndGetModulo(serverCount);
            server = allServers.get(nextServerIndex);

            if (server == null) {
                /* Transient. */
                Thread.yield();
                continue;
            }

            if (server.isAlive() && (server.isReadyToServe())) {
                return (server);
            }

            // Next.
            server = null;
        }

        if (count >= 10) {
            log.warn("No available alive servers after 10 tries from load balancer: "
                    + lb);
        }
        return server;
    }

    /**
     * Inspired by the implementation of {@link AtomicInteger#incrementAndGet()}.
     *
     * @param modulo The modulo to bound the value of the counter.
     * @return The next value.
     */
    private int incrementAndGetModulo(int modulo) {
        for (;;) {
            int current = nextServerCyclicCounter.get();
            int next = (current + 1) % modulo;
            if (nextServerCyclicCounter.compareAndSet(current, next))
                return next;
        }
    }

    @Override
    public Server choose(Object key) {
        return choose(getLoadBalancer(), key);
    }

    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
    }
}
```

### 自己继承 `AbstractLoadBalancerRule` 或者 `IRule ` 实现算法

```java
@Slf4j
public class myRule extends AbstractLoadBalancerRule {

    private final AtomicInteger thisServerId = new AtomicInteger(0);



    @Override
    public Server choose(Object key) {

        return choose(getLoadBalancer(),key);
    }

    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            log.warn("没有 ILoadBalancer");
            return null;
        }

        Server server = null;

        List<Server> hasServerCount = lb.getReachableServers();
        int count =hasServerCount.size();
        if(count <= 0){
            log.warn("没有可用的Server");
            return null;
        }
        while(true){
            int thisServer = thisServerId.get();
            int nextServer = (thisServer+1)%count;
            if(thisServerId.compareAndSet(thisServer,nextServer)){
                server = hasServerCount.get(nextServer);
                break;
            }
        }
        if (server.isAlive() && (server.isReadyToServe())) {
            return (server);
        }
        log.warn("获取到的server没有存活或者未待命");
        return null;
    }
    
    @Override
    public void initWithNiwsConfig(IClientConfig iClientConfig) {

    }
}
```

### 在配置类 `MyRuleConfig` 中注入自己编写的负载均衡规则

```java
@Configuration
public class MyRuleConfig {

    @Bean
    public IRule myRule(){
        System.out.println("被注入了");
        return new myRule();
    }
}
```

### 按照上文`使用其他负载均衡规则` 实现并测试



