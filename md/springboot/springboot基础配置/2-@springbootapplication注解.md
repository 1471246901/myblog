# @SpringBootApplication注解

@springbootapplication 注解是加载项目的启动类上的 而且@SpringBootApplication注解实际是一个组合注解,源码为

```java
/**
 * Indicates a {@link Configuration configuration} class that declares one or more
 * {@link Bean @Bean} methods and also triggers {@link EnableAutoConfiguration
 * auto-configuration} and {@link ComponentScan component scanning}. This is a convenience
 * annotation that is equivalent to declaring {@code @Configuration},
 * {@code @EnableAutoConfiguration} and {@code @ComponentScan}.
 *
 * @author Phillip Webb
 * @author Stephane Nicoll
 * @author Andy Wilkinson
 * @since 1.2.0
 */
@Target(ElementType.TYPE)    // 用于描述注解的使用范围（即：被描述的注解可以用在什么地方）
@Retention(RetentionPolicy.RUNTIME)   //用来指示其它注解类型保留的生命周期。
/**                 注解被保留的阶段   默认RUNTIME
                    public enum RetentionPolicy {
                        //注解只会存在源代码中，将会被编译器丢弃
                        SOURCE,
                        //注解将会保留到class文件阶段，但是在加载如vm的时候会被抛弃
                        CLASS,
                        //注解不单会被保留到class文件阶段，而且也会被vm加载进虚拟机的时候保留
                        RUNTIME
                    }
*/
@Documented    //指明修饰的注解，可以被例如javadoc此类的工具文档化，只负责标记，没有成员取值。
@Inherited     //指明  允许子类继承父类中的注解。
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

	/**
	 * Exclude specific auto-configuration classes such that they will never be applied.
	 * @return the classes to exclude
	 */
	@AliasFor(annotation = EnableAutoConfiguration.class)
	Class<?>[] exclude() default {};

	/**
	 * Exclude specific auto-configuration class names such that they will never be
	 * applied.
	 * @return the class names to exclude
	 * @since 1.3.0
	 */
	@AliasFor(annotation = EnableAutoConfiguration.class)
	String[] excludeName() default {};

	/**
	 * Base packages to scan for annotated components. Use {@link #scanBasePackageClasses}
	 * for a type-safe alternative to String-based package names.
	 * <p>
	 * <strong>Note:</strong> this setting is an alias for
	 * {@link ComponentScan @ComponentScan} only. It has no effect on {@code @Entity}
	 * scanning or Spring Data {@link Repository} scanning. For those you should add
	 * {@link org.springframework.boot.autoconfigure.domain.EntityScan @EntityScan} and
	 * {@code @Enable...Repositories} annotations.
	 * @return base packages to scan
	 * @since 1.3.0
	 */
	@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
	String[] scanBasePackages() default {};

	/**
	 * Type-safe alternative to {@link #scanBasePackages} for specifying the packages to
	 * scan for annotated components. The package of each class specified will be scanned.
	 * <p>
	 * Consider creating a special no-op marker class or interface in each package that
	 * serves no purpose other than being referenced by this attribute.
	 * <p>
	 * <strong>Note:</strong> this setting is an alias for
	 * {@link ComponentScan @ComponentScan} only. It has no effect on {@code @Entity}
	 * scanning or Spring Data {@link Repository} scanning. For those you should add
	 * {@link org.springframework.boot.autoconfigure.domain.EntityScan @EntityScan} and
	 * {@code @Enable...Repositories} annotations.
	 * @return base packages to scan
	 * @since 1.3.0
	 */
	@AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
	Class<?>[] scanBasePackageClasses() default {};

	/**
	 * The {@link BeanNameGenerator} class to be used for naming detected components
	 * within the Spring container.
	 * <p>
	 * The default value of the {@link BeanNameGenerator} interface itself indicates that
	 * the scanner used to process this {@code @SpringBootApplication} annotation should
	 * use its inherited bean name generator, e.g. the default
	 * {@link AnnotationBeanNameGenerator} or any custom instance supplied to the
	 * application context at bootstrap time.
	 * @return {@link BeanNameGenerator} to use
	 * @see SpringApplication#setBeanNameGenerator(BeanNameGenerator)
	 * @since 2.3.0
	 */
	@AliasFor(annotation = ComponentScan.class, attribute = "nameGenerator")
	Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

	/**
	 * Specify whether {@link Bean @Bean} methods should get proxied in order to enforce
	 * bean lifecycle behavior, e.g. to return shared singleton bean instances even in
	 * case of direct {@code @Bean} method calls in user code. This feature requires
	 * method interception, implemented through a runtime-generated CGLIB subclass which
	 * comes with limitations such as the configuration class and its methods not being
	 * allowed to declare {@code final}.
	 * <p>
	 * The default is {@code true}, allowing for 'inter-bean references' within the
	 * configuration class as well as for external calls to this configuration's
	 * {@code @Bean} methods, e.g. from another configuration class. If this is not needed
	 * since each of this particular configuration's {@code @Bean} methods is
	 * self-contained and designed as a plain factory method for container use, switch
	 * this flag to {@code false} in order to avoid CGLIB subclass processing.
	 * <p>
	 * Turning off bean method interception effectively processes {@code @Bean} methods
	 * individually like when declared on non-{@code @Configuration} classes, a.k.a.
	 * "@Bean Lite Mode" (see {@link Bean @Bean's javadoc}). It is therefore behaviorally
	 * equivalent to removing the {@code @Configuration} stereotype.
	 * @since 2.2
	 * @return whether to proxy {@code @Bean} methods
	 */
	@AliasFor(annotation = Configuration.class)
	boolean proxyBeanMethods() default true;

}
```

