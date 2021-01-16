# Quartz

Quartz是OpenSymphony开源组织在Job scheduling领域又一个开源项目，完全由Java开发，可以用来执行定时任务，类似于java.util.Timer。但是相较于Timer， Quartz增加了很多功能：

-   持久性作业 - 就是保持调度定时的状态;
-   作业管理 - 对调度作业进行有效的管理;

`任务 Task或者Job` 负责执行相关任务

`触发器Trigger `负责指定相关Job的执行时间，执行间隔，运行次数

`调度器 Schedule` 负责将将指定的触发器应用到指定的任务中，包含  `SimpleTrigger和CronTrigger`



## 搭建

### 创建Maven工程

```xml
<!-- https://mvnrepository.com/artifact/org.quartz-scheduler/quartz -->
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.3.0</version>
</dependency>
```

### 创建一个Job

用于打印当前时间信息

```java
public class TestJob implements org.quartz.Job {

	@Override
	public void execute(JobExecutionContext context) throws JobExecutionException {
		
		String printTime = new SimpleDateFormat("yy-MM-dd HH-mm-ss").format(new Date());
        System.out.println("Job 在" + printTime + "执行");

	}

}
```

### 创建Schedule

```java
// 1、创建调度器Scheduler
	        SchedulerFactory schedulerFactory = new StdSchedulerFactory();
	        Scheduler scheduler = schedulerFactory.getScheduler();
	        // 2、创建JobDetail实例，并与TestJob类绑定(Job执行内容)
	        JobDetail jobDetail = JobBuilder.newJob(TestJob.class)
	                                        .withIdentity("Job名", "Job组1").build();
	        // 3、构建Trigger实例,每隔1s执行一次
	        Trigger trigger = TriggerBuilder.newTrigger().withIdentity("trigger名", "trigger组1")
	                .startNow()//立即生效 withSchedule 方法来设置 SimpleScheduleBuilder 构建规则
	                .withSchedule(SimpleScheduleBuilder.simpleSchedule()
	                .withIntervalInSeconds(1)//每隔1s执行一次
	                .repeatForever()).build();//一直执行

	        //4、执行
	        scheduler.scheduleJob(jobDetail, trigger);
	        System.out.println("--------scheduler start ! ------------");
	        scheduler.start();

	        //睡眠
	        TimeUnit.MINUTES.sleep(1);
	        scheduler.shutdown();
	        System.out.println("--------scheduler shutdown ! ------------");
```

### 执行

```
--------scheduler start ! ------------
Job 在20-12-08 18-33-48执行
Job 在20-12-08 18-33-49执行
Job 在20-12-08 18-33-50执行
Job 在20-12-08 18-33-51执行
Job 在20-12-08 18-33-52执行
```

## Quartz 中出现的类

-   Job和JobDetail
-   JobExecutionContext
-   JobDataMap
-   Trigger、SimpleTrigger、CronTrigger

### Job和JobDetail

