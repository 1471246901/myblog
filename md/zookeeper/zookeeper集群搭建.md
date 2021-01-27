# zookeeper集群搭建

使用一台虚拟机，进行伪集群搭建，zookeeper中包含三个节点，节点对外提供服务的端口号 2000 ，2001 ，2002

## 准备文件

将zookeeper程序复制三份，分别命名为`zookeeper2000 `   `zookeeper2001 `  `zookeeper2002  `

```powershell
gushiyu@gushiyuzookeeper:~/zookeeper$ mkdir zookeeperColony
gushiyu@gushiyuzookeeper:~/zookeeper$ cp -r apache-zookeeper-3.6.2-bin ./zookeeperColony/zookeeper2000
gushiyu@gushiyuzookeeper:~/zookeeper$ cp -r apache-zookeeper-3.6.2-bin ./zookeeperColony/zookeeper2001
gushiyu@gushiyuzookeeper:~/zookeeper$ cp -r apache-zookeeper-3.6.2-bin ./zookeeperColony/zookeeper2002
gushiyu@gushiyuzookeeper:~/zookeeper$ ls
apache-zookeeper-3.6.2-bin  apache-zookeeper-3.6.2-bin.tar.gz  zookeeperColony
gushiyu@gushiyuzookeeper:~/zookeeper$ cd zookeeperColony/
gushiyu@gushiyuzookeeper:~/zookeeper/zookeeperColony$ ls
zookeeper2000  zookeeper2001  zookeeper2002
```

## 修改配置文件

修改 2000 的配置文件

```
# 数据快照文件所在路径
dataDir=/home/gushiyu/zookeeper/zookeeperColony/zookeeper2000/data
# the port at which the clients will connect
# 服务端口号
clientPort=2000
# 集群配置信息
#  server.A=B:C:D
#  A:是一个数字，表示服务器编号
#  B:服务器IP地址
#  C:zookeeper 服务器之间沟通的端口号
#  D:Leader选举的端口号
server.1=192.168.1.20:2200:3000
server.2=192.168.1.20:2201:3001
server.3=192.168.1.20:2202:3002
```

修改 2001 的配置文件

```
# 数据快照文件所在路径
dataDir=/home/gushiyu/zookeeper/zookeeperColony/zookeeper2001/data
# the port at which the clients will connect
# 服务端口号
clientPort=2001
# 集群配置信息
#  server.A=B:C:D
#  A:是一个数字，表示服务器编号
#  B:服务器IP地址
#  C:zookeeper 服务器之间沟通的端口号
#  D:Leader选举的端口号
server.1=192.168.1.20:2200:3000
server.2=192.168.1.20:2201:3001
server.3=192.168.1.20:2202:3002
```

修改 2002 的配置文件

```
# 数据快照文件所在路径
dataDir=/home/gushiyu/zookeeper/zookeeperColony/zookeeper2002/data
# the port at which the clients will connect
# 服务端口号
clientPort=2002
# 集群配置信息
#  server.A=B:C:D
#  A:是一个数字，表示服务器编号
#  B:服务器IP地址
#  C:zookeeper 服务器之间沟通的端口号
#  D:Leader选举的端口号
server.1=192.168.1.20:2200:3000
server.2=192.168.1.20:2201:3001
server.3=192.168.1.20:2202:3002
```

## 创建myid文件

在配置文件的 `dataDir=/home/gushiyu/zookeeper/zookeeperColony/zookeeperXXXX/data` 文件夹下创建写有自己服务器号的文件  

```powershell
echo "1" > myid
```

## 分别启动三台zookeeper

```powershell
gushiyu@gushiyuzookeeper:~/zookeeper/zookeeperColony$ ./zookeeper2000/bin/zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /home/gushiyu/zookeeper/zookeeperColony/zookeeper2000/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
gushiyu@gushiyuzookeeper:~/zookeeper/zookeeperColony$ ./zookeeper2001/bin/zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /home/gushiyu/zookeeper/zookeeperColony/zookeeper2001/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
gushiyu@gushiyuzookeeper:~/zookeeper/zookeeperColony$ ./zookeeper2002/bin/zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /home/gushiyu/zookeeper/zookeeperColony/zookeeper2002/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

检查启动状态

```powershell
gushiyu@gushiyuzookeeper:~/zookeeper/zookeeperColony$ ./zookeeper2000/bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /home/gushiyu/zookeeper/zookeeperColony/zookeeper2000/bin/../conf/zoo.cfg
Client port found: 2000. Client address: localhost. Client SSL: false.
Mode: follower
gushiyu@gushiyuzookeeper:~/zookeeper/zookeeperColony$ ./zookeeper2001/bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /home/gushiyu/zookeeper/zookeeperColony/zookeeper2001/bin/../conf/zoo.cfg
Client port found: 2001. Client address: localhost. Client SSL: false.
Mode: follower
gushiyu@gushiyuzookeeper:~/zookeeper/zookeeperColony$ ./zookeeper2002/bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /home/gushiyu/zookeeper/zookeeperColony/zookeeper2002/bin/../conf/zoo.cfg
Client port found: 2002. Client address: localhost. Client SSL: false.
Mode: leader
```

## 登录到集群中

使用 `zkCli.sh -server  ip:port` 来登录

```
# 使用 2000 创建一个 znode  
[zk: 192.168.1.20:2000(CONNECTED) 5] create /gushiyu "gushiyu"
Created /gushiyu
# 在2001 和2002 查看
[zk: 192.168.1.20:2001(CONNECTED) 2] get /gushiyu
gushiyu
[zk: 192.168.1.20:2002(CONNECTED) 0] get /gushiyu
gushiyu

```

