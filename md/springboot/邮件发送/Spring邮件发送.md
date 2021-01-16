# Spring邮件发送

在注册账号、身份认证、重要通知，发送广告等都会用到邮件

SUN公司提供了JavaMail用来实现邮件发送，但是配置繁琐

## 准备 使用QQ邮箱发送



使用QQ邮箱发送邮件、首先开通邮箱的POP3/SMTP服务或者IMAP/SMTP服务

### POP3/SMTP 协议

>   **POP3  --- 接收**
>
>   POP3是Post Office Protocol 3的简称，bai即邮局协议的第3个版本,它规定怎样将个人计算机连接到Internet的邮件服务器和下载电子邮件的电子协议。
>
>   它是因特网电子邮件的第一个离线协议标准,POP3允许用户从服务器上把邮件存储到本地主机（即自己的计算机）上,同时删除保存在邮件服务器上的邮件，而POP3服务器则是遵循POP3协议的接收邮件服务器，用来接收电子邮件的。
>
>   **SMTP  --- 发送**
>
>   SMTP 的全称是“Simple Mail Transfer Protocol”，即简单邮件传输协议。它是一组用于从源地址到目的地址传输邮件的规范，通过它来控制邮件的中转方式。
>
>   SMTP 协议属于 TCP/IP 协议簇，它帮助每台计算机在发送或中转信件时找到下一个目的地。SMTP 服务器就是遵循 SMTP 协议的发送邮件服务器。 
>
>   ![STMP和POP3简介](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20201206193048486.png)

### QQ邮箱申请POP3/SMTP服务

登录到qq邮箱网页版，打开`设置`-`账户`

![image-20201206193637135](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20201206193637135.png)

开启POP3/SMTP服务，这个时候开启成功会得到一个授权码

![image-20201206193527814](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20201206193527814.png)

## SpringBoot环境下发送邮件

### 环境搭建

#### maven依赖

新建Springboot项目，添加maven依赖

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

#### application.properties配置邮箱基本信息

相关邮箱服务的地址可以在邮箱服务方查看

```properties
#邮件服务器的地址
spring.mail.host=smtp.qq.com
#邮件服务器的端口
spring.mail.port=465
#邮箱账号
spring.mail.username=1471246901@qq.com
# 授权码
spring.mail.password=hbwqaonhbaligghb
# 编码
spring.mail.default-encoding=UTF-8
# ssl连接配置
spring.mail.properties.smtp.socketFactory.class=javax.net.ssl.SSLSocketFactory
# 开启debug
spring.mail.properties.mail.debug=true
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
spring.mail.properties.mail.smtp.ssl.enable=true
```

### JavaMailSender 以及手动配置

