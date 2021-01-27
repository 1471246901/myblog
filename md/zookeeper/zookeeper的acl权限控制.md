# zookeeper的acl权限控制

zookeeper 类似文件系统，client 可以创建节点、更新节点、删除节点，zookeeper的access control list 访问控制列表可以做到可以做到acl 权限控制。

使用`scheme：id：permission `来标识

主要涵盖 3 个方面： 

-   权限模式（scheme）：授权的策略
-   授权对象（id）：授权的对象
-   权限（permission）：授予的权限

特性：

-   zooKeeper的权限控制是基于每个znode节点的，需要对每个节点设置权限
-   每个znode支持设置多种权限控制方案和多个权限
-   子节点不会继承父节点的权限，客户端无权访问某节点，但可能可以访问它的子节点

例如：

```powershell
setAcl /hadoop ip:192.168.1.20:crwda 
// 将节点权限设置为Ip:192.168.60.130的客户端可以对节点进行增、删、改、查、管理权限
```

## 权限模式

| 授权模式 | 描述                                                 |
| -------- | ---------------------------------------------------- |
| world    | 只有一个用户：anyone，代表登录zokeeper所有人（默认） |
| ip       | 对客户端使用IP地址认证                               |
| auth     | 使用已添加认证的用户认证  明文密码                   |
| digest   | 使用“用户名:密码”方式认证 加密密码                   |

## 授权对象

指给谁授予权限   例如 anyone ，IP ，用户

## 权限

create、delete、read、writer、admin也就是 增、删、改、查、管理权限， 这5种权限简写为cdrwa，注意:这5种权限中，delete是指对子节点的删除权限，其它4种 权限指对自身节点的操作权限

| 权限   | ACL简写 | 描述                             |
| ------ | ------- | -------------------------------- |
| create | c       | 可以创建子节点                   |
| delete | d       | 可以删除子节点（仅下一级节点）   |
| read   | r       | 可以读取节点数据及显示子节点列表 |
| write  | w       | 可以设置节点数据                 |
| admin  | a       | 可以设置节点访问控制列表权限     |

## 授权使用的命令

| 命令    | 使用方式            | 描述         |
| ------- | ------------------- | ------------ |
| getAcl  | getAcl <path>       | 读取权限     |
| setAcl  | setAcl <path> <acl> | 设置权限     |
| addauth | addauth             | 添加认证用户 |

## world授权模式

```
setAcl <path> world:anyone:<acl>
```

授权一个节点 使他不可以创建子节点

```shell
[zk: localhost:2181(CONNECTED) 1] create /worldtest "worldtest"
Created /worldtest
[zk: localhost:2181(CONNECTED) 2] create /worldtest/test1 "test1"
Created /worldtest/test1
[zk: localhost:2181(CONNECTED) 3] create /worldtest/test2 "test2"
Created /worldtest/test2
[zk: localhost:2181(CONNECTED) 4] getAcl /worldtest
'world,'anyone
: cdrwa
# 以上创建了三个节点

[zk: localhost:2181(CONNECTED) 5] setAcl /worldtest world:anyone:drwa
[zk: localhost:2181(CONNECTED) 6] create /worldtest/test3 "test3"
Authentication is not valid : /worldtest/test3
# 权限不足不能创建
[zk: localhost:2181(CONNECTED) 7] getAcl /worldtest
'world,'anyone
: drwa

```

## IP授权模式

```
setAcl <path> ip:<ip>:<acl>
```

授权一个节点 使它只能读取结点数据

在另一台电脑登陆zookeeper `./zkCli.cmd -server 192.168.1.20`

```shell
# 在本机使用setAcl 设置权限
# 设置权限的时候黑使用逗号同时设置几组规则
[zk: localhost:2181(CONNECTED) 2] create /iptest "iptest"
Created /iptest
[zk: localhost:2181(CONNECTED) 3] create /iptest/test1 "iptest1"
Created /iptest/test1
[zk: localhost:2181(CONNECTED) 4] create /iptest/test2 "iptest2"
Created /iptest/test2
[zk: localhost:2181(CONNECTED) 5] ls -s /
[hadoop, iptest, worldtest, zookeeper]
# 设置192.168.1.2 只能查看
[zk: localhost:2181(CONNECTED) 6] setAcl /iptest ip:192.168.1.2:r
```

在 192.168.1.2 中

```shell
[zk: 192.168.1.20(CONNECTED) 1] create /iptest/test3 "test3"
Authentication is not valid : /iptest/test3
[zk: 192.168.1.20(CONNECTED) 2] delete /iptest/test2
Authentication is not valid : /iptest/test2
[zk: 192.168.1.20(CONNECTED) 3] set /iptest "iptestfrom1.2"
Authentication is not valid : /iptest
[zk: 192.168.1.20(CONNECTED) 4] getAcl /iptest
'ip,'192.168.1.2
: r
[zk: 192.168.1.20(CONNECTED) 5] setAcl /iptest ip:192.168.1.2:crwda
Authentication is not valid : /iptest
```

