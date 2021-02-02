# zookeeper JavaAPI

zookeeper是由java开发的，它有两种客户端，一种是zkCli 另一种就是api客户端

znode是zooKeeper集合的核心组件，zookeeper API提供了一小组方法使用 zookeeper集合来操纵znode的所有细节。 

客户端应该遵循以步骤，与zookeeper服务器进行清晰和干净的交互。 

-   连接到zookeeper服务器。zookeeper服务器为客户端分配会话ID。 
-   定期向服务器发送心跳。否则，zookeeper服务器将过期会话ID，客户端需要重新连 接。 
-   只要会话ID处于活动状态，就可以获取/设置znode。 
-   所有任务完成后，断开与zookeeper服务器的连接。如果客户端长时间不活动，则 zookeeper服务器将自动断开客户端。



## 创建项目

新建maven项目

引入

```xml
<dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.6.2</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.10</version>
            <scope>test</scope>
        </dependency>
```

## zookeeper的连接



```java
ZooKeeper(String connectionString, int sessionTimeout, Watcher watcher)
```

**在进行集群链接时，connectionString 参数使用逗号把ip隔开 如：**

```java
ZooKeeper("192.168.60.130:2181,192.168.60.130:2182,192.168.60.130:2183",sessionTimeout,Watcher)
```

-   connectString（String）：连接串，包括ip+端口 ,集群模式下用**逗号**隔开；
-   sessionTimeout（int）：会话超时时间，该值不能超过服务端所设置的minSessionTimeout 和maxSessionTimeout；
-   watcher（Watcher）：会话监听器，服务端事件将会触该监听；
-   sessionId（long）：自定义会话ID；
-   sessionPasswd（byte[]）：会话密码；
-   canBeReadOnly（boolean）：该连接是否为只读的；
-   hostProvider（HostProvider）：服务端地址提供者，指示客户端如何选择某个服务来调用，默认采用StaticHostProvider实现；

**实现案例**

```java
public static ZooKeeper getConnection() throws Exception {
        try {
            CountDownLatch count = new CountDownLatch(1);

            ZooKeeper zooKeeper = new ZooKeeper("192.168.1.20", 5000, new Watcher() {
                @Override
                public void process(WatchedEvent watchedEvent) {
                    if (watchedEvent.getState() == Event.KeeperState.SyncConnected) {
                        System.out.println("连接创建成功！");
                        count.countDown();
                    }
                }
            });
            count.await();
            System.out.println("zookeeperSessionID :" + zooKeeper.getSessionId());
            return zooKeeper;
        }catch (Exception e){
            throw  new Exception("未建立成功连接",e);
        }
    }
    
    
    
//Test
@Test
    public void testConnection() throws Exception {
        ZooKeeper zoo = ZookeeperApiMain.getConnection();
        zoo.close();
    }
```

结果

```
连接创建成功！
zookeeperSessionID :72057624837554178
```

## 新增节点

```
// 同步方式
create(String path, byte[] data, List<ACL> acl, CreateMode createMode)
// 异步方式
create(String path, byte[] data, List<ACL> acl, CreateMode createMode，
AsyncCallback.StringCallback callBack,Object ctx)
```

create的参数说明：

-   path（String）：节点路径；

-   data（btye[]）：节点存储的数据；

-   acl（List<ACL>）：节点的权限；

    ZooDefs.Ids 来获取一些基本的acl列表。例如，ZooDefs.Ids.OPEN_ACL_UNSAFE 返回打开znode的acl列表

-   createMode（CreateMode）：节点的类型 (枚举)（持久、持久序号、临时、临时序号）

-   cb（StringCallback）： 异步回调接口

-   ctx（Object）：传递上下文参数

**实现案例**

