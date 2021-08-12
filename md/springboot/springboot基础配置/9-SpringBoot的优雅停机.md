# SpringBoot的优雅停机

SpringBoot的优雅停机 `actuator-starter` 包的端点  /actuator/shutdown 接口并未实现优雅停机

##### 1 当容器为tomcat时使用SpringBoot注册以下组件

当接收到关闭信号时拒绝新的请求,并且完成现有请求,最多等待现有请求30秒 或者`shutdown-wait-second` 秒后停止

```java

/**
 * @author gushiyu
 * @Date: 2021/8/11
 * @Time: 4:42 下午
 * @Desc: 控制系统在处理完所有请求后才关机，对kill -9 无效
 */
@Configuration
public class ShortDownConfig {

    @Autowired
    private GracefulShutdownTomcat gracefulShutdownTomcat;

    @Bean
    public ServletWebServerFactory servletContainer() {
        TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
        tomcat.addConnectorCustomizers(gracefulShutdownTomcat);
        return tomcat;
    }

}

```

```java

/**
 *  关闭 Spring Boot tomcat
 */

@Component
public class GracefulShutdownTomcat implements TomcatConnectorCustomizer, ApplicationListener<ContextClosedEvent> {
    private final Logger log = LoggerFactory.getLogger(GracefulShutdownTomcat.class);
    private volatile Connector connector;

    @Value("${shutdown-wait-second:30}")
    private int waitTime;

    @Override
    public void customize(Connector connector) {
        this.connector = connector;
    }
    @Override
    public void onApplicationEvent(ContextClosedEvent contextClosedEvent) {
        this.connector.pause();
        Executor executor = this.connector.getProtocolHandler().getExecutor();
        if (executor instanceof ThreadPoolExecutor) {
            try {
                ThreadPoolExecutor threadPoolExecutor = (ThreadPoolExecutor) executor;
                threadPoolExecutor.shutdown();
                if (!threadPoolExecutor.awaitTermination(waitTime, TimeUnit.SECONDS)) {
                    log.warn("Tomcat thread pool did not shut down gracefully within " + waitTime + " seconds. Proceeding with forceful shutdown");
                }
            } catch (InterruptedException ex) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

