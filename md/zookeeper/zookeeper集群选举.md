# zookeeper的集群选举



## 服务器状态 

服务器状态代表服务器此时的状态

**looking**：寻找leader状态。当服务器处于该状态时，它会认为当前集群中没有 leader，因此需要进入leader选举状态。 

**leading**： 领导者状态。表明当前服务器角色是leader。 

**following**： 跟随者状态。表明当前服务器角色是follower。 

**observing**：观察者状态。表明当前服务器角色是observer。

## 启动时的leader选举

在集群初始化阶段，当有一台服务器server1启动时，其单独无法进行和完成 leader选举，当第二台服务器server2启动时，此时两台机器可以相互通信，每台机器都 试图找到leader，于是进入leader选举过程。选举过程如下:

1.  每个server发出一个投票。由于是初始情况，server1和server2都会将自己作为 leader服务器来进行投票，每次投票会包含所推举的服务器的myid和zxid，使用 (myid, zxid)来表示，此时server1的投票为(1, 0)，server2的投票为(2, 0)，然后各自 将这个投票发给集群中其他机器。 
2.  集群中的每台服务器接收来自集群中各个服务器的投票。 
3.  处理投票。针对每一个投票，服务器都需要将别人的投票和自己的投票进行pk，pk 规则如下 优先检查zxid。zxid比较大的服务器优先作为leader。 如果zxid相同，那么就比较myid。myid较大的服务器作为leader服务器。 对于Server1而言，它的投票是(1, 0)，接收Server2的投票为(2, 0)，首先会比较 两者的zxid，均为0，再比较myid，此时server2的myid最大，于是更新自己的投票 为(2, 0)，然后重新投票，对于server2而言，其无须更新自己的投票，只是再次向集 群中所有机器发出上一次投票信息即可。 
4.   统计投票。每次投票后，服务器都会统计投票信息，判断是否已经有过半机器接受到 相同的投票信息，对于server1、server2而言，都统计出集群中已经有两台机器接受 了(2, 0)的投票信息，此时便认为已经选出了leader 
5.  改变服务器状态。一旦确定了leader，每个服务器就会更新自己的状态，如果是 follower，那么就变更为following，如果是leader，就变更为leading。

由于启动时ZXID都相同，所以启动时的选举服务器ID大的成为leader（两台服务器启动就会进行选举）

## 服务器运行时期的Leader选举

在zookeeper运行期间，leader与非leader服务器各司其职，即便当有非leader 服务器宕机或新加入，此时也不会影响leader，但是一旦leader服务器挂了，那么整个集 群将暂停对外服务，进入新一轮leader选举，其过程和启动时期的Leader选举过程基本 一致。

 假设正在运行的有server1、server2、server3三台服务器，当前leader是 server2，若某一时刻leader挂了，此时便开始Leader选举。选举过程如下:



1.  变更状态。leader挂后，余下的服务器都会将自己的服务器状态变更为looking，然 后开始进入leader选举过程。 
2.   每个server会发出一个投票。在运行期间，每个服务器上的zxid可能不同，此时假定 server1的zxid为122，server3的zxid为122，在第一轮投票中，server1和server3 都会投自己，产生投票(1, 122)，(3, 122)，然后各自将投票发送给集群中所有机器。 
3.  接收来自各个服务器的投票。与启动时过程相同 
4.  处理投票。与启动时过程相同，此时，server3将会成为leader。 
5.  统计投票。与启动时过程相同。 
6.  改变服务器的状态。与启动时过程相同。

在发生leader宕机时，会根据ZXID大的选举出Leader,如若相同，服务器编号大的成为Leader