其中前四个都是元注解

## @SpringBootConfiguration

源码:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {

	/**
	 * Specify whether {@link Bean @Bean} methods should get proxied in order to enforce
	 * bean lifecycle behavior, e.g. to return shared singleton bean instances even in
	 * case of direct {@code @Bean} method calls in user code. This feature requires
	 * method interception, implemented through a runtime-generated CGLIB subclass which
	 * comes with limitations such as the configuration class and its methods not being
	 * allowed to declare {@code final}.
	 * <p>
	 * The default is {@code true}, allowing for 'inter-bean references' within the
	 * configuration class as well as for external calls to this configuration's
	 * {@code @Bean} methods, e.g. from another configuration class. If this is not needed
	 * since each of this particular configuration's {@code @Bean} methods is
	 * self-contained and designed as a plain factory method for container use, switch
	 * this flag to {@code false} in order to avoid CGLIB subclass processing.
	 * <p>
	 * Turning off bean method interception effectively processes {@code @Bean} methods
	 * individually like when declared on non-{@code @Configuration} classes, a.k.a.
	 * "@Bean Lite Mode" (see {@link Bean @Bean's javadoc}). It is therefore behaviorally
	 * equivalent to removing the {@code @Configuration} stereotype.
	 * @return whether to proxy {@code @Bean} methods
	 * @since 2.2
	 */
	@AliasFor(annotation = Configuration.class)
	boolean proxyBeanMethods() default true;

}
```

其实@springbootconfiguration 本质上就是  @Configuration   表明可以从这个类中配置bean  注入对象,从这个角度来讲这个类所具有的角色类似spring 中的applicationcontext.xml 文件的角色.



## @EnableAutoConfiguration

部分源码:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
```

### @AutoConfigurationPackage

@AutoConfigurationPackage注解的作用是将 **添加该注解的类所在的package** 作为 **自动配置package** 进行管理。也就是说当SpringBoot应用启动时默认会将启动类所在的package作为自动配置的package。

### @Import(AutoConfigurationImportSelector.class)

@import 注解 

@Import只能用在类上 ，

@Import通过快速导入的方式实现把实例加入spring的IOC容器中

**@Import注解可以用于导入第三方包** ，@Bean注解也可以，但是@Import注解快速导入的方式更加便捷

#### @Import 注解使用的三种方法

##### 直接填class数组方式

直接填对应的class数组，class数组可以有0到多个

语法如下：

```java
@Import({ 类名.class , 类名.class... })
public class TestDemo {

}
```

对应的import的bean都将加入到spring容器中，这些在容器中bean名称是该类的**全类名** ，比如com.yc.类名