```java
public class CreateTest {


    ZooKeeper zooKeeper;

    @Before
    public void before() throws Exception {
        zooKeeper = ZookeeperApiMain.getConnection();
    }
	@after
    public void after() throws Exception {
        zooKeeper.close();
    }


    /**
     * // arg1:节点的路径
     * // arg2:节点的数据
     * // arg3:权限列表 world:anyone:cdrwa
     * // arg4:节点类型 持久化节点
     */
    @Test
    public void create1() throws Exception {
        zooKeeper.create("/createapi/node1", "node1".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    }

    /**
     * arg3:权限列表 world:anyone:r
     */
    @Test
    public void create2() throws Exception {
        zooKeeper.create("/createapi/node2", "node2".getBytes(), ZooDefs.Ids.READ_ACL_UNSAFE, CreateMode.PERSISTENT);
    }

    /**
     * 持久的节点，并添加了自定义ACL权限
     */
    @Test
    public void create3() throws Exception {

        List<ACL> acls = new ArrayList<>();
        //授权对象
        Id id = new Id("world", "anyone");
        acls.add(new ACL(ZooDefs.Perms.READ, id));
        acls.add(new ACL(ZooDefs.Perms.WRITE, id));
        zooKeeper.create("/createapi/node3", "node3".getBytes(), acls, CreateMode.PERSISTENT);
    }

    /**
     * ip持久授权模式
     */
    @Test
    public void create4() throws Exception {

        // 权限列表
        List<ACL> acls = new ArrayList<ACL>();
        // 授权模式和授权对象
        Id id = new Id("ip", "192.168.1.7");
        // 权限设置
        acls.add(new ACL(ZooDefs.Perms.ALL, id));
        zooKeeper.create("/createapi/node4", "node4".getBytes(), acls,
                CreateMode.PERSISTENT);
    }

    /**
     * auth授权模式  授权所有权限
     * ZooDefs.Ids.CREATOR_ALL_ACL 此ACL授予创建者身份验证ID的所有权限
     */
    @Test
    public void create5() throws Exception {
        // auth授权模式
        // 添加授权用户
        zooKeeper.addAuthInfo("digest", "gushiyu:123456".getBytes());
        zooKeeper.create("/createapi/node5", "node5".getBytes(),
                ZooDefs.Ids.CREATOR_ALL_ACL, CreateMode.PERSISTENT);
    }

    /**
     * 为 /createapi/node6 节点添加 auth 授权，并且为只读
     *
     * @throws Exception
     */
    @Test
    public void create6() throws Exception {
        // auth授权模式
        // 添加授权用户
        zooKeeper.addAuthInfo("digest", "gushiyu:123456".getBytes());
        // 权限列表
        List<ACL> acls = new ArrayList<ACL>();
        // 授权模式和授权对象
        Id id = new Id("auth", "gushiyu");
        // 权限设置
        acls.add(new ACL(ZooDefs.Perms.READ, id));
        zooKeeper.create("/createapi/node6", "node6".getBytes(), acls,
                CreateMode.PERSISTENT);
    }


    /**
     * 使用 digest 模式授予全部权限
     */
    @Test
    public void create7() throws Exception {
        // digest授权模式
        // 权限列表
        List<ACL> acls = new ArrayList<ACL>();
        // 授权模式和授权对象
        Id id = new Id("digest", "gushiyu:LJiKfP2bedgKoVwhcbHHodr68Rs=");
        // 权限设置
        acls.add(new ACL(ZooDefs.Perms.ALL, id));
        zooKeeper.create("/createapi/node7", "node7".getBytes(), acls,
                CreateMode.PERSISTENT);
    }

    @Test
    public void create8() throws Exception {
        CountDownLatch count = new CountDownLatch(1);

        // 异步方式创建节点
        zooKeeper.create("/createapi/node8", "node8".getBytes(),
                ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT, new
                        AsyncCallback.StringCallback() {
                            @Override
                            public void processResult(int rc, String path, Object ctx,
                                                      String name) {
                                // 0 代表创建成功
                                System.out.println("创建结果"+rc);
                                // 节点的路径
                                System.out.println("路径"+path);
                                // 节点的路径
                                System.out.println("节点的路径"+name);
                                // 上下文参数
                                System.out.println("上下文参数"+ctx);
                                count.countDown();
                            }
                        }, "I am context");
        count.await();
        System.out.println("结束");
    }
}

```

## 更新节点

```java
// 同步方式
setData(String path, byte[] data, int version)
// 异步方式
setData(String path, byte[] data, int version，AsyncCallback.StatCallback
callBack， Object ctx)
```

-   path- znode路径 
-   data - 要存储在指定znode路径中的数据。 
-   version- znode的当前版本。每当数据更改时，ZooKeeper会更新znode的版本 号。 
-   callBack-异步回调接口 
-   ctx-传递上下文参数

**实现案例**

