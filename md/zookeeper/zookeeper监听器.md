# zookeeper监听器

## 监听器 get path [watch] 或 get -w path

使用 `get path [watch] `注册的监听器能够在节点内容发生改变的时候，向客 户端发出通知。需要注意的是 zookeeper 的触发器是一次性的 (One-time trigger)，即 触发一次后就会立即失效。

```shell
[zk: localhost:2181(CONNECTED) 15] get /hadoop
dsads1
[zk: localhost:2181(CONNECTED) 16] get /hadoop watch
'get path [watch]' has been deprecated. Please use 'get [-s] [-w] path' instead.
dsads1
[zk: localhost:2181(CONNECTED) 17] set /hadoop 1234

WATCHER::

WatchedEvent state:SyncConnected type:NodeDataChanged path:/hadoop

```

## 监听器 stat path [watch] 或 stat [-w] path

使用 `stat path [watch]` 注册的监听器能够在节点状态发生改变的时候，向客 户端发出通知

```shell
[zk: localhost:2181(CONNECTED) 19] stat /hadoop watch
'stat path [watch]' has been deprecated. Please use 'stat [-w] path' instead.
cZxid = 0x2
ctime = Fri Jan 22 03:01:49 UTC 2021
mZxid = 0x1a
mtime = Fri Jan 22 11:32:18 UTC 2021
pZxid = 0x2
cversion = 0
dataVersion = 10
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 0
[zk: localhost:2181(CONNECTED) 20] set /hadoop "12312313213213132132"

WATCHER::

WatchedEvent state:SyncConnected type:NodeDataChanged path:/hadoop
```

## 监听器 ls\ls2 path [watch]   或 ls [-w] path

使用 `ls path [watch]` 或 `ls2 path [watch]` 注册的监听器能够监听该节点下 所有子节点的增加和删除操作。  

**对子节点的增加**

```powershell
[zk: localhost:2181(CONNECTED) 21] ls /hadoop watch
'ls path [watch]' has been deprecated. Please use 'ls [-w] path' instead.
[]
[zk: localhost:2181(CONNECTED) 22] create /hadoop/yarn aaa

WATCHER::

WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/hadoop
Created /hadoop/yarn

```

**对子节点删除**

```powershell
[zk: localhost:2181(CONNECTED) 23] ls -w /hadoop
[yarn]
[zk: localhost:2181(CONNECTED) 24] delete /hadoop/yarn

WATCHER::

WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/hadoop
```

