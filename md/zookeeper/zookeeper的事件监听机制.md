# zookeeper的事件监听机制

## watcher

zookeeper提供了数据的发布/订阅功能，多个订阅者可同时监听某一特定主题对 象，当该主题对象的自身状态发生变化时(例如节点内容改变、节点下的子节点列表改变 等)，会实时、主动通知所有订阅者

zookeeper采用了Watcher机制实现数据的发布/订阅功能。该机制在被订阅对 象发生变化时会异步通知客户端，因此客户端不必在Watcher注册后轮询阻塞，从而减轻 了客户端压力。

watcher机制实际上与观察者模式类似，也可看作是一种观察者模式在分布式场 景下的实现方式。

### Watcher实现

**由三个部分组成：** 

-   Zookeeper服务端 
-   Zookeeper客户端 
-   客户端的ZKWatchManager对象 

客户端首先将Watcher注册到服务端，同时将Watcher对象保存到客户端的Watch管 理器中。当ZooKeeper服务端监听的数据状态发生变化时，服务端会主动通知客户端， 接着客户端的Watch管理器会触发相关Watcher来回调相应处理逻辑，从而完成整体的数 据发布/订阅流程。

![image-20210125110933784](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20210125110933784.png)

### watcher特性

| 特性            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| 一次性          | watcher是一次性的，一旦被触发就会移除，再次使用时需要重新注册 |
| 客户端顺序回 调 | watcher回调是顺序串行化执行的，只有回调后客户端才能看到最新的数 据状态。一个watcher回调逻辑不应该太多，以免影响别的watcher执行 |
| 轻量级          | WatchEvent是最小的通信单元，结构上只包含通知状态、事件类型和节点 路径，并不会告诉数据节点变化前后的具体内容； |
| 时效性          | watcher只有在当前session彻底失效时才会无效，若在session有效期内 快速重连成功，则watcher依然存在，仍可接收到通知； |

### watcher接口的设计

Watcher是Java的一个接口，任何实现了Watcher接口的类就是一个新的Watcher。 Watcher内部包含了两个枚举类：KeeperState、EventType

#### Watcher通知状态(KeeperState)

KeeperState是客户端与服务端连接状态发生变化时对应的通知类型。路径为 `org.apache.zookeeper.Watcher.Event.KeeperState`，是一个枚举类，其枚举属性 如下：

| 枚举属性      | 说明                     |
| ------------- | ------------------------ |
| SyncConnected | 客户端与服务器正常连接时 |
| Disconnected  | 客户端与服务器断开连接时 |
| Expired       | 会话session失效时        |
| AuthFailed    | 身份认证失败时           |

#### Watcher事件类型(EventType)

EventType是数据节点(znode)发生变化时对应的通知类型。**EventType变化时 KeeperState永远处于SyncConnected通知状态下**；当KeeperState发生变化时， EventType永远为None。其路径为`org.apache.zookeeper.Watcher.Event.EventType`， 是一个枚举类，枚举属性如下：

| 枚举属性            | 说明                                                       |
| ------------------- | ---------------------------------------------------------- |
| None                | 无                                                         |
| NodeCreated         | Watcher监听的数据节点被创建时                              |
| NodeDeleted         | Watcher监听的数据节点被删除时                              |
| NodeDataChanged     | Watcher监听的数据节点内容发生变更时(无论内容数据 是否变化) |
| NodeChildrenChanged | Watcher监听的数据节点的子节点列表发生变更时                |

==客户端接收到的相关事件通知中只包含状态及类型等信息，不包括节点变化前后的 具体内容，变化前的数据需业务自身存储，变化后的数据需调用get等方法重新获取；==

## 监听不同的事件

在zookeeper中采用 `zk.getChildren(path, watch)`、`zk.exists(path, watch)`、`zk.getData(path, watcher, stat) `这样的方式为某个znode注册监听。

| 注册方式                   | 创建 | 子节点变化 | 修改 | 删除 |
| -------------------------- | ---- | ---------- | ---- | ---- |
| zk.exists(path,watch)      | √    |            | √    | √    |
| zk.getData(path,watch)     |      |            | √    | √    |
| zk.getChildren(path,watch) |      | 删除和创建 |      | √    |

### 检查节点是否存在

使用连接对象的监视器 指的是使用注册zookeeper的watcher

```java
// boolean b  使用连接对象的监视器  
exists(String path, boolean b)
// 自定义监视器
exists(String path, Watcher w)
```