`JavaMailSender` : SpringBoot 自动配置的邮件发送类

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ MimeMessage.class, MimeType.class, MailSender.class })
@ConditionalOnMissingBean(MailSender.class)
@Conditional(MailSenderCondition.class)
@EnableConfigurationProperties(MailProperties.class)
@Import({ MailSenderJndiConfiguration.class, MailSenderPropertiesConfiguration.class })
public class MailSenderAutoConfiguration {
```

JavaMailSender源自 `MailSenderPropertiesConfiguration.class` 

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnProperty(prefix = "spring.mail", name = "host")
class MailSenderPropertiesConfiguration {

	@Bean
	@ConditionalOnMissingBean(JavaMailSender.class)
	JavaMailSenderImpl mailSender(MailProperties properties) {
		JavaMailSenderImpl sender = new JavaMailSenderImpl();
		applyProperties(properties, sender);
		return sender;
	}

	private void applyProperties(MailProperties properties, JavaMailSenderImpl sender) {
		sender.setHost(properties.getHost());
		if (properties.getPort() != null) {
			sender.setPort(properties.getPort());
		}
		sender.setUsername(properties.getUsername());
		sender.setPassword(properties.getPassword());
		sender.setProtocol(properties.getProtocol());
		if (properties.getDefaultEncoding() != null) {
			sender.setDefaultEncoding(properties.getDefaultEncoding().name());
		}
		if (!properties.getProperties().isEmpty()) {
			sender.setJavaMailProperties(asProperties(properties.getProperties()));
		}
	}
```

**手动配置**

在面临SpringBoot无法自动注入这个类时可以使用手动配置

两个类

![image-20201206213515133](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20201206213515133.png)

模仿自动配置的两个类

MailConfig

```java
@Configuration
@EnableConfigurationProperties(MailProperties.class)
public class MailConfig {
	

	@Bean
	JavaMailSenderImpl mailSender(MailProperties properties) {
		System.out.println("自动注入JavaMailSender");
		JavaMailSenderImpl sender = new JavaMailSenderImpl();
		applyProperties(properties, sender);
		return sender;
	}

	private void applyProperties(MailProperties properties, JavaMailSenderImpl sender) {
		sender.setHost(properties.getHost());
		if (properties.getPort() != null) {
			sender.setPort(properties.getPort());
		}
		sender.setUsername(properties.getUsername());
		sender.setPassword(properties.getPassword());

		if (!properties.getProperties().isEmpty()) {
			sender.setJavaMailProperties(asProperties(properties.getProperties()));
		}
	}

	private Properties asProperties(Map<String, String> source) {
		Properties properties = new Properties();
		properties.putAll(source);
		return properties;
	}

}
```

MailProperties

```java
@ConfigurationProperties(prefix = "spring.mail")
public class MailProperties {

	private String Host;

	private Integer port;

	private String username;

	private String password;

	private Map<String, String> properties = new HashMap<>();

	//getter and setter...

}
```

### 简单发送

创建一个服务service来实现邮件发送

```java
@Service
public class MailService {
	
	@Autowired
	JavaMailSender javaMailSender;
	
	/**
	 * 
	 * @param from 发送者
	 * @param to 收件人
	 * @param cc 抄送人
	 * @param subject 主题
	 * @param content 内容
	 */
	public void sendSimpleMail(String from,String to,String cc,String subject,String content) {
		SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
		simpleMailMessage.setFrom(from);
		simpleMailMessage.setTo(to);
		simpleMailMessage.setCc(cc);
		simpleMailMessage.setSubject(subject);
		simpleMailMessage.setText(content);
		javaMailSender.send(simpleMailMessage);
	    
		System.out.println("mail Send ");
	}
	
	
}
```

**测试**

```java
@Autowired
	MailService mailService;
	
	
	@Test
	void testSendSimpleMail(){
		
		mailService.sendSimpleMail("1471246901@qq.com", "gushiyuwe@gmail.com", "SimpleMailTest", "这是一封自动发送的测试邮件");
	}
```

### 带附件的邮件

通过调用  `javaMailSender.createMimeMessage` 来创建 `MimeMessage`

**MimeMessage** 能够携带非文本格式的信息，并且收件方浏览器能够自动寻找应用程序打开文件

>   MIME(Multipurpose Internet Mail Extensions)多用途互联网邮件扩展类型。是设定某种[扩展名](https://baike.baidu.com/item/扩展名/103577)的[文件](https://baike.baidu.com/item/文件/6270998)用一种[应用程序](https://baike.baidu.com/item/应用程序/5985445)来打开的方式类型，当该扩展名文件被访问的时候，[浏览器](https://baike.baidu.com/item/浏览器/213911)会自动使用指定应用程序来打开。多用于指定一些客户端自定义的文件名，以及一些媒体文件打开方式。
>
>   它是一个互联网标准，扩展了电子邮件标准，使其能够支持：
>
>   非[ASCII](https://baike.baidu.com/item/ASCII/309296)字符文本；非文本格式附件（二进制、声音、图像等）；由多部分（multiple parts）组成的消息体；包含非ASCII字符的头信息（Header information）。

**编写发送带附件的邮件代码**

```java
/**
	 * 
	 * @param from 发送者
	 * @param to 收件人
	 * @param subject 主题
	 * @param content 内容
	 * @param files 附件
	 */
	public void sendAttachFileMail(String from,String to,String subject,String content,File... files ) {
		try {
			//Mime代表邮件类型是多用途互联网邮件扩展协议的邮件
			MimeMessage message = javaMailSender.createMimeMessage();
			// 使用MimeMessageHelper简化MimeMessage邮件配置 第二个参数 boolean multipart 代表是否是multipart 类型邮件
			MimeMessageHelper helper = new MimeMessageHelper(message,true);
			helper.setFrom(from);
			helper.setTo(to);
			helper.setSubject(subject);
			helper.setText(content);
			for (int i = 0; i < files.length; i++) {
				helper.addAttachment(files[i].getName(),files[i]);
			}
			javaMailSender.send(message);
			
		} catch (MessagingException e) {
			e.printStackTrace();
			System.out.println("SimpleMail Send filed");
		}
		System.out.println("AttachFileMail Send ");
	}
```

**测试**

```java
@Test
	void testSendAttachFileMail() throws FileNotFoundException {

        //在应用程序中推荐使用相对位置编码
        //通过 mailService.getClass().getResource("/") 获取类路径
		File ss = new File(
				"C:\\Users\\14712\\eclipse-workspace\\SpringMailDemo\\src\\main\\resources\\static\\test.txt");

		mailService.sendAttachFileMail("1471246901@qq.com", "gushiyuwe@gmail.com", "AttachFileMailTest", "这是一封带有附件的邮件",
				ss);
	}
```

### 带有图片的邮件

带有图片的邮件可以通过两个方式来实现，一是发送一段HTML代码，用户收到HTML后会从服务器请求从而获得图片显示在邮件上，二是通过静态资源来显示图片，此时图片通过邮件传输，再次演示第二种

编写代码

```java
/**
	 * 
	 * @param from 发送者
	 * @param to 收件人
	 * @param subject 主题
	 * @param content 内容
	 * @param files 图片
	 */
	public void sendImgMail(String from,String to,String subject,String content,File... files ) {
		try {
			//Mime代表邮件类型是多用途互联网邮件扩展协议的邮件
			MimeMessage message = javaMailSender.createMimeMessage();
			// 使用MimeMessageHelper简化MimeMessage邮件配置 第二个参数 boolean multipart 代表是否是multipart 类型邮件
			MimeMessageHelper helper = new MimeMessageHelper(message,true,"UTF-8");
			helper.setFrom(from);
			helper.setTo(to);
			helper.setSubject(subject);
			for (int i = 0; i < files.length; i++) {
				//也可以通过FileSystemResource或者ClassPathResource 加载资源
				helper.addInline(files[i].getName(),files[i]);
				content+="</br>这是图片"+(i+1)+"<img src='cid:"+files[i].getName()+"' /></br>";
			}
			//boolean html 第二个参数代表是否是以 HTML解析
			helper.setText(content,true);
			javaMailSender.send(message);
			
		} catch (MessagingException e) {
			e.printStackTrace();
			System.out.println("ImgMail Send filed");
		}
		System.out.println("ImgMail Send ");
		
		
	}
```

使用`addInline` 方法添加图片资源 使用 `cid:` 引用添加的资源

在 `helper.setText(content,true);` 是设置了以html方式解析

有些时候在 `application.properties`中配置 编码并不起作用，这时在实例化 `MimeMessageHelper` 指定编码

`MimeMessageHelper helper = new MimeMessageHelper(message,true,"UTF-8");`

**测试**

```java
@Test
	void testSendImgMail() {
		String to = "gushiyuwe@foxmail.com";
		File file1 = new File("C:\\Users\\14712\\eclipse-workspace\\SpringMailDemo\\src\\main\\resources\\static\\11.jpg");
		File file2 = new File("C:\\Users\\14712\\eclipse-workspace\\SpringMailDemo\\src\\main\\resources\\static\\12.jpg");
		File file3 = new File("C:\\Users\\14712\\eclipse-workspace\\SpringMailDemo\\src\\main\\resources\\static\\13.jpg");
		mailService.sendImgMail("1471246901@qq.com", "gushiyuwe@foxmail.com", "ImgMailTest", "这是一封带有图片的邮件</br>",file1,file3,file2);
	}
```

测试发现最后一张图片显示不出来，在网页版Gmail中有时显示不出来。*可能不是标准HTML导致的*

### 使用Thymeleaf模板构建邮件

使用Thymeleaf模板需要先添加依赖

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

thymeleaf 模板默认路径为 `resources/templates` 再此编写 mail.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta charset="UTF-8">
<title>This is HTML File</title>
</head>
<body>
<p>您正在注册xxxx账号，信息为<span th:text="${username}"></span>如核实无误，请<a th:href="${activatelink}">点击本链接激活你的账号。</a></p>
</body>
</html>
```

编写代码

```java
/**
	 * 
	 * @param from 发送者
	 * @param to 收件人
	 * @param subject 主题
	 * @param map 用于渲染模板的参数
	 * @param files 附件
	 */
	public void sendThymeleafHTMLMail(String from,String to,String subject,Map<String, String> map,File... files ) {
		Iterator<String> it = map.keySet().iterator();
		Context context = new Context();
		it.forEachRemaining((String k)->{
			context.setVariable(k, map.get(k));
		});
		String Mail = templateEngine.process("Mail.html", context);
		
		try {
			//Mime代表邮件类型是多用途互联网邮件扩展协议的邮件
			MimeMessage message = javaMailSender.createMimeMessage();
			// 使用MimeMessageHelper简化MimeMessage邮件配置 第二个参数 boolean multipart 代表是否是multipart 类型邮件
			MimeMessageHelper helper = new MimeMessageHelper(message,true,"UTF-8");
			helper.setFrom(from);
			helper.setTo(to);
			helper.setSubject(subject);
			
			//boolean html 第二个参数代表是否是以 HTML解析
			helper.setText(Mail,true);
			for (int i = 0; i < files.length; i++) {
				helper.addAttachment(files[i].getName(), files[i]);
			}
			javaMailSender.send(message);
			
		} catch (MessagingException e) {
			e.printStackTrace();
			System.out.println("HTMLMail Send filed");
		}
		System.out.println("HTMLMail Send ");
		
	}
```

测试

```java
@Test
	void testSendThymeleafMail() {
		File file1 = new File("C:\\Users\\14712\\eclipse-workspace\\SpringMailDemo\\src\\main\\resources\\static\\11.jpg");
		File file2 = new File("C:\\Users\\14712\\eclipse-workspace\\SpringMailDemo\\src\\main\\resources\\static\\12.jpg");
		File file3 = new File("C:\\Users\\14712\\eclipse-workspace\\SpringMailDemo\\src\\main\\resources\\static\\13.jpg");
		Map<String, String> map = new HashMap<>();
		map.put("username", "Gu");
		map.put("activatelink", "https://www.baidu.com");
		mailService.sendThymeleafHTMLMail("1471246901@qq.com", "gushiyuwe@foxmail.com", "ImgMailTest", map ,file1,file3,file2);
	}
```

![image-20201207212739990](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20201207212739990.png)

