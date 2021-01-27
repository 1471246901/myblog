# zookeeper完成生成分布式唯一ID

在过去的单库单表型系统中，通常可以使用数据库字段自带的auto_increment 属性来自动为每条记录生成一个唯一的ID。但是分库分表后，就无法在依靠数据库的 auto_increment属性来唯一标识一条记录了。此时我们就可以用zookeeper在分布式环 境下生成全局唯一ID。

**思路**

可以在一个节点下，生成临时有序节点，再把节点的名字的序列号作为分布式环境下的唯一ID

**实现案例**

```java
public class GloballyUniqueId {

    private ZooKeeper zooKeeper;
    private String zookeeperUrl;
    private String uniqueIdUrl;

    public GloballyUniqueId(String zookeeperUrl,String uniqueIdUrl) throws Exception{
        this.zookeeperUrl = zookeeperUrl;
        this.uniqueIdUrl = uniqueIdUrl;
        content();
    }

    private void content() throws Exception {
        try {
            CountDownLatch count = new CountDownLatch(1);

            zooKeeper = new ZooKeeper(zookeeperUrl, 5000, new Watcher() {
                @Override
                public void process(WatchedEvent watchedEvent) {
                    if (watchedEvent.getState() == Event.KeeperState.SyncConnected) {
                        System.out.println("连接创建成功！");
                        count.countDown();
                    }else if(watchedEvent.getState() == Event.KeeperState.Expired){
                        try {
                            content();
                        } catch (Exception e) {
                            new Exception("超时连接失败",e).printStackTrace();
                        }
                    }
                }
            });
            count.await();
            System.out.println("zookeeperSessionID :" + zooKeeper.getSessionId());
        }catch (Exception e){
            throw  new Exception("未建立成功连接",e);
        }
    }

    public String getUniqueId(){
        String path = "";
        try {
            path = zooKeeper.create(uniqueIdUrl,new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
        } catch (KeeperException|InterruptedException e) {
            e.printStackTrace();
        }
        return path.substring(uniqueIdUrl.length());

    }

}

```

**测试**

```java
@Test
    public void testGloballyUniqueId ( ) throws Exception {

        GloballyUniqueId globallyUniqueId = new GloballyUniqueId("192.168.1.20","/uniqueid");

        for (int i = 0; i < 10; i++) {
            Thread.sleep(1000);
            System.out.println(globallyUniqueId.getUniqueId());;
        }
    }
```

