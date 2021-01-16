# TCP  KEEPALIVE

TCP是面向连接的，一般情况，两端的应用程序可以通过发送和接收数据得知对端的存活。
当两端的应用程序都没有数据发送和接收时，如何判断连接是否正常呢？

这就是`SO_KEEPALIVE`的作用。

# 1. SO_KEEPALIVE 的作用

## 1.1 SO_KEEPALIVE的定义

`SO_KEEPALIVE`用于开启或者关闭保活探测，默认情况下是关闭的。

当`SO_KEEPALIVE`开启时，可以保持连接检测对方主机是否崩溃，避免（服务器）永远阻塞于TCP连接的输入。

相关的属性包括：
`tcp_keepalive_time`、`tcp_keepalive_probes`、`tcp_keepalive_intvl`。

```
tcp_keepalive_intvl (integer; default: 75; since Linux 2.4)
      The number of seconds between TCP keep-alive probes.

tcp_keepalive_probes (integer; default: 9; since Linux 2.2)
      The maximum number of TCP keep-alive probes to send before
      giving up and killing the connection if no response is
      obtained from the other end.

tcp_keepalive_time (integer; default: 7200; since Linux 2.2)
      The number of seconds a connection needs to be idle before TCP
      begins sending out keep-alive probes.  Keep-alives are sent
      only when the SO_KEEPALIVE socket option is enabled.  The
      default value is 7200 seconds (2 hours).  An idle connection
      is terminated after approximately an additional 11 minutes (9
      probes an interval of 75 seconds apart) when keep-alive is
      enabled.

      Note that underlying connection tracking mechanisms and
      application timeouts may be much shorter.
12345678910111213141516171819
```

这些属性可以在`/proc/sys/net/ipv4/`下查看：

```
cat /proc/sys/net/ipv4/tcp_keepalive_time
7200

cat /proc/sys/net/ipv4/tcp_keepalive_probes
9

cat /proc/sys/net/ipv4/tcp_keepalive_intvl
75
12345678
```

也可以通过命令行查看：

```
sudo sysctl -a | grep keepalive

net.ipv4.tcp_keepalive_time = 7200
net.ipv4.tcp_keepalive_probes = 9
net.ipv4.tcp_keepalive_intvl = 75
12345
```

## 1.2 连接探活的过程

开启`SO_KEEPALIVE`后，如果2小时内在此套接口的任一方向都没有数据交换，TCP就自动给对方发一个保持存活探测分节(keepalive probe)。这是一个对方必须响应的TCP分节.

它会导致以下三种情况：

-   对方接收一切正常：以期望的ACK响应。2小时后，TCP将发出另一个探测分节。
-   对方已崩溃且已重新启动：以RST响应。套接口的待处理错误被置为ECONNRESET，套接 口本身则被关闭。
-   对方无任何响应：源自berkeley的TCP发送另外8个探测分节，相隔75秒一个，试图得到一个响应。一共尝试9次，即在发出第一个探测分节11分钟 15秒后若仍无响应就放弃。套接口的待处理错误被置为ETIMEOUT，套接口本身则被关闭。如ICMP错误是“host unreachable(主机不可达)”，说明对方主机并没有崩溃，但是不可达，这种情况下待处理错误被置为 EHOSTUNREACH。

根据上面的介绍我们可以知道对端以一种非优雅的方式断开连接的时候，我们可以设置`SO_KEEPALIVE`属性使得我们在2小时以后发现对方的TCP连接是否依然存在。

```
int keepAlive = 1;

setsockopt(listenfd, SOL_SOCKET, SO_KEEPALIVE, (void*)&keepAlive, sizeof(keepAlive));
123
```

如果我们不能接受如此之长的等待时间，怎么办？

# 2.设置TCP KEEPALIVE

上面提到，`SO_KEEPALIVE`默认的时间间隔太长，不利于应用程序检测连接状态。

解决方法有2种：

-   全局设置
-   针对单个连接设置

## 2.1 全局设置

在Linux中我们可以通过修改 /etc/sysctl.conf 的全局配置：

```
net.ipv4.tcp_keepalive_time=7200
net.ipv4.tcp_keepalive_intvl=75
net.ipv4.tcp_keepalive_probes=9
123
```

添加上面的配置后输入 `sysctl -p` 使其生效，
你可以使用命令来查看当前的默认配置

```
sysctl -a | grep keepalive 
1
```

如果应用中已经设置SO_KEEPALIVE，程序不用重启，内核直接生效.

这种方法设置的全局的参数，针对整个系统生效，对单个socket的设置不够友好。

## 2.2 针对单个连接设置

我们可以使用TCP的`TCP_KEEPCNT`、`TCP_KEEPIDLE`、`TCP_KEEPINTVL`3个选项。
这些选项是连接级别的，每个socket都可以设置这些属性。

这些选项的定义，可以通过man查看。

```
man 7 tcp
1
```

socket option：

```
TCP_KEEPCNT (since Linux 2.4)
      The maximum number of keepalive probes TCP should send before
      dropping the connection.  This option should not be used in
      code intended to be portable.
      关闭一个非活跃连接之前的最大重试次数。
      该选项不具备可移植性。

TCP_KEEPIDLE (since Linux 2.4)
      The time (in seconds) the connection needs to remain idle
      before TCP starts sending keepalive probes, if the socket
      option SO_KEEPALIVE has been set on this socket.  This option
      should not be used in code intended to be portable.
      设置连接上如果没有数据发送的话，多久后发送keepalive探测分组，单位是秒
      该选项不具备可移植性。

TCP_KEEPINTVL (since Linux 2.4)
      The time (in seconds) between individual keepalive probes.
      This option should not be used in code intended to be
      portable.
      前后两次探测之间的时间间隔，单位是秒
      该选项不具备可移植性。
123456789101112131415161718192021
```

代码层面的设置步骤：

```
int keepAlive = 1;    // 非0值，开启keepalive属性

int keepIdle = 60;    // 如该连接在60秒内没有任何数据往来,则进行此TCP层的探测

int keepInterval = 5; // 探测发包间隔为5秒

int keepCount = 3;        // 尝试探测的最多次数

// 开启探活
setsockopt(sockfd, SOL_SOCKET, SO_KEEPALIVE, (void *)&keepAlive, sizeof(keepAlive));

setsockopt(sockfd, SOL_TCP, TCP_KEEPIDLE, (void*)&keepIdle, sizeof(keepIdle));

setsockopt(sockfd, SOL_TCP, TCP_KEEPINTVL, (void *)&keepInterval, sizeof(keepInterval));

setsockopt(sockfd, SOL_TCP, TCP_KEEPCNT, (void *)&keepCount, sizeof(keepCount) 
12345678910111213141516
```

# 3.为什么应用层需要heart beat/心跳包？

通过上面的介绍，感觉TCP keepalive已经很牛逼了，但为什么还会提到应用层的心跳呢？

目前了解的原因包括两个：

-   TCP keepalive处于传输层，由操作系统负责，能够判断进程存在，网络通畅，但无法判断进程阻塞或死锁等问题。
-   客户端与服务器之间有四层代理或负载均衡，即在传输层之上的代理，只有传输层以上的数据才被转发，例如socks5等

所以，基于以上原因，有时候还是需要应用程序自己去设计心跳规则的。
可以服务端负责周期发送心跳包，检测客户端，也可以客户端负责发送心跳包，或者服服务端和客户端同时发送心跳包。

可以根据具体的应用场景进行设计。