# zookeeper四字命令

zooKeeper支持某些特定的四字命令与其的交互。它们大多是查询命令，用来 获取 zooKeeper服务的当前状态及相关信息。用户在客户端可以通过 telnet 或 nc 向 zooKeeper提交相应的命令。 zooKeeper常用四字命令见下表 所示：

| 命令 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| conf | 输出相关服务配置的详细信息。比如端口、zk数据及日志配置路径、最大 连接数，session超时时间、serverId等 |
| cons | 列出所有连接到这台服务器的客户端连接/会话的详细信息。包括“接受/发 送”的包数量、session id 、操作延迟、最后的操作执行等信息 |
| crst | 重置当前这台服务器所有连接/会话的统计信息                    |
| dump | 列出未经处理的会话和临时节点                                 |
| envi | 输出关于服务器的环境详细信息                                 |
| ruok | 测试服务是否处于正确运行状态。如果正常返回"imok"，否则返回空 |
| stat | 输出服务器的详细信息：接收/发送包数量、连接数、模式 （leader/follower）、节点总数、延迟。 所有客户端的列表 |
| srst | 重置server状态                                               |
| wchs | 列出服务器watches的简洁信息：连接总数、watching节点总数和 watches总数 |
| wchc | 通过session分组，列出watch的所有节点，它的输出是一个与 watch 相关 的会话的节点列表 |
| mntr | 列出集群的健康状态。包括“接受/发送”的包数量、操作延迟、当前服务模 式（leader/follower）、节点总数、watch总数、临时节点总数 |

nc命令工具安装:

```
#root用户安装
#下载安装包
wget http://vault.centos.org/6.6/os/x86_64/Packages/nc-1.84-
22.el6.x86_64.rpm
#rpm安装
rpm -iUv nc-1.84-22.el6.x86_64.rpm
```

使用方式，在shell终端输入：`echo mntr | nc localhost 2181`



## 监控指标

```sell
conf:
clientPort:客户端端口号 
dataDir：数据文件目录
dataLogDir：日志文件目录  
tickTime：间隔单位时间
maxClientCnxns：最大连接数  
minSessionTimeout：最小session超时
maxSessionTimeout：最大session超时  
serverId：id  
initLimit：初始化时间  
syncLimit：心跳时间间隔  
electionAlg：选举算法 默认3  
electionPort：选举端口  
quorumPort：法人端口  
peerType：未确认

cons：
ip=ip
port=端口
queued=所在队列
received=收包数
sent=发包数
sid=session id
lop=最后操作
est=连接时间戳
to=超时时间
lcxid=最后id(未确认具体id)
lzxid=最后id(状态变更id)
lresp=最后响应时间戳
llat=最后/最新 延时
minlat=最小延时
maxlat=最大延时
avglat=平均延时


crst:
重置所有连接


dump:
session id : znode path  (1对多   ,  处于队列中排队的session和临时节点)


envi:
zookeeper.version=版本
host.name=host信息
java.version=java版本
java.vendor=供应商
java.home=jdk目录
java.class.path=classpath
java.library.path=lib path
java.io.tmpdir=temp目录
java.compiler=<NA>
os.name=Linux
os.arch=amd64
os.version=2.6.32-358.el6.x86_64
user.name=hhz
user.home=/home/hhz
user.dir=/export/servers/zookeeper-3.4.6


ruok:
查看server是否正常
imok=正常


srst:
重置server状态


srvr：
Zookeeper version:版本
Latency min/avg/max: 延时
Received: 收包
Sent: 发包
Connections: 连接数
Outstanding: 堆积数
Zxid: 操作id
Mode: leader/follower
Node count: 节点数

stat：
Zookeeper version: 3.4.6-1569965, built on 02/20/2014 09:09 GMT
Clients:
 /192.168.147.102:56168[1](queued=0,recved=41,sent=41)
 /192.168.144.102:34378[1](queued=0,recved=54,sent=54)
 /192.168.162.16:43108[1](queued=0,recved=40,sent=40)
 /192.168.144.107:39948[1](queued=0,recved=1421,sent=1421)
 /192.168.162.16:43112[1](queued=0,recved=54,sent=54)
 /192.168.162.16:43107[1](queued=0,recved=54,sent=54)
 /192.168.162.16:43110[1](queued=0,recved=53,sent=53)
 /192.168.144.98:34702[1](queued=0,recved=41,sent=41)
 /192.168.144.98:34135[1](queued=0,recved=61,sent=65)
 /192.168.162.16:43109[1](queued=0,recved=54,sent=54)
 /192.168.147.102:56038[1](queued=0,recved=165313,sent=165314)
 /192.168.147.102:56039[1](queued=0,recved=165526,sent=165527)
 /192.168.147.101:44124[1](queued=0,recved=162811,sent=162812)
 /192.168.147.102:39271[1](queued=0,recved=41,sent=41)
 /192.168.144.107:45476[1](queued=0,recved=166422,sent=166423)
 /192.168.144.103:45100[1](queued=0,recved=54,sent=54)
 /192.168.162.16:43133[0](queued=0,recved=1,sent=0)
 /192.168.144.107:39945[1](queued=0,recved=1825,sent=1825)
 /192.168.144.107:39919[1](queued=0,recved=325,sent=325)
 /192.168.144.106:47163[1](queued=0,recved=17891,sent=17891)
 /192.168.144.107:45488[1](queued=0,recved=166554,sent=166555)
 /172.17.36.11:32728[1](queued=0,recved=54,sent=54)
 /192.168.162.16:43115[1](queued=0,recved=54,sent=54)

Latency min/avg/max: 0/0/599
Received: 224869
Sent: 224817
Connections: 23
Outstanding: 0
Zxid: 0x68000af707
Mode: follower
Node count: 101081

（同上面的命令整合的信息）


wchs:
connectsions=连接数
watch-paths=watch节点数
watchers=watcher数量


wchc:
session id 对应 path

wchp:
path 对应 session id

mntr:
zk_version=版本
zk_avg_latency=平均延时
zk_max_latency=最大延时
zk_min_latency=最小延时
zk_packets_received=收包数  
zk_packets_sent=发包数
zk_num_alive_connections=连接数
zk_outstanding_requests=堆积请求数
zk_server_state=leader/follower 状态
zk_znode_count=znode数量
zk_watch_count=watch数量
zk_ephemerals_count=临时节点（znode）
zk_approximate_data_size=数据大小
zk_open_file_descriptor_count=打开的文件描述符数量
zk_max_file_descriptor_count=最大文件描述符数量
zk_followers=follower数量
zk_synced_followers=同步的follower数量
zk_pending_syncs=准备同步数
```

