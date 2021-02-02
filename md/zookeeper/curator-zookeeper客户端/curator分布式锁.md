# curator分布式锁

InterProcessMutex：分布式可重入排它锁 

InterProcessReadWriteLock：分布式读写锁

## InterProcessMutex 分布式可重入排它锁 

分布式可重入排它锁 

```java
// 排他锁
// arg1:连接对象
// arg2:节点路径
InterProcessLock interProcessLock = new InterProcessMutex(client,"/lock1");

// 获取锁
interProcessLock.acquire();

//同步代码块

// 释放锁
interProcessLock.release();


```

## InterProcessReadWriteLock 分布式读写锁

分布式读写锁

```java
// 读写锁
InterProcessReadWriteLock interProcessReadWriteLock=new InterProcessReadWriteLock(client, "/lock1");
// 获取读锁对象
InterProcessLock interProcessLock = interProcessReadWriteLock.readLock();

// 获取锁
interProcessLock.acquire();

// 同步代码块

// 释放锁
interProcessLock.release();


```

```java
// 读写锁
InterProcessReadWriteLock interProcessReadWriteLock=new InterProcessReadWriteLock(client, "/lock1");
// 获取读锁对象
InterProcessLock interProcessLock = interProcessReadWriteLock.writeLock();

// 获取锁
interProcessLock.acquire();

// 同步代码块

// 释放锁
interProcessLock.release();
```

