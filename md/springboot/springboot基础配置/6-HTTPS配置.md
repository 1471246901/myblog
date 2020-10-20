# HTTPS配置(tomcat)

首选要取得https证书,国内的一些云服务区厂商会提供免费的https证书.

在jdk中有一个java数字证书管理工具`kettool` 能生成数字证书,路径在`jdk/bin`下

![image-20200727162449675](6-HTTPS%E9%85%8D%E7%BD%AE.assets/image-20200727162449675.png)

口令为

```shell
keytool -genkey -alias tomcathttps -keyalg RSA -keysize 2048 -keystore D:\test\gu.p12 -validity 365
```

其中

| 参数      | 含义            |
| --------- | :-------------- |
| -genkey   | 代表生成秘钥    |
| -alias    | keystore 的别名 |
| -keyalg   | 使用的加密算法  |
| -keysize  | 秘钥长度        |
| -keystore | 秘钥存放位置    |
| -validity | 秘钥有效时间    |

将生成的秘钥复制到项目根目录

![image-20200727164611682](6-HTTPS%E9%85%8D%E7%BD%AE.assets/image-20200727164611682.png)

然后在properties 做配置

```properties
# 秘钥文件名
server.ssl.key-store=gu.p12
# 秘钥别名
server.ssl.key-alias=tomcathttps
# 秘钥生成时输入的密码
server.ssl.key-store-password=123456
```

![image-20200727164702497](6-HTTPS%E9%85%8D%E7%BD%AE.assets/image-20200727164702497.png)

这个时候访问http:// 是不成功的 .这个时候我们可以配置请求重定向,让访问http请求重定向到https

添加配置类

```java
import org.apache.tomcat.util.descriptor.web.SecurityCollection;
import org.apache.tomcat.util.descriptor.web.SecurityConstraint;
import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.apache.catalina.connector.Connector;

@Configuration
public class TomcatConfig {

	@Bean
	public TomcatServletWebServerFactory tomcatServletWebServerFactory() {
		
		TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory() {

			@Override
			protected void postProcessContext(org.apache.catalina.Context context) {
				SecurityConstraint constraint = new SecurityConstraint();
				constraint.setUserConstraint("CONFIDENTIAL");
				SecurityCollection collection = new SecurityCollection();
				collection.addPattern("/*");
				constraint.addCollection(collection);
				context.addConstraint(constraint);
			};

		};
		factory.addAdditionalTomcatConnectors(createTomcatConnector());
		return factory;
	}

	private Connector createTomcatConnector() {
		Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
		connector.setScheme("http");
		connector.setPort(8080);
		connector.setSecure(false);
		connector.setRedirectPort(8081);
		return connector;

	}

}

```

其含义是增加一个Connector 去监听`http:// 8080` 端口,然后重定向到`https:// 8081` 端口

这时访问`http://name/hello`  就会跳转到   `https://name/hello` 上