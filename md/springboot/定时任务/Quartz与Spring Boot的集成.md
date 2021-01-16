# Quartz与Spring Boot的集成

与SpringBoot集成主要要注入三个类  `JobDetail`、`Trigger`、`SchedulerFactory` 



## 基础配置

### maven依赖

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```





### 编写Job

实现接口  `org.quartz.Job`

```java
public class MyJob implements Job {

	@Override
	public void execute(JobExecutionContext context) throws JobExecutionException {
		SimpleDateFormat simpleDateFormat = new SimpleDateFormat("YYYY-MM-DD-hh:mm:ss");
		
		System.out.println(simpleDateFormat.format(new Date())+"implements Job被调用的时候执行该该方法...");

	}

}
```

实现抽象类 `org.springframework.scheduling.quartz.QuartzJobBean` 的Job

```java
public class Job2 extends QuartzJobBean {

	@Override
	protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
		SimpleDateFormat simpleDateFormat = new SimpleDateFormat("YYYY-MM-DD-hh:mm:ss");

		System.out.println(simpleDateFormat.format(new Date()) + "extends QuartzJobBean被调用的时候执行该该方法...");

	}

}
```

不实现接口的Job

```java
//@Component可以加入
public class Job3 {
	
	public void todo() {
		SimpleDateFormat simpleDateFormat = new SimpleDateFormat("YYYY-MM-DD-hh:mm:ss");

		System.out.println(simpleDateFormat.format(new Date()) + "容器管理的Job被调用的时候执行该该方法...");

		
	}
	
}
```

### 创建配置类`QuartzConfig` 对JobDetail,Trigger和进行配置

```java
@Configuration
public class QuartzConfig {

	/**
	 * 此种方法无法为任务传送数据，只能在job内实现获取数据
	 * 
	 * @return MethodInvokingJobDetailFactoryBean 产生 Job3 的 JobDetail
	 */
	@Bean
	MethodInvokingJobDetailFactoryBean job3JobDetail() {
		MethodInvokingJobDetailFactoryBean factoryBean = new MethodInvokingJobDetailFactoryBean();
		factoryBean.setTargetClass(Job3.class);
		factoryBean.setTargetMethod(Job3.class.getMethods()[0].getName());
		return factoryBean;
	}

	@Bean
	@ConfigurationProperties(prefix = "job2.datamap")
	Map<String, String> job2DataMap() {
		return new HashMap<>();
	}

	/**
	 * 此方法可以传递参数，可以从配置文件中获取参数
	 * 
	 * @param datamap
	 * @return job2 的JobDetailFactoryBean
	 */
	@Bean
	JobDetailFactoryBean job2JobDetail(@Qualifier("job2DataMap") Map<String, String> datamap) {
		JobDetailFactoryBean factoryBean = new JobDetailFactoryBean();
		factoryBean.setJobClass(Job2.class);
		factoryBean.setJobDataAsMap(datamap);
		factoryBean.setDurability(true);
		return factoryBean;
	}

	@Bean
	SimpleTriggerFactoryBean simpleTriggerFactoryBean(MethodInvokingJobDetailFactoryBean bean) {
		SimpleTriggerFactoryBean simpleFactoryBean = new SimpleTriggerFactoryBean();
		simpleFactoryBean.setJobDetail(bean.getObject());
		simpleFactoryBean.setRepeatCount(3);// 执行次数
		simpleFactoryBean.setStartDelay(1000);// 延迟执行毫秒
		simpleFactoryBean.setRepeatInterval(2000);// 重复间隔
		return simpleFactoryBean;
	}

	@Bean
	CronTriggerFactoryBean cronTriggerFactoryBean(JobDetailFactoryBean bean) {
		CronTriggerFactoryBean factoryBean = new CronTriggerFactoryBean();
		factoryBean.setJobDetail(bean.getObject());
		factoryBean.setCronExpression("* * * * * ?");
		return factoryBean;
	}

