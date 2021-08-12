# SpringBootWebSocket

## maven依赖

SpringBoot2.0对WebSocket的支持，直接就有包可以引入

```xml
		<dependency>  
           <groupId>org.springframework.boot</groupId>  
           <artifactId>spring-boot-starter-websocket</artifactId>  
       </dependency> 
```

## WebSocketConfig

启用WebSocket的支持也是很简单，几句代码搞定

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.server.standard.ServerEndpointExporter;

/**
 * 开启WebSocket支持
 * @author zhengkai.blog.csdn.net
 */
@Configuration  
public class WebSocketConfig {  
	
    @Bean  
    public ServerEndpointExporter serverEndpointExporter() {  
        return new ServerEndpointExporter();  
    }  
  
} 
```

WebSocket组件

```java
@Component
@ServerEndpoint("/websocket/{sid}")
public class WebSocketServer {

    //静态变量，用来记录当前在线连接数。应该把它设计成线程安全的。
    private static AtomicInteger onlineNum = new AtomicInteger();

    //concurrent包的线程安全Set，用来存放每个客户端对应的WebSocketServer对象。
    private static ConcurrentHashMap<String, Session> sessionPools = new ConcurrentHashMap<>();

    //发送消息
    public void sendMessage(Session session, String message) throws IOException {
        if(session != null){
            synchronized (session) {
//                System.out.println("发送数据：" + message);
                session.getBasicRemote().sendText(message);
            }
        }
    }
    //给指定用户发送信息
    public void sendInfo(String userName, String message){
        Session session = sessionPools.get(userName);
        try {
            sendMessage(session, message);
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    //建立连接成功调用
    @OnOpen
    public void onOpen(Session session, @PathParam(value = "sid") String userName){
        sessionPools.put(userName, session);
        addOnlineCount();
        System.out.println(userName + "加入webSocket！当前人数为" + onlineNum);
        try {
            sendMessage(session, "欢迎" + userName + "加入连接！");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //关闭连接时调用
    @OnClose
    public void onClose(@PathParam(value = "sid") String userName){
        sessionPools.remove(userName);
        subOnlineCount();
        System.out.println(userName + "断开webSocket连接！当前人数为" + onlineNum);
    }

    //收到客户端信息
    @OnMessage
    public void onMessage(String message) throws IOException {
        if(message.startsWith("to")){
            String[] strs = message.split(":");
            sendInfo(strs[1],strs[2]);
            return;
        }
        message = "客户端：" + message + ",已收到";
        System.out.println(message);

        for (Session session: sessionPools.values()) {
            try {
                sendMessage(session, message);
            } catch(Exception e){
                e.printStackTrace();
                continue;
            }
        }
    }

    //错误时调用
    @OnError
    public void onError(Session session, Throwable throwable){
        System.out.println("发生错误");
        throwable.printStackTrace();
    }

    public static void addOnlineCount(){
        onlineNum.incrementAndGet();
    }

    public static void subOnlineCount() {
        onlineNum.decrementAndGet();
    }
}

```

表示监听 `/imserver/{userId}` websocket URL 

使用 线程安全类保证并发

```java
//静态变量，用来记录当前在线连接数。应该把它设计成线程安全的。
    private static AtomicInteger onlineNum = new AtomicInteger();

//concurrent包的线程安全Set，用来存放每个客户端对应的WebSocketServer对象。
    private static ConcurrentHashMap<String, Session> sessionPools = new ConcurrentHashMap<>();
```

onMessage 方法中实现了 点对点和群发

```java
//收到客户端信息
    @OnMessage
    public void onMessage(String message) throws IOException {
        if(message.startsWith("to")){
            String[] strs = message.split(":");
            sendInfo(strs[1],strs[2]);
            return;
        }
        message = "客户端：" + message + ",已收到";
        System.out.println(message);

        for (Session session: sessionPools.values()) {
            try {
                sendMessage(session, message);
            } catch(Exception e){
                e.printStackTrace();
                continue;
            }
        }
    }
```

`Session `存放着会话信息,并存放在`ConcurrentHashMap `中 使用用户名作为`key`

## 编写前端页面

```html
<!DOCTYPE html>
<html lang="ch">
<head>
    <meta charset="UTF-8">
    <title>websocket</title>
</head>
<body>
<div id="content">
</div>

<div>
    <input type="text" id="userid" placeholder="用户id">
    <button type="button" id="connect">连接</button>
    <br>
    <input type="text" id="input" placeholder="输入内容">
    <button type="button" id="sub">提交全部</button>
    <br>
    <input type="text" id="to" placeholder="发送到的位置">
    <button type="button" id="subuser">提交</button>
</div>

<script type="text/javascript">
    let input = document.querySelector('#input');
    let to = document.querySelector('#to');
    let content = document.querySelector('#content');
    let sub = document.querySelector('#sub');
    let subuser = document.querySelector('#subuser');
    let conn = document.querySelector('#connect');
    var socket = null;

    function connect() {
        if (typeof (WebSocket) == "undefined") {
            console.log("您的浏览器不支持WebSocket");
        } else {

            var userId = document.getElementById('userid').value;
            var socketUrl = "ws://localhost:8080//websocket/" + userId;
            console.log(socketUrl);
            if (socket != null) {
                socket.close();
                socket = null;
            }
            socket = new WebSocket(socketUrl);

            socket.onopen = function () {
                passHtml('连接成功 用户名' + userId);
            }
            socket.onmessage = function (message) {
                passHtml(message.data);
            }

            socket.onclose = function () {
                passHtml('连接关闭');
            }

            socket.onerror = function () {
                passHtml('发生了错误');
            }


        }
    }

    function sendAll() {
        socket.send("all:" + input.value);
    }

    function sendUser() {
        socket.send("to:" + to.value + ":" + input.value);
    }

    function passHtml(message) {
        content.append('<div>' + message + '</div>');
    }

    console.log("11111111111111111111111111111111111");
    conn.addEventListener('click', function () {
        connect();
    })
    sub.addEventListener('click', function () {
        sendAll();
    });
    subuser.addEventListener('click', function () {
        sendUser();
    });


</script>
</body>
</html>
```

启动访问 `websocket` 页面