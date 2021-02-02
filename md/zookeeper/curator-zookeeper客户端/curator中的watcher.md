# curator watcher API

curator提供了两种Watcher(Cache)来监听结点的变化 

**Node Cache** : 只是监听某一个特定的节点，监听节点的新增和修改 

**PathChildren Cache** : 监控一个ZNode的子节点. 当一个子节点增加， 更新，删除 时， Path Cache会改变它的状态， 会包含最新的子节点， 子节点的数据和状态



## Node Cache

监视某个节点的数据变化

```java
@Test
public void watcher1() throws Exception {
    // 监视某个节点的数据变化
    // arg1:连接对象
    // arg2:监视的节点路径
    final NodeCache nodeCache=new NodeCache(client,"/watcher1");
    // 启动监视器对象
    nodeCache.start();
    nodeCache.getListenable().addListener(new NodeCacheListener() {
            // 节点变化时回调的方法
            public void nodeChanged() throws Exception {
            System.out.println(nodeCache.getCurrentData().getPath());
            System.out.println(new String(nodeCache.getCurrentData().getData()));
        }
    });
    Thread.sleep(100000);
    System.out.println("结束");
    //关闭监视器对象
    nodeCache.close();
}

```

## PathChildren Cache

监听子节点的变化

```java
@Test
public void watcher2() throws Exception {
    // 监视子节点的变化
    // arg1:连接对象
    // arg2:监视的节点路径
    // arg3:事件中是否可以获取节点的数据
    PathChildrenCache pathChildrenCache=new
    PathChildrenCache(client,"/watcher1",true);
    // 启动监听
    pathChildrenCache.start();
    pathChildrenCache.getListenable().addListener(new PathChildrenCacheListener() {
        // 当子节点方法变化时回调的方法
        public void childEvent(CuratorFramework curatorFramework, PathChildrenCacheEvent pathChildrenCacheEvent) throws Exception {
            // 节点的事件类型
            System.out.println(pathChildrenCacheEvent.getType());
            // 节点的路径
            System.out.println(pathChildrenCacheEvent.getData().getPath());
            // 节点数据
            System.out.println(new
            String(pathChildrenCacheEvent.getData().getData()));
        }
    });
    Thread.sleep(100000);
    System.out.println("结束");
    // 关闭监听
    pathChildrenCache.close();
}
```



**curator ** 版本5.1.0 后 `TreeCache `替换成`CuratorChache` ，`NodeCache` 等被弃用

可以使用 CuratorCache 监听节点，还有节点的子节点

## 5.1.0 后 CuratorCache 实现了链式风格

```java
CuratorCache cache = CuratorCache.build(client, "/hello3");
        CuratorCacheListener listener = CuratorCacheListener.builder()
                .forChanges((oldNode, node) -> {
                    System.out.println("old data: " + new String(node.getData()) + " new data:" + new String(node.getData()));
                })
                .forCreates((node) -> {
                    System.out.println("节点被创建：" + node.getPath());
                })
                .forDeletes((node) -> {
                    System.out.println("节点被删除：" + node.getPath());
                })
                .forInitialized(() -> System.out.println("Cache initialized"))
                .build();
        cache.listenable().addListener(listener);
        cache.start();
```

