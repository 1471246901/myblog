# zookeeperCli的常用操作

#### zookeeper节点类似于树的形式进行保存

#### zookeeper节点类型

-   永久节点：已创建就会一直保存，直到手动删除

-   临时节点：一旦session断开，临时节点便会删除

#### zookeeper每个数据节点之间都有一个版本号，在每次修改数据的时候对应的版本号回增加。

#### zookeeper的节点不适合存放大量的数据

#### zookeeper使用场景：

-   分布式锁
-   dubbo的注册中心

#### 使用zkCli客户端工具

可以使用apache-zookeeper-3.5.6-bin\bin下的zkCli.cmd连接本地或者远程启动的zookeeper，下面是zkCli的使用命令

使用powershell进入到`apache-zookeeper-3.5.6-bin\bin`目录下

在windows下运行的是.cmd文件，在Linux下运行的是.sh文件

.\zkCli.cmd 不填后面的参数，默认连接的是`localhost:2181`

**连接远程服务器**

`.\zkCli.cmd -timeout 5000 -r -server ip:2181`

#### zkCli命令

##### 查看帮助文档

```shell
./zkCli.sh h
1
```

##### 创建一个节点

`create [-s] [-e] path [data] [acl]`

 -s 顺序创建

 -e 创建一个临时节点

  *创建临时节点，临时节点会在会话过期后被删除*

 path 创建的路径

 data 节点中存放的数据

 acl 权限相关

##### 创建一个顺序节点

```shell
create -s /testnode test
Created /testnode0000000000
```

Created /testnode0000000000,后面的数据随着创建的节点增加

##### **获取节点的子节点：ls path**

```shell
ls /
```

##### **获取节点的数据：get path**

```shell
get /testnode0000000000
```

test  --返回数据

##### **查看节点的状态: stat path**

```shell
stat /testnode0000000000
```

节点各个属性如下表。其中一个重要的概念是 Zxid(ZooKeeper Transaction Id)，ZooKeeper 节点的每一次更改都具有唯一的 Zxid，如果 Zxid1 小于 Zxid2，则 Zxid1 的更改发生在 Zxid2 更改之前。

| 状态属性       | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| cZxid          | 数据节点创建时的事务 ID                                      |
| ctime          | 数据节点创建时的时间                                         |
| mZxid          | 数据节点最后一次更新时的事务 ID                              |
| mtime          | 数据节点最后一次更新时的时间                                 |
| pZxid          | 数据节点的子节点最后一次被修改时的事务 ID                    |
| cversion       | 子节点的更改次数                                             |
| dataVersion    | 节点数据的更改次数                                           |
| aclVersion     | 节点的 ACL 的更改次数                                        |
| ephemeralOwner | 如果节点是临时节点，则表示创建该节点的会话的 SessionID；如果节点是持久节点，则该属性值为 0 |
| dataLength     | 数据内容的长度   字节数                                      |
| numChildren    | 数据节点当前的子节点个数                                     |



##### **查看节点的子节点以及当前节点的状态**

查看节点列表有 ls path 和 ls2 path 两个命令，后者是前者的增强，不仅可以查看指定路径下的所有节点，还可以查看当前节点的信息

```shell
ls2  /testnode0000000000   # 或者使用 ls -s /节点名
```

[data]cZxid = 0x5 --子节点
ctime = Sun Oct 27 11:47:38 CST 2019 --当前节点
mZxid = 0x5
mtime = Sun Oct 27 11:47:38 CST 2019
pZxid = 0x8
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 1

##### **修改节点中的数据： `set [-v version] path data`**

```shell
set /testnode0000000000 test
```

##### **删除一个节点**

删除某个节点，如果当前节点存在子节点则不允许删除，如果版本不符合也不允许删除 ： delete [-v version] path

递归删除1：rmr path

递归删除2： deleteall path