## auth授权模式

```
addauth digest <user>:<password> #添加认证用户
setAcl <path> auth:<user>:<acl>
```

授权一个节点 使它只能读取结点数据

本机

```shell

[zk: localhost:2181(CONNECTED) 0] ls /
[authtest, hadoop, iptest, worldtest, zookeeper]
# 添加认证用户
[zk: localhost:2181(CONNECTED) 1] addauth digest windows:123456
# 这里如果没有认证，默认无任何权限
[zk: localhost:2181(CONNECTED) 2] setAcl /authtest auth:windows:r,world:anyone:
[zk: localhost:2181(CONNECTED) 3] get /authtest
authtest
[zk: localhost:2181(CONNECTED) 4] get /authtest/test1
authtest1
[zk: localhost:2181(CONNECTED) 5] set /authtest/test1 "test11"
```

另一客户端

```shell
[zk: 192.168.1.20(CONNECTED) 1] ls /
[authtest, hadoop, iptest, worldtest, zookeeper]
[zk: 192.168.1.20(CONNECTED) 2] get /authtest
org.apache.zookeeper.KeeperException$NoAuthException: KeeperErrorCode = NoAuth for /authtest
# 认证之后才可以访问
[zk: 192.168.1.20(CONNECTED) 3] addauth digest windows:123456
[zk: 192.168.1.20(CONNECTED) 4] get /authtest
authtest
[zk: 192.168.1.20(CONNECTED) 5] get /authtest/test1
test11
```

## digest 授权模式

```
setAcl <path> digest:<user>:<password>:<acl>
```

这里的密码是经过SHA1及BASE64处理的密文，在SHELL中可以通过以下命令计算：

```
echo -n <user>:<password> | openssl dgst -binary -sha1 | openssl base64
```

授权一个节点 使它能对节点完全控制，没有授权不能操作

```
gushiyu@gushiyuzookeeper:~$ echo -n windowsdigest:123456 | openssl dgst -binary -sha1 | openssl base64
/D39VFSoTJqM1AT/V4ocbjSHxxI=
# 使用shell计算出编码
```

```powershell
[zk: localhost:2181(CONNECTED) 13] create /digesttest "digest"
Created /digesttest
[zk: localhost:2181(CONNECTED) 14] create /digesttest/test1 "test1"
Created /digesttest/test1
[zk: localhost:2181(CONNECTED) 15] create /digesttest/test2 "test2"
Created /digesttest/test2
# 添加权限
[zk: localhost:2181(CONNECTED) 16] setAcl /digesttest digest:windowsdigest:/D39VFSoTJqM1AT/V4ocbjSHxxI=:crdwa,world:anyone:
[zk: localhost:2181(CONNECTED) 17] get /digesttest
org.apache.zookeeper.KeeperException$NoAuthException: KeeperErrorCode = NoAuth for /digesttest
```

另一主机

```shell
[zk: 192.168.1.20(CONNECTED) 15] get /digesttest
org.apache.zookeeper.KeeperException$NoAuthException: KeeperErrorCode = NoAuth for /digesttest
[zk: 192.168.1.20(CONNECTED) 16] addauth digest windowsdigest:123456
[zk: 192.168.1.20(CONNECTED) 17] get /digesttest
digest
```

## ACL超级管理员

zookeeper的权限管理模式有一种叫做super，该模式提供一个超管可以方便的访问 任何权限的节点 

假设这个超管是：super:admin，需要先为超管生成密码的密文

```
echo -n super:admin | openssl dgst -binary -sha1 | openssl base64
xQJmxLMiHGwaqBvst5y6rkB6HQs=
```

打开zookeeper目录下的/bin/zkServer.sh服务器脚本文件，找到如下一行：

```
nohup $JAVA "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-
Dzookeeper.root.logger=${ZOO_LOG4J_PROP}
```

这就是脚本中启动zookeeper的命令，默认只有以上两个配置项，我们需要加一个 超管的配置项

```
"-
Dzookeeper.DigestAuthenticationProvider.superDigest=super:xQJmxLMiHGwaqBvst5y6rkB6HQs="
```

那么修改以后这条完整命令变成了

```
nohup $JAVA "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-
Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" "-
Dzookeeper.DigestAuthenticationProvider.superDigest=super:xQJmxLMiHGwaqBv
st5y6rkB6HQs="\
-cp "$CLASSPATH" $JVMFLAGS $ZOOMAIN "$ZOOCFG" > "$_ZOO_DAEMON_OUT"
2>&1 < /dev/null &
```

之后在使用过程中使用超级管理员，就需要 使用 addauth 来添加超级管理员

