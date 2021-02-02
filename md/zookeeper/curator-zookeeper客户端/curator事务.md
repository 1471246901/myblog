# curator事务

通过 `curatorClient`对象 的` inTransaction()` 方法可以打开事务

`commit()` 提交事务

```java
client.inTransaction().
    create().forPath("node1", "node1".getBytes()).
    and().
    create().forPath("node2", "node2".getBytes()).
    and().
    commit();
```

