# zookeeper Watcher 的监听次数机制

## 案例一

同一客户端使用不同new 出来的Watcher对一个znode节点进行多次监听，在触发事件之后，监听多少次就会有多少次回调，并且是一此事件把所有的监听全部触发完毕

```java
@Test
    public void testDoubleWatcher ( ) throws KeeperException, InterruptedException {

        for (int i = 0; i < 10; i++) {
            zooKeeper.getData("/watchertest",new LsWatcher() ,null);
        }
        for (int i = 0; i < 200; i++) {
            System.out.println("正在运行");
            System.out.println(atomicInteger.get());
            Thread.sleep(1000);
        }

    }
//atomicInteger == 10
```

## 案例二

同一客户端使用一个new 出来的Watcher对一个znode节点进行多次监听，在触发事件之后，只会响应一次(即只向服务器注册了一次) 

```java
@Test
    public void testDoubleWatcher ( ) throws KeeperException, InterruptedException {

        LsWatcher ls = new LsWatcher()
        for (int i = 0; i < 10; i++) {
            zooKeeper.getData("/watchertest", ls,null);
        }
        for (int i = 0; i < 200; i++) {
            System.out.println("正在运行");
            System.out.println(atomicInteger.get());
            Thread.sleep(1000);
        }

    }
//atomicInteger  == 1
```

## 案例三

一个 watcher对象分别被不同的 znode注册，当一个znode出发事件之后，不会影响另一znode ，

```java
@Test
    public void testDoubleWatcher ( ) throws KeeperException, InterruptedException {
        var ls = new LsWatcher();
            zooKeeper.getData("/watchertest",ls ,null);
            zooKeeper.getData("/watchertest2",ls,null);

        for (int i = 0; i < 200; i++) {
            System.out.println("正在运行");
            System.out.println(atomicInteger.get());
            Thread.sleep(1000);
        }

    }
```

## 案例四

和案例一一样，且不同znode 分别计算

```java
@Test
    public void testDoubleWatcher ( ) throws KeeperException, InterruptedException {

        for (int i = 0; i < 10; i++) {
            zooKeeper.getData("/watchertest",new LsWatcher() ,null);
        
            zooKeeper.getData("/watchertest2",new LsWatcher() ,null);
        }

        for (int i = 0; i < 200; i++) {
            System.out.println("正在运行");
            System.out.println(atomicInteger.get());
            Thread.sleep(1000);
        }

    }
```