```java
public class SetTest {
    ZooKeeper zooKeeper;

    @Before
    public void before() throws Exception {
        zooKeeper = ZookeeperApiMain.getConnection();
    }

    @After
    public void after() throws Exception {
        zooKeeper.close();
    }

    /**
     * 通过版本号修改
     */
    @Test
    public void set1() throws Exception {
        //[zk: localhost:2181(CONNECTED) 5] get /set/node1
        //node
        // arg3:版本号 -1代表版本号不作为修改条件
        Stat stat = zooKeeper.setData("/set/node1", "modify1".getBytes(), -1);
        // 节点的版本号
        System.out.println(stat.getVersion());
        // 节点的创建时间
        System.out.println(stat.getCtime());
    }

    /**
     *  异步方式修改节点
     */
    @Test
    public void set2() throws Exception {

        CountDownLatch countDownLatch = new CountDownLatch(1);

        zooKeeper.setData("/set/node2", "modify2".getBytes(), -1, new
                AsyncCallback.StatCallback() {
                    @Override
                    public void processResult(int rc, String path, Object ctx,
                                              Stat stat) {
                        // 0 代表修改成功
                        System.out.println("修改状态"+rc);
                        // 修改节点的路径
                        System.out.println("路径"+path);
                        // 上线文的参数对象
                        System.out.println("上下文参数"+ctx);
                        // 的属性信息
                        System.out.println("修改后版本号"+stat.getVersion());
                        countDownLatch.countDown();
                    }
                }, "I am Context");
        countDownLatch.await();
        System.out.println("异步方式修改节点结束");
    }
}

```

## 删除节点

```java
// 同步方式
delete(String path, int version)
// 异步方式
delete(String path, int version, AsyncCallback.VoidCallback callBack,
Object ctx)
```

-   path - znode路径。 
-   version - znode的当前版本 
-   callBack-异步回调接口 
-   ctx-传递上下文参数

**实现案例**

```java
public void delete1() throws Exception {
        // arg1:删除节点的节点路径
        // arg2:数据版本信息 -1代表删除节点时不考虑版本信息
        zooKeeper.delete("/delete/node1",-1);
    }
    @Test
    public void delete2() throws Exception {
        // 异步使用方式
        zooKeeper.delete("/delete/node2", -1, new
                AsyncCallback.VoidCallback() {
                    @Override
                    public void processResult(int rc, String path, Object ctx) {
                        // 0代表删除成功
                        System.out.println(rc);
                        // 节点的路径
                        System.out.println(path);
                        // 上下文参数对象
                        System.out.println(ctx);
                    }
                },"I am Context");
        Thread.sleep(10000);
        System.out.println("结束");
    }
```

## 查看节点

```java
// 同步方式
getData(String path, boolean b, Stat stat)
// 异步方式
getData(String path, boolean b，AsyncCallback.DataCallback callBack，
Object ctx)
```

-   path - znode路径。 
-   b- 是否使用连接对象中注册的监视器。 
-   stat - 返回znode的元数据。 
-   callBack-异步回调接口 
-   ctx-传递上下文参数

**实现案例**

```java
@Test
    public void get1() throws Exception {
        // arg1:节点的路径
        // arg3:读取节点属性的对象
        Stat stat=new Stat();
        byte [] bys=zooKeeper.getData("/get/node1",false,stat);
        // 打印数据
        System.out.println(new String(bys));
        // 版本信息
        System.out.println(stat.getVersion());
    }
    @Test
    public void get2() throws Exception {
        //异步方式
        zooKeeper.getData("/get/node1", false, new
                AsyncCallback.DataCallback() {
                    @Override
                    public void processResult(int rc, String path, Object ctx,
                                              byte[] data, Stat stat) {
                        // 0代表读取成功
                        System.out.println(rc);
                        // 节点的路径
                        System.out.println(path);
                        // 上下文参数对象
                        System.out.println(ctx);
                        // 数据
                        System.out.println(new String(data));
                        // 属性对象
                        System.out.println(stat.getVersion());
                    }
                },"I am Context");
        Thread.sleep(10000);
        System.out.println("结束");
    }
```

## 查看子节点

```java
// 同步方式
getChildren(String path, boolean b)
// 异步方式
getChildren(String path, boolean b,AsyncCallback.ChildrenCallback
callBack,Object ctx)
```

-   path - Znode路径。 
-   b- 是否使用连接对象中注册的监视器。 
-   callBack - 异步回调接口。 
-   ctx-传递上下文参数

## 检查节点是否存在

```java
// 同步方法
exists(String path, boolean b)
// 异步方法
exists(String path, boolean b，AsyncCallback.StatCallback callBack,Object
ctx)
```

-   path- znode路径。 
-   b- 是否使用连接对象中注册的监视器。 
-   callBack - 异步回调接口。 
-   ctx-传递上下文参数