##### <span style="color:red;">ImportSelector方式【重点】</span>  AutoConfigurationImportSelector.class 就是使用的这种方式

这种方式的前提就是一个类要实现ImportSelector接口，假如我要用这种方法，目标对象是Myclass这个类，分析具体如下：

创建Myclass类并实现ImportSelector接口

```javascript
public class Myclass implements ImportSelector {
//既然是接口肯定要实现这个接口的方法
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        return new String[0];
    }
}
```

分析实现接口的selectImports方法中的：

-   1、返回值： 就是我们实际上要导入到容器中的组件全类名【**重点** 】
-   2、参数： AnnotationMetadata表示当前被@Import注解给标注的所有注解信息【不是重点】

>   需要注意的是selectImports方法可以返回空数组但是不能返回null，否则会报空指针异常！



##### ImportBeanDefinitionRegistrar方式

同样是一个接口，类似于第二种ImportSelector用法，相似度80%，只不过这种用法比较自定义化注册，具体如下：

第一步：创建Myclass2类并实现ImportBeanDefinitionRegistrar接口

```java
public class Myclass2 implements ImportBeanDefinitionRegistrar {
//该实现方法默认为空
    @Override
    public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) {
      
    }
}
```

参数分析：

-   第一个参数：annotationMetadata 和之前的ImportSelector参数一样都是表示当前被@Import注解给标注的所有注解信息
-   第二个参数表示用于注册定义一个bean

第二步：编写代码，自定义注册bean

```java
public class Myclass2 implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) {
        //指定bean定义信息（包括bean的类型、作用域...）
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(TestDemo4.class);
        //注册一个bean指定bean名字（id）
        beanDefinitionRegistry.registerBeanDefinition("TestDemo4444",rootBeanDefinition);
    }
}
```

第三步：编写TestDemo 配置类，并标注上使用ImportBeanDefinitionRegistrar方式的Myclass2类

```java
@Import({TestDemo2.class,Myclass.class,Myclass2.class})
public class TestDemo {

        @Bean
        public AccountDao2 accountDao222(){
            return new AccountDao2();
        }

}
```



#### AutoConfigurationImportSelector.class   

AutoConfigurationImportSelector.class   中的部分内容

```java
@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}
```

## @ComponentScan

@ComponentScan注解用于自动扫描指定包下的所有组件，也可以通过添加属性值来指定扫描规则。

ComponentScan 提供扫描是的配置,可查看源码,最常用的

<span style="color:orange">不指定basePackages 时默认就是自己这个类所在的包作为扫描的包,所以建议把SpringBootApplication 注解的类放到跟包中</span>

### **@ComponentScan(basePackages="包名")**  直接指定包名

最简单的使用方法，扫描包名下的所有组件。

### **排除指定组件 excludeFilters = {@ComponentScan.Filter(type= FilterType.*,classes = ***.class),...}**

指定扫描规则，去除指定注解的组件。

如源码所示，excludeFilters属性是一个@ComponentScan.Filter注解数组，每个@ComponentScan.Filter之间用逗号分隔。  源码:

```java
/**
	 * Specifies which types are not eligible for component scanning.
	 * @see #resourcePattern
	 */
	Filter[] excludeFilters() default {};

```



#### @ComponentScan.Filter 注解,制定规则

先来看type属性，它是FilterType类型的，而FilterType是一个枚举类型，一种有五个值。分别是FitlerType.ANNOTATION：指定注解

FilterType.ASSIGNABLE_TYPE：指定类型

FilterType.ASPECTJ：Aspectj语法指定，很少使用

FilterType.REGEX：使用正则表达式指定

FilterType.CUSTOM：自定义规则

### 指定扫描规则,只注入符合注解条件的组件   includeFilters = {@ComponentScan.Filter(type= FilterType.*****,classes = ***.class),...}，

这个与excludeFilters用法相似，但是有一个注意点，用这个属性时，需要添加一个属性。因为spring默认是注入所有扫描到的组件，**所以要将useDefaultFilters 的属性值设置为false**，includeFilters才能生效。





@ComponentScan 注解 会扫描  @Service  @Repository  @Component  @Controller  @Configuration  这些注解