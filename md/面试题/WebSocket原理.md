# WebSocket原理

####  背景

WebSocket 是基于Http 协议的改进，Http 为无状态协议，基于短连接，需要频繁的发起请求，第二 Http 只能客户端发起请求，服务端无法主动请求。

#### 相同点

都是基于TCP的应用层协议。
都使用Request/Response模型进行连接的建立。
在连接的建立过程中对错误的处理方式相同，在这个阶段WS可能返回和HTTP相同的返回码。
都可以在网络中传输数据。

#### 不同点

WS使用HTTP来建立连接，但是定义了一系列新的header域，这些域在HTTP中并不会使用。
WS的连接不能通过中间人来转发，它必须是一个直接连接。
WS连接建立之后，通信双方都可以在任何时刻向另一方发送数据。
WS连接建立之后，数据的传输使用帧来传递，不再需要Request消息。
WS的数据帧有序。

WebSocket 分为握手和数据传输

#### 握手

WebSocket 的握手基于http GET方法进行，

-   必须是有效的http request 格式
    HTTP request method 必须是GET，协议应不小于1.1 如： Get /chat HTTP/1.1

-   必须包括Upgrade 头域，并且其值为“websocket”，表明http 协议升级为WebSocket.

-   必须包括"Connection" 头域，并且其值为 "Upgrade"

-   必须包括"Sec-WebSocket-Key"头域，其值采用base64编码的随机16字节长的字符序列， 服务器端根据该域来判断client 确实是websocket请求而不是冒充的，如http。响应方式是，首先要获取到请求头中的Sec-WebSocket-Key的值，再把这一段GUID "258EAFA5-E914-47DA-95CA-C5AB0DC85B11"加到获取到的Sec-WebSocket-Key的值的后面，然后拿这个字符串做SHA-1 hash计算，然后再把得到的结果通过base64加密，就得到了返回给客户端的Sec-WebSocket-Accept的http响应头的值。

-   必须包括"Sec-webSocket-Version" 头域，当前值必须是13.
    可能包括"Sec-WebSocket-Protocol"，表示client（应用程序）支持的协议列表，server选择一个或者没有可接受的协议响应之。
    可能包括"Sec-WebSocket-Extensions"， 协议扩展， 某类协议可能支持多个扩展，通过它可以实现协议增强

-   可能包括任意其他域，如cookie

如客户端握手

```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Origin: http://example.com
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```

服务端的握手

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
```



>   [WebSocket SSL 加密浅析 - stalla - 博客园 (cnblogs.com)](https://www.cnblogs.com/stalla/articles/9993576.html)