# Sentinel与OpenFeign(Feign)

配置中开启feign对Sentinel的支持既可

```yml
feign:
  sentinel:
    enabled: true
```

其余按照feign配置既可

[](..\..\服务调用\Feign和OpenFeign\openfeign简单使用.md)

## fallback降级方法的配置

此时我们遇到了问题 , 就是这时的service 是接口的形式,我们无法在其中创建方法

这时我们可以创建一个类 继承自 我们的service接口

```java
/**
 * 这时我们可以将Fallback 单独生成一个类 分离后更清晰
 */
@Component
public class PaymentServiceFallback implements PaymentService {
    @Override
    public String test(int id) {
        return "触发了服务降级 在PaymentServiceFallback 中的test 方法 id:"+id;
    }
}
```

在 PaymentService 接口上标注 `@FeignClient` 的注解加上 fallback 属性 并指向我们刚刚创建的类 `PaymentServiceFallback` ,这时我们由Feign 进行 Fallback的执行,

也可以使用@SentinelResource 继续进行 Fallback 相关处理