Job是Quartz中的一个接口，接口下只有execute方法，在这个方法中编写业务逻辑。
接口中的源码：
![这里写图片描述](https://raw.githubusercontent.com/1471246901/myblog/master/img/20180710135513678)

JobDetail用来绑定Job，为Job实例提供许多属性：

-   name
-   group
-   jobClass
-   jobDataMap

JobDetail绑定指定的Job，每次Scheduler调度执行一个Job的时候，首先会拿到对应的Job，然后创建该Job实例，再去执行Job中的execute()的内容，任务执行结束后，关联的Job对象实例会被释放，且会被JVM GC清除。

为什么设计成JobDetail + Job，不直接使用Job

>   JobDetail定义的是任务数据，而真正的执行逻辑是在Job中。
>   这是因为任务是有可能并发执行，如果Scheduler直接使用Job，就会存在对同一个Job实例并发访问的问题。而JobDetail & Job 方式，Sheduler每次执行，都会根据JobDetail创建一个新的Job实例，这样就可以规避并发访问的问题。

### JobExecutionContext

JobExecutionContext中包含了Quartz运行时的环境以及Job本身的详细数据信息。
当Schedule调度执行一个Job的时候，就会将JobExecutionContext传递给该Job的execute()中，Job就可以通过JobExecutionContext对象获取信息。
主要信息有：
![这里写图片描述](https://raw.githubusercontent.com/1471246901/myblog/master/img/20180710135537517)

### JobExecutionContext 中存储数据

JobDataMap实现了JDK的Map接口，可以以Key-Value的形式存储数据。
JobDetail、Trigger都可以使用JobDataMap来设置一些参数或信息，
Job执行execute()方法的时候，JobExecutionContext可以获取到JobExecutionContext中的信息：
如：

```java
JobDetail jobDetail = JobBuilder.newJob(PrintWordsJob.class)                        .usingJobData("jobDetail1", "这个Job用来测试的")
                  .withIdentity("job1", "group1").build();

 Trigger trigger = TriggerBuilder.newTrigger().withIdentity("trigger1", "triggerGroup1")
      .usingJobData("trigger1", "这是jobDetail1的trigger")
      .startNow()//立即生效
      .withSchedule(SimpleScheduleBuilder.simpleSchedule()
      .withIntervalInSeconds(1)//每隔1s执行一次
      .repeatForever()).build();//一直执行
12345678910
```

Job执行的时候，可以获取到这些参数信息：

```java
    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {

        System.out.println(jobExecutionContext.getJobDetail().getJobDataMap().get("jobDetail1"));
        System.out.println(jobExecutionContext.getTrigger().getJobDataMap().get("trigger1"));
        String printTime = new SimpleDateFormat("yy-MM-dd HH-mm-ss").format(new Date());
        System.out.println("PrintWordsJob start at:" + printTime + ", prints: Hello Job-" + new Random().nextInt(100));


    }12345678910
```

### Trigger、SimpleTrigger、CronTrigger

#### Trigger

Trigger是Quartz的触发器，会去通知Scheduler何时去执行对应Job。

```
new Trigger().startAt():表示触发器首次被触发的时间;
new Trigger().endAt():表示触发器结束触发的时间;12
```

#### SimpleTrigger

SimpleTrigger可以实现在一个指定时间段内执行一次作业任务或一个时间段内多次执行作业任务。
下面的程序就实现了程序运行5s后开始执行Job，执行Job 5s后结束执行：

```
Date startDate = new Date();
startDate.setTime(startDate.getTime() + 5000);

 Date endDate = new Date();
 endDate.setTime(startDate.getTime() + 5000);

        Trigger trigger = TriggerBuilder.newTrigger().withIdentity("trigger1", "triggerGroup1")
                .usingJobData("trigger1", "这是jobDetail1的trigger")
                .startNow()//立即生效
                .startAt(startDate)
                .endAt(endDate)
                .withSchedule(SimpleScheduleBuilder.simpleSchedule()
                .withIntervalInSeconds(1)//每隔1s执行一次
                .repeatForever()).build();//一直执行
123456789101112131415
```

#### CronTrigger

CronTrigger功能非常强大，是基于日历的作业调度，而SimpleTrigger是精准指定间隔，所以相比SimpleTrigger，CroTrigger更加常用。CroTrigger是基于Cron表达式的，先了解下Cron表达式：

cron表达式语法

```
[秒] [分] [小时] [日] [月] [周] [年]
```

>   注：[年]不是必须的域，可以省略[年]，则一共6个域

| 序号 | 说明 | 必填 | 允许填写的值   | 允许的通配符  |
| :--- | :--- | :--- | :------------- | :------------ |
| 1    | 秒   | 是   | 0-59           | , - * /       |
| 2    | 分   | 是   | 0-59           | , - * /       |
| 3    | 时   | 是   | 0-23           | , - * /       |
| 4    | 日   | 是   | 1-31           | , - * ? / L W |
| 5    | 月   | 是   | 1-12 / JAN-DEC | , - * /       |
| 6    | 周   | 是   | 1-7 or SUN-SAT | , - * ? / L # |
| 7    | 年   | 否   | 1970-2099      | , - * /       |

通配符说明:

-   `*` 表示所有值。 例如:在分的字段上设置 *,表示每一分钟都会触发。
-   `?` 表示不指定值。使用的场景为不需要关心当前设置这个字段的值。例如:要在每月的10号触发一个操作，但不关心是周几，所以需要周位置的那个字段设置为”?” 具体设置为 0 0 0 10 * ?
-   `-` 表示区间。例如 在小时上设置 “10-12”,表示 10,11,12点都会触发。
-   `,` 表示指定多个值，例如在周字段上设置 “MON,WED,FRI” 表示周一，周三和周五触发
-   `/` 用于递增触发。如在秒上面设置”5/15” 表示从5秒开始，每增15秒触发(5,20,35,50)。 在月字段上设置’1/3’所示每月1号开始，每隔三天触发一次。
-   `L` 表示最后的意思。在日字段设置上，表示当月的最后一天(依据当前月份，如果是二月还会依据是否是润年[leap]), 在周字段上表示星期六，相当于”7”或”SAT”。如果在”L”前加上数字，则表示该数据的最后一个。例如在周字段上设置”6L”这样的格式,则表示“本月最后一个星期五”
-   `W` 表示离指定日期的最近那个工作日(周一至周五). 例如在日字段上置”15W”，表示离每月15号最近的那个工作日触发。如果15号正好是周六，则找最近的周五(14号)触发, 如果15号是周未，则找最近的下周一(16号)触发.如果15号正好在工作日(周一至周五)，则就在该天触发。如果指定格式为 “1W”,它则表示每月1号往后最近的工作日触发。如果1号正是周六，则将在3号下周一触发。(注，”W”前只能设置具体的数字,不允许区间”-“)。
-   `#` 序号(表示每月的第几个周几)，例如在周字段上设置”6#3”表示在每月的第三个周六.注意如果指定”#5”,正好第五周没有周六，则不会触发该配置(用在母亲节和父亲节再合适不过了) ；小提示：’L’和 ‘W’可以一组合使用。如果在日字段上设置”LW”,则表示在本月的最后一个工作日触发；周字段的设置，若使用英文字母是不区分大小写的，即MON与mon相同。

示例

每隔5秒执行一次：*/5 * * * * ?

每隔1分钟执行一次：0 */1 * * * ?

每天23点执行一次：0 0 23 * * ?

每天凌晨1点执行一次：0 0 1 * * ?

每月1号凌晨1点执行一次：0 0 1 1 * ?

每月最后一天23点执行一次：0 0 23 L * ?

每周星期天凌晨1点实行一次：0 0 1 ? * L

在26分、29分、33分执行一次：0 26,29,33 * * * ?

每天的0点、13点、18点、21点都执行一次：0 0 0,13,18,21 * * ?

>   [是Guava不是瓜娃](https://blog.csdn.net/noaman_wgs)