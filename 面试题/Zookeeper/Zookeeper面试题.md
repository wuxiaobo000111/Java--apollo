# Zookeeper面试题

转载于:[https://www.cnblogs.com/lanqiu5ge/p/9405601.html](https://www.cnblogs.com/lanqiu5ge/p/9405601.html)
<a name="C4zhh"></a>
# Zookeeper是什么
ZooKeeper 是一个开放源码的分布式协调服务，它是集群的管理者，监视着集群中各个节点的状态根据节点提交的反馈进行下一步合理操作。最终，将简单易用的接口和性能高效、功能稳定的系统提供给用户。分布式应用程序可以基于 Zookeeper 实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。<br />

<a name="fn3Ae"></a>
# Zookeeper特性

1. 顺序一致性(有序性): 从同一个客户端发起的事务请求，最终将会严格地按照其发起顺序被应用到 Zookeeper 中去。有序性是 Zookeeper 中非常重要的一个特性。
  - 所有的更新都是全局有序的，每个更新都有一个唯一的时间戳，这个时间戳称为zxid(Zookeeper Transaction Id)。
  - 而读请求只会相对于更新有序，也就是读请求的返回结果中会带有这个 Zookeeper 最新的 zxid 。
2. 原子性: 所有事务请求的处理结果在整个集群中所有机器上的应用情况是一致的，即整个集群要么都成功应用了某个事务，要么都没有应用。
2. 单一视图: 无论客户端连接的是哪个 Zookeeper 服务器，其看到的服务端数据模型都是一致的。
2. 可靠性: 一旦服务端成功地应用了一个事务，并完成对客户端的响应，那么该事务所引起的服务端状态变更将会一直被保留，除非有另一个事务对其进行了变更。
2. 实时性: Zookeeper 保证在一定的时间段内，客户端最终一定能够从服务端上读取到最新的数据状态。

客户端的读请求可以被集群中的任意一台机器处理，**如果读请求在节点上注册了监听器，这个监听器也是由所连接的zookeeper机器来处理**。对于写请求，这些请求会同时发给其他zookeeper机器并且达成一致后，请求才会返回成功。因此，随着zookeeper的集群机器增多，读请求的吞吐会提高但是写请求的吞吐会下降。

      有序性是zookeeper中非常重要的一个特性，所有的更新都是全局有序的，每个更新都有一个唯一的时间戳，这个时间戳称为zxid（Zookeeper Transaction Id）。而读请求只会相对于更新有序，也就是读请求的返回结果中会带有这个zookeeper最新的zxid。

<a name="qNw7b"></a>
# Zookeeper提供了什么

1. 文件系统
1. 通知机制

<a name="qEhFR"></a>
# Zookeeper文件系统
Zookeeper提供一个多层级的节点命名空间（节点称为znode）。与文件系统不同的是，这些节点都可以设置关联的数据，而文件系统中只有文件节点可以存放数据而目录节点不行。

Zookeeper为了保证高吞吐和低延迟，在内存中维护了这个树状的目录结构，**这种特性使得Zookeeper不能用于存放大量的数据，每个节点的存放数据上限为1M**。

<a name="gj30I"></a>
# ZAB协议
ZAB协议是为分布式协调服务Zookeeper专门设计的一种支持崩溃恢复的原子广播协议。ZAB协议包括两种基本的模式：崩溃恢复和消息广播。<br />当整个zookeeper集群刚刚启动或者Leader服务器宕机、重启或者网络故障导致不存在过半的服务器与Leader服务器保持正常通信时，所有进程（服务器）进入崩溃恢复模式，首先选举产生新的Leader服务器，然后集群中Follower服务器开始与新的Leader服务器进行数据同步，当集群中超过半数机器与该Leader服务器完成数据同步之后，退出恢复模式进入消息广播模式，Leader服务器开始接收客户端的事务请求生成事物提案来进行事务请求处理。

<a name="aL5i6"></a>
## 服务器角色
<a name="leader"></a>
### Leader

- 事务请求的唯一调度和处理者，保证集群事务处理的顺序性
- 集群内部各服务的调度者

[]()
<a name="follower"></a>
### Follower

- 处理客户端的非事务请求，转发事务请求给Leader服务器
- 参与事务请求Proposal的投票
- 参与Leader选举投票

[]()
<a name="observer"></a>
### Observer
3.3.0版本以后引入的一个服务器角色，在不影响集群事务处理能力的基础上提升集群的非事务处理能力

- 处理客户端的非事务请求，转发事务请求给Leader服务器
- 不参与任何形式的投票

<a name="kYqCp"></a>
## Server的工作状态
服务器具有四种状态，分别是LOOKING、FOLLOWING、LEADING、OBSERVING。

- LOOKING：寻找Leader状态。当服务器处于该状态时，它会认为当前集群中没有Leader，因此需要进入Leader选举状态。
- FOLLOWING：跟随者状态。表明当前服务器角色是Follower。
- LEADING：领导者状态。表明当前服务器角色是Leader。
- OBSERVING：观察者状态。表明当前服务器角色是Observer。

<a name="KmnXd"></a>
## Leader选举
Leader选举是保证分布式数据一致性的关键所在。当Zookeeper集群中的一台服务器出现以下两种情况之一时，需要进入Leader选举。<br />　　(1) 服务器初始化启动。<br />　　(2) 服务器运行期间无法和Leader保持连接。

<a name="8PkB4"></a>
### 服务器启动时期的Leader选举
      若进行Leader选举，则至少需要两台机器，这里选取3台机器组成的服务器集群为例。在集群初始化阶段，当有一台服务器Server1启动时，其单独无法进行和完成Leader选举，当第二台服务器Server2启动时，此时两台机器可以相互通信，每台机器都试图找到Leader，于是进入Leader选举过程。选举过程如下：<br />　    (1) 每个Server发出一个投票。由于是初始情况，Server1和Server2都会将自己作为Leader服务器来进行投票，每次投票会包含所推举的服务器的myid和ZXID，使用(myid, ZXID)来表示，此时Server1的投票为(1, 0)，Server2的投票为(2, 0)，然后各自将这个投票发给集群中其他机器。<br />　　(2) 接受来自各个服务器的投票。集群的每个服务器收到投票后，首先判断该投票的有效性，如检查是否是本轮投票、是否来自LOOKING状态的服务器。<br />　　(3) 处理投票。针对每一个投票，服务器都需要将别人的投票和自己的投票进行PK，PK规则如下<br />　　　　· 优先检查ZXID。ZXID比较大的服务器优先作为Leader。<br />　　　　· 如果ZXID相同，那么就比较myid。myid较大的服务器作为Leader服务器。<br />　　        对于Server1而言，它的投票是(1, 0)，接收Server2的投票为(2, 0)，首先会比较两者的ZXID，均为0，再比较myid，此时Server2的myid最大，于是更新自己的投票为(2, 0)，然后重新投票，对于Server2而言，其无须更新自己的投票，只是再次向集群中所有机器发出上一次投票信息即可。<br />　　(4) 统计投票。每次投票后，服务器都会统计投票信息，判断是否已经有过半机器接受到相同的投票信息，对于Server1、Server2而言，都统计出集群中已经有两台机器接受了(2, 0)的投票信息，此时便认为已经选出了Leader。<br />　　(5) 改变服务器状态。一旦确定了Leader，每个服务器就会更新自己的状态，如果是Follower，那么就变更为FOLLOWING，如果是Leader，就变更为LEADING。

<a name="moFJW"></a>
### 服务器运行时期的Leader选举
在Zookeeper运行期间，Leader与非Leader服务器各司其职，即便当有非Leader服务器宕机或新加入，此时也不会影响Leader，但是一旦Leader服务器挂了，那么整个集群将暂停对外服务，进入新一轮Leader选举，其过程和启动时期的Leader选举过程基本一致。假设正在运行的有Server1、Server2、Server3三台服务器，当前Leader是Server2，若某一时刻Leader挂了，此时便开始Leader选举。选举过程如下<br />　　(1) 变更状态。Leader挂后，余下的非Observer服务器都会讲自己的服务器状态变更为LOOKING，然后开始进入Leader选举过程。<br />　　(2) 每个Server会发出一个投票。在运行期间，每个服务器上的ZXID可能不同，此时假定Server1的ZXID为123，Server3的ZXID为122；在第一轮投票中，Server1和Server3都会投自己，产生投票(1, 123)，(3, 122)，然后各自将投票发送给集群中所有机器。<br />　　(3) 接收来自各个服务器的投票。与启动时过程相同。<br />　　(4) 处理投票。与启动时过程相同，此时，Server1将会成为Leader。<br />　　(5) 统计投票。与启动时过程相同。<br />　　(6) 改变服务器的状态。与启动时过程相同。

<a name="IHRWL"></a>
#### Leader选举算法分析
在3.4.0后的Zookeeper的版本只保留了TCP版本的FastLeaderElection选举算法。当一台机器进入Leader选举时，当前集群可能会处于以下两种状态<br />　　　　· 集群中已经存在Leader。<br />　　　　· 集群中不存在Leader。<br />　　对于集群中已经存在Leader而言，此种情况一般都是某台机器启动得较晚，在其启动之前，集群已经在正常工作，对这种情况，该机器试图去选举Leader时，会被告知当前服务器的Leader信息，对于该机器而言，仅仅需要和Leader机器建立起连接，并进行状态同步即可。而在集群中不存在Leader情况下则会相对复杂，其步骤如下<br />　　(1) 第一次投票。无论哪种导致进行Leader选举，集群的所有机器都处于试图选举出一个Leader的状态，即LOOKING状态，LOOKING机器会向所有其他机器发送消息，该消息称为投票。投票中包含了SID（服务器的唯一标识）和ZXID（事务ID），(SID, ZXID)形式来标识一次投票信息。假定Zookeeper由5台机器组成，SID分别为1、2、3、4、5，ZXID分别为9、9、9、8、8，并且此时SID为2的机器是Leader机器，某一时刻，1、2所在机器出现故障，因此集群开始进行Leader选举。在第一次投票时，每台机器都会将自己作为投票对象，于是SID为3、4、5的机器投票情况分别为(3, 9)，(4, 8)， (5, 8)。<br />　　(2) 变更投票。每台机器发出投票后，也会收到其他机器的投票，每台机器会根据一定规则来处理收到的其他机器的投票，并以此来决定是否需要变更自己的投票，这个规则也是整个Leader选举算法的核心所在，其中术语描述如下<br />　　　　· vote_sid：接收到的投票中所推举Leader服务器的SID。<br />　　　　· vote_zxid：接收到的投票中所推举Leader服务器的ZXID。<br />　　　　· self_sid：当前服务器自己的SID。<br />　　　　· self_zxid：当前服务器自己的ZXID。<br />　　每次对收到的投票的处理，都是对(vote_sid, vote_zxid)和(self_sid, self_zxid)对比的过程。<br />　　　　规则一：如果vote_zxid大于self_zxid，就认可当前收到的投票，并再次将该投票发送出去。<br />　　　　规则二：如果vote_zxid小于self_zxid，那么坚持自己的投票，不做任何变更。<br />　　　　规则三：如果vote_zxid等于self_zxid，那么就对比两者的SID，如果vote_sid大于self_sid，那么就认可当前收到的投票，并再次将该投票发送出去。<br />　　　　规则四：如果vote_zxid等于self_zxid，并且vote_sid小于self_sid，那么坚持自己的投票，不做任何变更。<br />　　结合上面规则，给出下面的集群变更过程。<br />　　(3) 确定Leader。经过第二轮投票后，集群中的每台机器都会再次接收到其他机器的投票，然后开始统计投票，如果一台机器收到了超过半数的相同投票，那么这个投票对应的SID机器即为Leader。此时Server3将成为Leader。<br />　　由上面规则可知，通常那台服务器上的数据越新（ZXID会越大），其成为Leader的可能性越大，也就越能够保证数据的恢复。如果ZXID相同，则SID越大机会越大。

<a name="KX4fv"></a>
# 四种数据节点

1. PERSISTENT-持久节点: 除非手动删除，否则节点一直存在于Zookeeper上
1. EPHEMERAL-临时节点: 临时节点的生命周期与客户端会话绑定，一旦客户端会话失效（客户端与zookeeper连接断开不一定会话失效），那么这个客户端创建的所有临时节点都会被移除。
1. PERSISTENT_SEQUENTIAL-持久顺序节点: 基本特性同持久节点，只是增加了顺序属性，节点名后边会追加一个由父节点维护的自增整型数字。
1. EPHEMERAL_SEQUENTIAL-临时顺序节点: 基本特性同临时节点，增加了顺序属性，节点名后边会追加一个由父节点维护的自增整型数字。

<a name="8JWE8"></a>
# Watch机制
Zookeeper允许客户端向服务端的某个Znode注册一个Watcher监听，当服务端的一些指定事件触发了这个Watcher，服务端会向指定客户端发送一个事件通知来实现分布式的通知功能，然后客户端根据Watcher通知状态和事件类型做出业务上的改变。

<a name="0EtGn"></a>
## 工作机制

- 客户端注册watcher
- 服务端处理watcher
- 客户端回调watcher

<a name="wq1sJ"></a>
## Watcher特性总结：

1. 一次性<br />    无论是服务端还是客户端，一旦一个Watcher被触发，Zookeeper都会将其从相应的存储中移除。这样的设计有效的减轻了服务端的压力，不然对于更新非常频繁的节点，服务端会不断的向客户端发送事件通知，无论对于网络还是服务端的压力都非常大。
1. 客户端串行执行<br />    客户端Watcher回调的过程是一个串行同步的过程。
1. 轻量
  1. Watcher通知非常简单，只会告诉客户端发生了事件，而不会说明事件的具体内容。
  1. 客户端向服务端注册Watcher的时候，并不会把客户端真实的Watcher对象实体传递到服务端，仅仅是在客户端请求中使用boolean类型属性进行了标记。
4. watcher event异步发送watcher的通知事件从server发送到client是异步的，这就存在一个问题，不同的客户端和服务器之间通过socket进行通信，由于网络延迟或其他因素导致客户端在不通的时刻监听到事件，由于Zookeeper本身提供了ordering guarantee，即客户端监听事件后，才会感知它所监视znode发生了变化。所以我们使用Zookeeper不能期望能够监控到节点每次的变化。Zookeeper只能保证最终的一致性，而无法保证强一致性。
4. 注册watcher getData、exists、getChildren
4. 触发watcher create、delete、setData
4. 当一个客户端连接到一个新的服务器上时，watch将会被以任意会话事件触发。当与一个服务器失去连接的时候，是无法接收到watch的。而当client重新连接时，如果需要的话，所有先前注册过的watch，都会被重新注册。通常这是完全透明的。只有在一个特殊情况下，watch可能会丢失：对于一个未创建的znode的exist watch，如果在客户端断开连接期间被创建了，并且随后在客户端连接上之前又删除了，这种情况下，这个watch事件可能会被丢失。

<a name="XzBkm"></a>
# 客户端Watch注册

1. 调用getData()/getChildren()/exist()三个API，传入Watcher对象
1. 标记请求request，封装Watcher到WatchRegistration
1. 封装成Packet对象，发服务端发送request
1. 收到服务端响应后，将Watcher注册到ZKWatcherManager中进行管理
1. 请求返回，完成注册。

<a name="ZR3l5"></a>
# 服务端处理Wathch

1. 服务端接收Watcher并存储: 接收到客户端请求，处理请求判断是否需要注册Watcher，需要的话将数据节点的节点路径和ServerCnxn（ServerCnxn代表一个客户端和服务端的连接，实现了Watcher的process接口，此时可以看成一个Watcher对象）存储在WatcherManager的WatchTable和watch2Paths中去。
1. Watcher触发
  1. 以服务端接收到 setData() 事务请求触发NodeDataChanged事件为例：
    - 封装WatchedEvent<br />将通知状态（SyncConnected）、事件类型（NodeDataChanged）以及节点路径封装成一个WatchedEvent对象
    - 查询Watcher<br />从WatchTable中根据节点路径查找Watcher
    - 没找到；说明没有客户端在该数据节点上注册过Watcher
    - 找到；提取并从WatchTable和Watch2Paths中删除对应Watcher（**从这里可以看出Watcher在服务端是一次性的，触发一次就失效了**）
3. 调用process方法来触发Watcher: 这里process主要就是通过ServerCnxn对应的TCP连接发送Watcher事件通知。

<a name="rrzLd"></a>
# ACL权限控制机制

<a name="3almj"></a>
## UGO（User/Group/Others）
目前在Linux/Unix文件系统中使用，也是使用最广泛的权限控制方式。是一种粗粒度的文件系统权限控制模式。<br />[]()
<a name="1ccgh"></a>
## ACL（Access Control List）访问控制列表
包括三个方面：

  - 权限模式（Scheme）
    - IP：从IP地址粒度进行权限控制
    - Digest：最常用，用类似于 username:password 的权限标识来进行权限配置，便于区分不同应用来进行权限控制
    - World：最开放的权限控制方式，是一种特殊的digest模式，只有一个权限标识“world:anyone”
    - Super：超级用户
  - 授权对象
    - 授权对象指的是权限赋予的用户或一个指定实体，例如IP地址或是机器灯。
  - 权限 Permission
    - CREATE：数据节点创建权限，允许授权对象在该Znode下创建子节点
    - DELETE：子节点删除权限，允许授权对象删除该数据节点的子节点
    - READ：数据节点的读取权限，允许授权对象访问该数据节点并读取其数据内容或子节点列表等
    - WRITE：数据节点更新权限，允许授权对象对该数据节点进行更新操作
    - ADMIN：数据节点管理权限，允许授权对象对该数据节点进行ACL相关设置操

<a name="OTk60"></a>
# Zookeeper如何保证事务的一致性
zookeeper采用了全局递增的事务Id来标识，所有的proposal（提议）都在被提出的时候加上了zxid，zxid实际上是一个64位的数字，高32位是epoch（时期; 纪元; 世; 新时代）用来标识leader周期，如果有新的leader产生出来，epoch会自增，低32位用来递增计数。当新产生proposal的时候，会依据数据库的两阶段过程，首先会向其他的server发出事务执行请求，如果超过半数的机器都能执行并且能够成功，那么就会开始执行。

<a name="HmXBC"></a>
# Zookeeper对Watch是永久的吗
不是。官方声明：一个Watch事件是一个一次性的触发器，当被设置了Watch的数据发生了改变的时候，则服务器将这个改变发送给设置了Watch的客户端，以便通知它们。<br />为什么不是永久的，举个例子，如果服务端变动频繁，而监听的客户端很多情况下，每次变动都要通知到所有的客户端，给网络和服务器造成很大压力。一般是客户端执行getData(“/节点A”,true)，如果节点A发生了变更或删除，客户端会得到它的watch事件，但是在之后节点A又发生了变更，而客户端又没有设置watch事件，就不再给客户端发送。在实际应用中，很多情况下，我们的客户端不需要知道服务端的每一次变动，我只要最新的数据即可。

<a name="jHXkW"></a>
# Zookeeper命令
这里就不给出了，有单独的一篇文章介绍。

<a name="rRi4U"></a>
# Zookeeper的应用场景
Zookeeper是一个典型的发布/订阅模式的分布式数据管理与协调框架，开发人员可以使用它来进行分布式数据的发布和订阅。<br />通过对Zookeeper中丰富的数据节点进行交叉使用，配合Watcher事件通知机制，可以非常方便的构建一系列分布式应用中年都会涉及的核心功能，如：

  - 数据发布/订阅
  - 负载均衡
  - 命名服务
  - 分布式协调/通知
  - 集群管理
  - Master选举
  - 分布式锁
  - 分布式队列