    //在这可以对自动注入的SchedulerFactoryBean进行配置
	@Autowired
	@Order(Integer.MAX_VALUE)
	public void name(@Qualifier("quartzScheduler") SchedulerFactoryBean factory, ApplicationContext context) {
		System.out.println("ssssssssssssssssssssssss" + factory);
		System.out.println(context);
		// 在这设置Scheduler相关参数
//		factory.setJobFactory(autoWiredSpringBeanToJobFactory);
//	    factory.setOverwriteExistingJobs(true);
//	    // 设置自行启动
//	    factory.setAutoStartup(true);
//	    // 延时启动，应用启动1秒后
//	    factory.setStartupDelay(1);
//	    factory.setQuartzProperties(quartzProperties());
//	    // 使用应用的dataSource替换quartz的dataSource
//	    factory.setDataSource(quartzDataSource);

	}

}
```

### 启动后任务开始正常运行

```shell
2020-12-09 21:04:02.775  INFO 8184 --- [           main] org.quartz.core.QuartzScheduler          : Scheduler quartzScheduler_$_NON_CLUSTERED started.
2020-12-344-09:04:02extends QuartzJobBean被调用的时候执行该该方法...具有参数：zheshichuanshujinqude
也可以通过context获取到传入的参数zheshichuanshujinqude
2020-12-09 21:04:02.794  INFO 8184 --- [           main] o.g.s.SpringScheduleApplication          : Started SpringScheduleApplication in 1.477 seconds (JVM running for 2.118)
2020-12-344-09:04:03extends QuartzJobBean被调用的时候执行该该方法...具有参数：zheshichuanshujinqude
也可以通过context获取到传入的参数zheshichuanshujinqude
2020-12-344-09:04:03容器管理的Job被调用的时候执行该该方法...
2020-12-344-09:04:04extends QuartzJobBean被调用的时候执行该该方法...具有参数：zheshichuanshujinqude
也可以通过context获取到传入的参数zheshichuanshujinqude
2020-12-344-09:04:05extends QuartzJobBean被调用的时候执行该该方法...具有参数：zheshichuanshujinqude
也可以通过context获取到传入的参数zheshichuanshujinqude
2020-12-344-09:04:05容器管理的Job被调用的时候执行该该方法...
```

## 自己配置`SchedulerFactoryBean`

```java
@Configuration
public class SchedulerConfig {
    // 配置文件路径
    @Value("${quartzConfig.location}")
    private String quartzConfig;

    // 自行配置数据源,这里不进行配置
    @Qualifier("quartzDataSource")
    @Autowired
    private DataSource quartzDataSource;

    /**
     * 从quartz.properties文件中读取Quartz配置属性
     * @return
     * @throws IOException
     */
    @Bean
    public Properties quartzProperties() throws IOException {
        PropertiesFactoryBean propertiesFactoryBean = new PropertiesFactoryBean();
        // new ClassPathResource(QUARTZ_CONFIG)
        propertiesFactoryBean.setLocation(new FileSystemResource(quartzConfig));
        propertiesFactoryBean.afterPropertiesSet();
        return propertiesFactoryBean.getObject();
    }


    @Bean
    public SchedulerFactoryBean schedulerFactoryBean(ApplicationContext applicationContext) throws IOException {
        SchedulerFactoryBean factory = new SchedulerFactoryBean();
        AutoWiredSpringBeanToJobFactory jobFactory = new AutoWiredSpringBeanToJobFactory();
        jobFactory.setApplicationContext(applicationContext);
        factory.setJobFactory(jobFactory);
        factory.setOverwriteExistingJobs(true);
        // 设置自行启动
        factory.setAutoStartup(true);
        // 延时启动，应用启动1秒后
        factory.setStartupDelay(1);
        factory.setQuartzProperties(quartzProperties());
        // 使用应用的dataSource替换quartz的dataSource
        factory.setDataSource(quartzDataSource);
        return factory;
    }

    @Bean
    public Scheduler scheduler(SchedulerFactoryBean schedulerFactoryBean)
            throws IOException, SchedulerException {
        Scheduler scheduler = schedulerFactoryBean.getScheduler();
        scheduler.start();
        return scheduler;
    }

}
```

### 编写Quartz配置文件 并在配置文件编写属性 `quartzConfig.location`

```
#============================================================================
# Configure Main Scheduler Properties
#============================================================================

org.quartz.scheduler.instanceName: TestScheduler
org.quartz.scheduler.instanceId: AUTO
org.quartz.scheduler.skipUpdateCheck: true

#============================================================================
# Configure ThreadPool
#============================================================================

org.quartz.threadPool.class: org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount: 5
org.quartz.threadPool.threadPriority: 5

#============================================================================
# Configure JobStore
#============================================================================

#容许的最大作业延
org.quartz.jobStore.misfireThreshold: 60000

org.quartz.jobStore.class=org.quartz.impl.jdbcjobstore.JobStoreTX
org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.StdJDBCDelegate
org.quartz.jobStore.useProperties=false
org.quartz.jobStore.dataSource=springTxDataSource.schedulerFactoryBean
org.quartz.jobStore.tablePrefix=QRTZ_
org.quartz.jobStore.isClustered=true

#调度实例失效的检查时间间隔
org.quartz.jobStore.clusterCheckinInterval=20000
```

## 根据请求开启任务

```java
@RequestMapping("/aaaa")
public SchedulerMetaData addJob(@PathVariable("id") String id) throws SchedulerException {

        JobKey jobKey = JobKey.jobKey(id, GROUP);
        JobDetail job = JobBuilder.newJob(HiJob.class)
                .withIdentity(jobKey)
                .usingJobData(new JobDataMap(new HashMap<>()))  //添加
                .build();

			
        Trigger trigger = TriggerBuilder.newTrigger()
                .withIdentity(TriggerKey.triggerKey(id, GROUP))
                .withSchedule(SimpleScheduleBuilder.simpleSchedule().withIntervalInMilliseconds(5000).repeatForever())
                .startAt(new Date(System.currentTimeMillis() + 5000))
                .build();


    //开启任务
        scheduler.scheduleJob(job, trigger);
        return scheduler.getMetaData();
    }
```

