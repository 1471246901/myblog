# zookeeper实现分布式锁

每个客户端往/Locks下创建临时有序节点/Locks/Lock 000000001 

客户端取得/Locks下子节点，并进行排序，判断排在最前面的是否为自己，如果自己的 锁节点在第一位，代表获取锁成功 

如果自己的锁节点不在第一位，则监听自己前一位的锁节点。例如，自己锁节点 Lock 000000001 

当前一位锁节点（Lock 000000002）的逻辑 

监听客户端重新执行第2步逻辑，判断自己是否获得了锁

**实现案例**

```java
package org.gushiyu.zookeeperapi.component;

import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;

import java.util.Collections;
import java.util.List;
import java.util.concurrent.CountDownLatch;

/**
 * @Description
 * @Author Gushiyu
 * @Date 2021-01-26 11:56
 */
public class DistributedLock {

    private ZooKeeper zooKeeper;
    private final String zookeeperUrl;
    private final String lockPath;
    private final int sessionTimeout;
    private String path;

    private final Watcher watcher = new Watcher() {
        @Override
        public void process(WatchedEvent event) {
            if (event.getType() == Event.EventType.NodeDeleted) {
                synchronized (this) {
                    notifyAll();
                }
            }
        }
    };

    public DistributedLock(String zookeeperUrl, String lockPath, int sessionTimeout) throws Exception {
        this.zookeeperUrl = zookeeperUrl;
        this.lockPath = lockPath;
        this.sessionTimeout = sessionTimeout;
        content();
    }

    private void content() throws Exception {
        try {
            CountDownLatch count = new CountDownLatch(1);

            zooKeeper = new ZooKeeper(zookeeperUrl, sessionTimeout, new Watcher() {
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
        } catch (Exception e) {
            throw new Exception("未建立成功连接", e);
        }
    }


    /**
     * 加锁
     */
    public void lock() throws KeeperException, InterruptedException {

        //将自己加入锁的队列中
        createLock();
        //判断自己是否持有锁，没有就阻塞，有则继续运行
        attemptLock();

    }

    /**
     * 解锁
     */
    public void unlock() throws KeeperException, InterruptedException {
        //删除临时有序节点
        zooKeeper.delete(this.path,-1);
        zooKeeper.close();
        System.out.println("锁已经释放:"+this.path);
    }

    /**
     * 尝试获取锁
     */
    private void attemptLock() throws KeeperException, InterruptedException {


        // 获取Locks节点下的所有子节点
        List<String> list = zooKeeper.getChildren(lockPath, false);
        // 对子节点进行排序
        Collections.sort(list);
        //这里的index 是我们刚刚创建的path 在 lockPath中的位置，如果是0代表当前我们是持有锁的
        int index = list.indexOf(path.substring(lockPath.length() + 1));
        if(index != 0){
            //此时别的线程正持有锁，所以我们应该监听队列的前一项，并且让自己阻塞
            String previousPath = list.get(index - 1);
            Stat stat = zooKeeper.exists(lockPath + "/" + previousPath, watcher);
            //(stat == null) 代表加锁的时候，另一线程释放了锁，并且队列的前一项已经删除，
            // 此时上一步操作是无效的，watcher没有监听到，此时我们尝试再次加锁
            if(stat != null){
                //此时代表监听成功，所以我们应阻塞自己，等待watcher来唤醒
                synchronized (watcher) {
                    watcher.wait();
                }
            }
            //再次加锁
            attemptLock();
        }
        //此提示有可能一次加锁打印好多次, 因为存在再次加锁的可能性，在没有锁（自己直接就可以加上锁）时候打印一次，在有锁的时候打印两次
        System.out.println("加锁成功（此提示有可能一次加锁打印好多次）");


    }

    /**
     * 创建锁 并且添加自己到队列
     */
    private void createLock() throws KeeperException, InterruptedException {
        //如果不存在锁，就创建
        if(!existsLock()){
            try{
                //有可能含多个程序在同一时间创建锁，可能在创建锁的时候一起创建，这时就会报错，所以这个时候不进行处理就行
                zooKeeper.create(lockPath, new byte[0],
                        ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            }catch (Exception ignored){}
        }
        // 创建临时有序节点
        path = zooKeeper.create(lockPath + "/lock_", new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE,
                CreateMode.EPHEMERAL_SEQUENTIAL);
        System.out.println("已经有人持有过锁了(可能已经释放，或者没有释放)：节点创建成功:" + path);
    }

    /**
     * 是否存在锁
     * @return 存在锁 true  不存在 false
     */
    private boolean existsLock() throws KeeperException, InterruptedException {
        //判断Locks是否存在，
        Stat stat = zooKeeper.exists(lockPath, false);
        return stat != null;
    }


}

```

**测试**

```java
@Test
    public void testLock ( ) throws InterruptedException {
        final int[] p = {10};//10个商品

        for (int i = 0; i < 15; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        DistributedLock distributedLock = new DistributedLock("192.168.1.20","/lock_p",5000);
                        distributedLock.lock();
                        if(p[0]>0) {
							Thread.sleep(3000);
                            p[0] -=1;
                            System.out.println("成功卖出1个商品"+p[0]);
                        }else {
                            System.out.println("商品不够了，卖不出去"+p[0]);
                        }
                        
                        distributedLock.unlock();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }

                }
            }).start();
        }


        while (true){
            Thread.sleep(2000);
            System.out.println("商品P："+p[0]);
        }
    }
```

10个p商品成功全部卖出，没有多卖

**对比**

```java
@Test
    public void noLock ( ) throws InterruptedException {
        final int[] p = {10};//10个商品

        for (int i = 0; i < 15; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        if(p[0]>0) {
                            Thread.sleep(3000);
                            p[0] -=1;
                            System.out.println("成功卖出1个商品"+p[0]);
                        }else {
                            System.out.println("商品不够了，卖不出去"+p[0]);
                        }

                    } catch (Exception e) {
                        e.printStackTrace();
                    }

                }
            }).start();
        }


        while (true){
            Thread.sleep(2000);
            System.out.println("监视："+p[0]);
        }
    }
```

此时有可能会多卖