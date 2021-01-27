# zookeeper配置中心

zookeeper的watcher机制可以作为配置中心使用

工作中有这样的一个场景: 数据库用户名和密码信息放在一个配置文件中，应用 读取该配置文件，配置文件信息放入缓存。 若数据库的用户名和密码改变时候，还需要重新加载缓存，比较麻烦，通过 ZooKeeper可以轻松完成，当数据库发生变化时自动完成缓存同步。



**实现案例**

实现一个从数据库，不断查询信息，并且在数据库发生更改之后能自动切换数据库，不影响程序的正常工作

```java
package org.gushiyu.zookeeperapi.component;

import org.apache.log4j.net.SyslogAppender;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;

import javax.sql.DataSource;
import javax.swing.*;
import java.sql.SQLException;
import java.util.concurrent.CountDownLatch;

/**
 * @Description
 * @Author Gushiyu
 * @Date 2021-01-25 14:42
 */
public class DBConfigCenter {

    public String getUsername() {
        return username;
    }

    public String getUrl() {
        return url;
    }

    public String getPassword() {
        return password;
    }

    private String username;
    private String url;
    private String password;

    private String monitorNode;
    private Monitor monitor;

    private ZooKeeper zooKeeper;




    private class LsWatcher implements Watcher {

        @Override
        public void process(WatchedEvent event){
            if(event.getState() == Event.KeeperState.SyncConnected){
                if(event.getType() == Event.EventType.NodeDataChanged){
                    try {
                        System.out.println("检测到数据发生变化");
                        initVal();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }

    @FunctionalInterface
    public interface Monitor {
        void apply(String username,String url,String password) throws SQLException;
    }


    public DBConfigCenter(String zookeeperUrl,String monitorNode,Monitor monitor) throws Exception {
        this.monitorNode = monitorNode;
        this.monitor = monitor;
        try {
            CountDownLatch count = new CountDownLatch(1);

            ZooKeeper zooKeeper = new ZooKeeper(zookeeperUrl, 5000, new Watcher() {
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
            this.zooKeeper = zooKeeper;
        }catch (Exception e){
            throw  new Exception("未建立成功连接",e);
        }

        initVal();
    }

    private void initVal() throws Exception{

        try {
            this.username = new String(zooKeeper.getData(monitorNode+"/username",new LsWatcher(),null));
            this.url = new String(zooKeeper.getData(monitorNode+"/url",new LsWatcher(),null));
            this.password = new String(zooKeeper.getData(monitorNode+"/password",new LsWatcher(),null));
        } catch (Exception e) {
            throw  new Exception("数据获取异常",e);
        }
        monitor.apply(this.username,this.url,this.password);
    }
}

```



**测试**

```java
DBConfigCenter dbConfigCenter = new DBConfigCenter("192.168.1.20","/db",(n,url,p)->{
            System.out.println("发生改变");
            System.out.println(n);
            System.out.println(url);
            System.out.println(p);
        });

        while (true){
            Thread.sleep(3000);
        }
```

然后使用zkCli 修改节点数据，成功监听到改变，通过回调函数即可做到配置