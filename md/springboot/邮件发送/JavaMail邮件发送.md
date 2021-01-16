# JavaMail邮件发送

Java 官方提供了对电子邮件协议封装的 Java 类库，就是JavaMail，但并没有包含到标准的 JDK 中，需要我们自己去 [Java](https://java.net/projects/javamail/pages/Home ) 或 [Oracle](http://www.oracle.com/technetwork/java/javamail/index.html) 官网下载。

或者使用Maven进行下载

```xml
<dependency>
			<groupId>javamail</groupId>
			<artifactId>javamail</artifactId>
			<version>1.3.3</version>
</dependency>
```

编写代码

```java

public class Mail {

	private String host = ""; // smtp服务器
	private Integer port;   //端口
	private String from = ""; // 发件人地址
	private String to = ""; // 收件人地址
	private Map<String, String> affixfileMap = new HashMap<>();
	private String user = ""; // 用户名
	private String pwd = ""; // 密码
	private String subject = ""; // 邮件标题
	private Properties props = new Properties();
	private String content = "";

	public Mail(String host,Integer port, String from, String to, String user, String pwd, String subject) {
		super();
		this.port= port;
		this.host = host;
		this.from = from;
		this.to = to;
		this.user = user;
		this.pwd = pwd;
		this.subject = subject;
	}

	public void setAddress(String from, String to, String subject) {
		this.from = from;
		this.to = to;
		this.subject = subject;
	}

	public void setAffix(String affixName, String affix) {
		affixfileMap.put(affixName, affix);
	}

	public void setContent(String content) {
		this.content = content;
	}

	public void setProps(Object k, Object v) {
		props.put(k, v);
	}

	public void send() {
		//构建session
		Session session = Session.getDefaultInstance(props);

		//设置为调试模式
		session.setDebug(true);

		// 用session为参数定义消息对象
		MimeMessage message = new MimeMessage(session);

		try {
			// 发件人地址
			message.setFrom(new InternetAddress(from));
			// 收件人地址
			message.addRecipient(Message.RecipientType.TO, new InternetAddress(to));
			// 标题
			message.setSubject(subject);

			// 向multipart对象中添加邮件的各个部分内容，包括文本内容和附件
			Multipart multipart = new MimeMultipart();

			// 设置邮件的文本内容
			BodyPart contentPart = new MimeBodyPart();
			contentPart.setText(content);
			multipart.addBodyPart(contentPart);

			this.affixfileMap.forEach((name, fileurl) -> {
				// 添加附件
				BodyPart messageBodyPart = new MimeBodyPart();
				DataSource source = new FileDataSource(fileurl);
				// 添加附件的内容
				try {
					messageBodyPart.setDataHandler(new DataHandler(source));
					// 添加附件的标题
					//Encoder enc = Base64.getEncoder();
					messageBodyPart.setFileName(name);
					multipart.addBodyPart(messageBodyPart);

				} catch (Exception e) {
					throw new RuntimeException("添加附件失败");
				}
			});

			// 将multipart对象放到message中
			message.setContent(multipart);
			// 保存邮件
			message.saveChanges();
			// 发送邮件
			Transport transport = session.getTransport("smtp");
			// 连接服务器的邮箱
			transport.connect(host, port, user, pwd);
			// 把邮件发送出去
			transport.sendMessage(message, message.getAllRecipients());
			transport.close();
		} catch (Exception e) {
			RuntimeException a = new RuntimeException("邮件发送失败");
			a.addSuppressed(e);
			throw a;
		}
	}

	public static void main(String[] args) {
		Mail aMail = new Mail("smtp.qq.com",465, "1471246901@qq.com", "gushiyuwe@foxmail.com", "1471246901@qq.com", "hbwqaonhbaligghb", "javamail测试");
		aMail.setContent("这是消息体");
		aMail.setProps("mail.smtp.socketFactory.class", "javax.net.ssl.SSLSocketFactory");
		aMail.setProps("mail.smtp.socketFactory.fallback", "false");
		aMail.setProps("mail.transport.protocol", "smtp");   // 使用的协议（JavaMail规范要求）
		aMail.setProps("mail.smtp.auth", "true");            // 需要请求认证
		aMail.setAffix("11.jpg", "C:\\Users\\14712\\eclipse-workspace\\SpringMailDemo\\src\\main\\resources\\static\\11.jpg");
		aMail.setAffix("12.jpg", "C:\\Users\\14712\\eclipse-workspace\\SpringMailDemo\\src\\main\\resources\\static\\12.jpg");
		aMail.setAffix("13.jpg", "C:\\Users\\14712\\eclipse-workspace\\SpringMailDemo\\src\\main\\resources\\static\\13.jpg");
		aMail.send();
	}
}

```

javamail相关类解释

Session： 表示一个会话，邮件传送依靠session传输，相关参数由session维护

MimeMessage： 表示一封Mime形式的邮件，包含 发送者，接收者，标题，内容，附件等

BodyPart： 表示邮件正文的一个项目，可以是正文，可以是附件等

Transport： 表示一次发送，由Session产生