### 查看节点

```java
// 使用连接对象的监视器
getData(String path, boolean b, Stat stat)
// 自定义监视器
getData(String path, Watcher w, Stat stat)
// NodeDeleted:节点删除
// NodeDataChanged:节点内容发生变化
// Stat 指的是znode 的元数据，节点数据为方法返回值

//异步获取data
zooKeeper.getData(<path>, <watcher>, new AsyncCallback.DataCallback() {
            public void processResult(int rc, String path, Object ctx, byte[] data, Stat stat)
                //回调接口
            {}},"应用上下文");
```

**实现案例**

```java
@Test
    public void watcherTestGetData ( ) throws KeeperException, InterruptedException {

        Stat getstat = new Stat();
        byte[] returndata = zooKeeper.getData("/createapi/node8", new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println("event.getState: " + event.getState());
                if (event.getState() == Event.KeeperState.SyncConnected) {
                    switch (event.getType()) {
                        case None:
                            System.out.println("没有Type事件发生");
                            break;
                        case NodeDataChanged:
                            System.out.println(event.getPath() + "节点修改了");
                            break;
                        case NodeDeleted:
                            System.out.println(event.getPath() + "节点被删除了");
                            break;
                    }
                }
            }
        }, getstat);

        System.out.println("返回的数据是"+new String(returndata));
        System.out.println("节点状态"+getstat);
        System.out.println("已经返回数据并注册好watcher");
        for (int i = 0; i < 5; i++) {
            System.out.println("再过"+(5-i)+"秒就删除节点");
            Thread.sleep(1000);
        }
        System.out.println("删除");
        zooKeeper.delete("/createapi/node8",-1);

    }
```

### 查看子节点

```java
// 使用连接对象的监视器
getChildren(String path, boolean b)
// 自定义监视器
getChildren(String path, Watcher w)
// NodeChildrenChanged:子节点发生变化
// NodeDeleted:节点删除
```

**实现案例**

```java
@Test
    public void watcherTestGetChildren() throws KeeperException, InterruptedException {
        Stat getstat = new Stat();
        List<String> childrenlist = zooKeeper.getChildren("/createapi", new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println("event.getState: " + event.getState());
                if (event.getState() == Event.KeeperState.SyncConnected) {
                    switch (event.getType()) {
                        case ChildWatchRemoved:
                            System.out.println(event.getPath() + "ChildWatchRemoved");
                            break;
                        case NodeChildrenChanged:
                            System.out.println(event.getPath() + "的子节点被创建(删除)了");
                            break;
                        case NodeDeleted:
                            System.out.println(event.getPath() + "节点被删除了");
                            break;
                        default:
                            throw new IllegalStateException("Unexpected value: " + event.getType());
                    }
                }
            }
        }, getstat);

        System.out.println("返回的数据是" + childrenlist);
        System.out.println("节点状态" + getstat);
        System.out.println("已经返回子节点并注册好watcher");
        for (int i = 0; i < 5; i++) {
            System.out.println("再过" + (5 - i) + "秒就删除子节点");
            Thread.sleep(1000);
        }
        System.out.println("删除");
        zooKeeper.delete("/createapi/node7", -1);
    }


    @Test
    public void watcherTestGetChildren1() throws KeeperException, InterruptedException {
        Stat getstat = new Stat();
        List<String> childrenlist = zooKeeper.getChildren("/createapi", new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println("event.getState: " + event.getState());
                if (event.getState() == Event.KeeperState.SyncConnected) {
                    switch (event.getType()) {
                        case ChildWatchRemoved:
                            System.out.println(event.getPath() + "ChildWatchRemoved");
                            break;
                        case NodeChildrenChanged:
                            System.out.println(event.getPath() + "的子节点被创建(删除)了");
                            break;
                        case NodeDeleted:
                            System.out.println(event.getPath() + "节点被删除了");
                            break;
                        default:
                            System.out.println("Unexpected value: " + event.getType());
                    }
                }


            }
        }, getstat);

        System.out.println("返回的数据是" + childrenlist);
        System.out.println("节点状态" + getstat);
        System.out.println("已经返回子节点并注册好watcher");
        for (int i = 0; i < 5; i++) {
            System.out.println("再过" + (5 - i) + "秒就创建子节点");
            Thread.sleep(1000);
        }
        System.out.println("创建");
        zooKeeper.create("/createapi/node11", "data11".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    }
```

