# curator 使用

## 连接和创建

```java
public static void main(String[] args) {
        CuratorFramework client = CuratorFrameworkFactory.builder()
                .connectString("192.168.1.20:2181")
                .sessionTimeoutMs(5000)
                //会话超时时的重试策略
                //RetryOneTime 一次重连，当会话超时3000的时候
                .retryPolicy(new RetryOneTime(3000))
                //CuratorFramework 的操作都在curator znode 下进行
                .namespace("curator")
                .build();
        //打开链接
        client.start();
        System.out.println(client.checkExists());

        //关闭连接
        client.close();
    }
```

## 重连策略   retryPolicy

```java
//只进行一次重连
RetryPolicy retryPolicy = new RetryOneTime(3000);
//当连接超时后进行3次重连，每次间隔3秒
RetryPolicy retryPolicy2 = new RetryNTimes(3,3000 );

//每3秒重连一次，总等待时间超过10秒后重连
RetryPolicy retryPolicy3 = new RetryUntilElapsed(10000,3000);

//重连三次，根据key计算重连的等待时间
// 重连时间等于 = baseSleepTimeMs * Math.max(1, random.nextInt(1 << (retryCount+ 1)))
RetryPolicy retryPolicy4 = new ExponentialBackoffRetry(1000,3 );

```

## 新增节点

```java
          client.create()
                .withMode(CreateMode.PERSISTENT)
                .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
              	//这个方法可以递归创建节点
                .creatingParentsIfNeeded()
                //这个方法可以以异步方式创建，参数为回调类
              	.inBackground(new BackgroundCallback(){})
                //节点路径和结点数据
                .forPath("/node1","node1".getBytes());
```

## 更新节点

```java
@Test
public void set1() throws Exception {
// 更新节点
client.setData()
// arg1:节点的路径
// arg2:节点的数据
.forPath("/node1", "node11".getBytes());
System.out.println("结束");
}
@Test
public void set2() throws Exception {
client.setData()
// 指定版本号
.withVersion(2)
.forPath("/node1", "node1111".getBytes());
System.out.println("结束");
}
@Test
public void set3() throws Exception {
// 异步方式修改节点数据
client.setData()
.withVersion(-1).inBackground(new BackgroundCallback() {
public void processResult(CuratorFramework curatorFramework,
CuratorEvent curatorEvent) throws Exception {
// 节点的路径
System.out.println(curatorEvent.getPath());
// 事件的类型
System.out.println(curatorEvent.getType());
}
}).forPath("/node1", "node1".getBytes());
Thread.sleep(5000);
System.out.println("结束");
}
```

## 删除节点

```java
@Test
public void delete1() throws Exception {
// 删除节点
client.delete()
// 节点的路径
.forPath("/node1");
System.out.println("结束");
}
@Test
public void delete2() throws Exception {
client.delete()
// 版本号
.withVersion(0)
.forPath("/node1");
System.out.println("结束");
}
@Test
public void delete3() throws Exception {
//删除包含字节点的节点
client.delete()
.deletingChildrenIfNeeded()
.withVersion(-1)
.forPath("/node1");
System.out.println("结束");
}
@Test
public void delete4() throws Exception {
// 异步方式删除节点
client.delete()
.deletingChildrenIfNeeded()
.withVersion(-1)
.inBackground(new BackgroundCallback() {
public void processResult(CuratorFramework
curatorFramework, CuratorEvent curatorEvent) throws Exception {
// 节点路径
System.out.println(curatorEvent.getPath());
// 事件类型
System.out.println(curatorEvent.getType());
}
})
.forPath("/node1");
Thread.sleep(5000);
System.out.println("结束");
}
```

## 查看节点

```java
@Test
public void get1() throws Exception {
// 读取节点数据
byte [] bys=client.getData()
// 节点的路径
.forPath("/node1");
System.out.println(new String(bys));
}
@Test
public void get2() throws Exception {
// 读取数据时读取节点的属性
Stat stat=new Stat();
byte [] bys=client.getData()
// 读取属性
.storingStatIn(stat)
.forPath("/node1");
System.out.println(new String(bys));
System.out.println(stat.getVersion());
}
@Test
public void get3() throws Exception {
// 异步方式读取节点的数据
client.getData()
.inBackground(new BackgroundCallback() {
public void processResult(CuratorFramework
curatorFramework, CuratorEvent curatorEvent) throws Exception {
// 节点的路径
System.out.println(curatorEvent.getPath());
// 事件类型
System.out.println(curatorEvent.getType());
// 数据
System.out.println(new
String(curatorEvent.getData()));
}
})
.forPath("/node1");
Thread.sleep(5000);
System.out.println("结束");
}
```

## 查看子节点

```java
@Test
public void getChild1() throws Exception {
// 读取子节点数据
List<String> list = client.getChildren()
// 节点路径
.forPath("/get");
for (String str : list) {
System.out.println(str);
}
}
@Test
public void getChild2() throws Exception {
// 异步方式读取子节点数据
client.getChildren()
.inBackground(new BackgroundCallback() {
public void processResult(CuratorFramework
curatorFramework, CuratorEvent curatorEvent) throws Exception {
// 节点路径
System.out.println(curatorEvent.getPath());
// 事件类型
System.out.println(curatorEvent.getType());
// 读取子节点数据
List<String> list=curatorEvent.getChildren();
for (String str : list) {
System.out.println(str);
}
}
})
.forPath("/get");
Thread.sleep(5000);
System.out.println("结束");
}
```

## 检查节点是否存在

```java
@Test
public void exists1() throws Exception {
// 判断节点是否存在
Stat stat= client.checkExists()
// 节点路径
.forPath("/node2");
System.out.println(stat.getVersion());
}
@Test
public void exists2() throws Exception {
// 异步方式判断节点是否存在
client.checkExists()
.inBackground(new BackgroundCallback() {
public void processResult(CuratorFramework
curatorFramework, CuratorEvent curatorEvent) throws Exception {
// 节点路径
System.out.println(curatorEvent.getPath());
// 事件类型
System.out.println(curatorEvent.getType());
System.out.println(curatorEvent.getStat().getVersion());
}
})
.forPath("/node2");
Thread.sleep(5000);
System.out.println("结束");
}
```

