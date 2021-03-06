## Zookeeper介绍

Zookeeper是分布式数据管理与协调框架。分布式应用可以基于Zookeeper实现数据发布订阅、命名服务、Master选举、分布式锁、集群管理等功能。

### Zookeeper中的请求

- **事务请求：** 会改变服务器状态的请求（创建节点、更新数据、删除节点、创建会话等等） 
- **非事务请求：**读取数据但是不对状态进行任何修改的请求。



### Zookeeper中的节点

Zookeeper中的数据节点称为ZNode，ZNode是Zookeeper中数据的最小单元，每个ZNode都可以保存数据，同时还可以挂载子节点。

**节点类型**

- 持久节点：该节点被创建后，就会一直存在于Zookeeper服务器上，直到主动删除这个节点。
- 持久顺序节点：除了持久性外，还保证节点间的顺序性。即每个父节点都会为它的第一级子节点维护子节点创建的先后顺序。在顺序节点的创建时，会自动为该节点名后添加一个数字后缀。
- 临时节点：节点的生命周期与客户端绑定，当客户端的会话失效，节点就自动被清理。临时节点只能作为叶子节点。
- 临时顺序节点：即临时节点+顺序节点。



### Zookeeper中的ZXID

对于每个一事务请求，Zookeeper都会为其分配一个全局唯一的事务Id，用ZXID表示，ZXID是一个64位的数字，其中低32位可以看作是一个简单的单调递增的计数器，针对客户端的每一个事务请求，Leader服务器在产生一个新的事务Proposal的时候，都会对该计数器进行加1操作；而高32位则代表了Leader周期epoch的编号，每当选举产生一个新的Leader服务器，就会从这个Leader服务器上取出其本地日志中最大事务Proposal的ZXID，并从该ZXID中解析出对应的epoch值，然后再对其进行加1操作，之后就会以此编号作为新的epoch，并将低32位置0来开始生成新的ZXID。

从ZXID中就可以看出Zookeeper处理事务的全局顺序。ZAB协议中这一通过epoch编号来区分Leader周期变化的策略，能够有效地避免不同的Leader服务器错误地使用相同的ZXID编号提出不一样的事务Proposal的异常情况。



### Zookeeper中服务器的几种角色

- **Leader**：其主要工作有以下两个：

  - 事务请求的唯一调度和处理者，保证集群事务处理的顺序性。**事务请求指的是会改变服务器状态的请求，即创建节点、更新数据、删除节点、创建会话等请求。**对于事务请求，Leader服务器会根据请求类型创建对应的Proposal提议，并发送给所有的Follower服务器来发起一次集群内的事务投票。

  - 集群内部各服务器的调度者。

    为了保持整个集群内的实时通信，同时为了确保可以控制所有的Follower/Observer服务器，Leader服务器会与每一个Follower/Observer服务器都建立一个TCP长连接。

- **Follower**：其主要工作有以下三个：

  - 处理客户端非事务请求，转发事务请求给Leader服务器。
  - 参与事务请求Proposal的投票。
  - 参与Leader选举投票。

- **Observer**：观察Zookeeper集群中的最新状态变化并将这些状态变更同步过来。其主要工作为：处理客户端非事务请求，转发事务请求给Leader服务器。即Observer通常用在不影响集群事务处理能力的前提下，提升集群的非事务处理能力。**Observer不参与任何形式的投票，包括事务请求Proposal的投票和Leader选举投票。**

### Zookeeper中服务器的几种状态

- **FOLLOWING**：跟随者状态 ，表示当前服务器的角色是Follower。
- **LEADING** ：领导者状态 ，表示当前服务器是Leader。 
- **OBSERVING**：观察者状态，表示当前服务器角色是Observer。
- **LOOKING**：寻找leader状态。当服务器处于该状态时，它会认为当前集群中没有Leader，因此需要进入Leader选举流程。

### Zookeeper中选票的数据结构

- **id**：被推举的Leader的SID。
- **zxid**：被推举的Leader事务ID。
- **electionEpoch**：逻辑时钟，用来判断多个投票是否在同一轮选举周期中，该值在服务端是一个自增序列，每次进入新一轮的投票后，都会对该值进行加1操作。
- **peerEpoch**：被推举的Leader的epoch。
- **state**：当前服务器的状态。

### zookeeper中的ACL (访问控制列表)

Zookeeper中的ACL包括三个方面：权限模式（scheme）、授权对象（ID）和权限（Permission）。通常用**Scheme：id：permission**来标识一个有效的ACL信息。

- Scheme：权限模式用来确定权限验证过程中使用的检验策略。包括四种：
  - IP：
  - Digest（用户名密码方式）：
  - World：数据节点的访问权限对所有用户开放
  - Super：超级用户可以对任意Zookeeper上的数据节点进行任何操作。
- ID：授权对象指的是权限赋予的用户或一个指定实体。不同的权限模式下，授权对象是不同的。
- Permission：Zookeeper中，权限分为5大类：
  - Create：数据节点的创建权限，允许授权对象在该数据节点下创建子节点。
  - Delete：子节点的删除权限，允许授权对象删除该数据节点下的子节点。
  - Read：数据节点的读取权限，允许授权对象访问该数据节点并读取其数据内容或子节点列表。
  - Write：数据节点的更新权限，允许授权对象对该数据节点进行更新操作。（删除该节点？）
  - Admin：数据节点的管理权限，允许授权对象对该数据节点进行ACL相关的设置操作。

ACL的设置

1. 创建节点的同时设置该节点的ACL
2. 使用setAcl命令单独对已存在的数据节点进行设置。

## ZAB协议

ZAB协议是为Zookeeper专门设计的一种支持崩溃恢复的原子广播协议。在Zookeeper中，主要依赖ZAB协议来实现分布式数据的一致性。基于该协议，ZooKeeper 实现了一种主从模式的系统架构来保持集群中各个副本之间的数据一致性。具体的，Zookeeper使用一个单一主进程来接收并处理客户端的所有事务请求，并采用ZAB的原子广播协议，将服务器数据的状态变更以事务Proposal的形式广播到所有的副本进程上。

ZAB协议包括两种基本模式：崩溃恢复、消息广播。

### 消息广播

当集群中已经有过半的follower与leader服务器完成了状态同步，那么整个zk集群就可以进入消息广播模式了。当一台同样遵守ZAB协议的服务器启动后加入到集群中时，如果此时集群中已经存在一个Leader服务器在负责消息广播，那么新加入的服务器就会自觉的进入数据恢复模式：找到Leader所在的服务器，并与其进行数据同步，然后一起参与到消息广播流程中。

ZAB协议的消息广播过程使用的是一个原子广播协议，类似于一个2PC提交过程，针对每个客户端的事务请求，leader服务器会为其生成对应的事务Proposal，并将其发送给集群中其余所有的机器，然后再分别收集各自的选票，最后进行事务提交。
1. Leader 接收到消息请求后，将消息赋予一个全局唯一的 64 位自增 id，叫做：zxid，通过 zxid 的大小比较即可实现因果有序这一特性。
2. Leader 通过先进先出队列（会给每个follower都创建一个队列，保证发送的顺序性）（通过 TCP 协议来实现，以此实现了全局有序这一特性）将带有 zxid 的消息作为一个提案（proposal）分发给所有 follower。
3. 当 follower 接收到 proposal，先将 proposal 写到本地事务日志，写事务成功后再向 leader 回一个 ACK。
4. 当 leader 接收到过半的 ACKs 后，leader 就向所有 follower 发送 COMMIT 命令通知其进行事务提交。同时会在本地执行该消息。
5. 当 follower 收到消息的 COMMIT 命令时，就会执行该消息，完成对事务的提交。

相比于完整的二阶段提交，Zab 协议最大的区别就是**移除了中断逻辑**，follower 要么回 ACK 给 leader，要么抛弃 leader，在某一时刻，leader 的状态与 follower 的状态很可能不一致，因此它不能处理 leader 挂掉的情况，所以 Zab 协议引入了恢复模式来处理这一问题。从另一角度看，正因为 Zab 的广播过程不需要终止事务，也就是说不需要所有 follower 都返回 ACK 才能进行 COMMIT，而是只需要合法数量（2f+1 台服务器中的 f+1 台） 的follower，也提升了整体的性能。不能正常反馈的就节点就抛弃leader，然后进入数据同步阶段，和集群达成一致。







### 崩溃恢复

ZAB协议会让ZK集群进入崩溃恢复模式的情况如下：
1. 当服务框架在启动过程中
2. 当Leader服务器出现网络中断，崩溃退出与重启等异常情况。
3. 当集群中已经不存在过半的服务器与Leader服务器保持正常通信。

Leader选举需要达到的再次使用的条件，需要解决以下两个问题：

1. 已经被leader提交的事务需要最终被所有的机器提交（已经发出commit了）。这一情况会出现在以下场景：当 leader 收到合法数量 follower 的 ACKs 后，就向各个 follower 广播 COMMIT 命令，同时也会在本地执行 COMMIT 并向连接的客户端返回「成功」。但是如果在各个 follower 在收到 COMMIT 命令前 leader 就挂了，导致剩下的服务器并没有执行都这条消息。

2. 保证丢弃那些只在leader上提出的事务。（只在leader上提出了proposal，还没有收到回应，还没有进行提交） 。这一情况会出现在以下场景：当 leader 接收到消息请求生成 proposal 后就挂了，其他 follower 并没有收到此 proposal，因此经过恢复模式重新选了 leader 后，这条消息是被跳过的。 此时，之前挂了的 leader 重新启动并注册成了 follower，他保留了被跳过消息的 proposal 状态，与整个系统的状态是不一致的，需要将其删除。


针对这些要求，如果让Leader选举算法能够保证新选举出来的Leader服务器拥有集群中所有机器最高编号（即ZXID最大）的事务Proposal，那么就可以保证这个新选举出来的Leader一定具有所有已经提交的提案。更为重要的是，如果让具有最高编号事务Proposal的机器来成为Leader，就可以省去Leader服务器检查Proposal的提案和丢弃工作了。

在该模式中，会进行新Leader的选举和数据状态的同步。当选举产生了新的Leader并且集群中过半的集器与该Leader状态同步成功之后，ZAB协议就会退出崩溃恢复模式。

#### Leader选举过程

Leader选举是保证分布式数据一致性的关键所在。当Zookeeper集群中的一台服务器出现以下两种情况之一时，需要进入Leader选举。

1. 服务器初始化启动。
2. 服务器运行期间无法和Leader保持连接。

##### 服务器启动时的Leader选举

若进行Leader选举，则至少需要两台机器。这里以三台机器为例，选举过程如下：

1. **自增选举轮次**。Zookeeper规定所有有效的投票都必须在同一轮次中，在开始新一轮投票时，会首先对logicalclock进行自增操作。

2. **初始化选票**。在开始进行新一轮投票之前，每个服务器都会初始化自身的选票，并且在初始化阶段，每台服务器都会将自己推举为Leader。每次投票会包含所推举的服务器的myid和ZXID，使用(myid, ZXID)来表示，此时Server1的投票为(1, 0)，Server2的投票为(2, 0)，然后各自将这个投票发给集群中其他机器。

3. **发送初始化选票**。完成选票的初始化后，服务器就会发起第一次投票。Zookeeper会将刚刚初始化好的选票放入sendqueue中，由发送器WorkerSender负责发送出去。

4. **接受来自各个服务器的投票**：集群的每个服务器收到投票后，首先判断该投票的有效性，如检查是否是本轮投票、是否来自LOOKING状态的服务器。

5. **判断选举轮次**。在发送完初始化选票之后，接着开始处理外部投票。在处理外部投票时，会根据选举轮次来进行不同的处理。

   - **外部投票的选举轮次大于内部投票**。若服务器自身的选举轮次落后于该外部投票对应服务器的选举轮次，那么就会立即更新自己的选举轮次(logicalclock)，并且清空所有已经收到的投票，然后使用初始化的投票来进行PK以确定是否变更内部投票。最终再将内部投票发送出去。
   - **外部投票的选举轮次小于内部投票**。若服务器接收的外选票的选举轮次落后于自身的选举轮次，那么Zookeeper就会直接忽略该外部投票，不做任何处理，并返回步骤4。
   - **外部投票的选举轮次等于内部投票**。此时可以开始进行选票PK。

6. **选票PK**：针对每一个投票，服务器都需要将**别人的投票和自己的投票**进行PK，PK规则如下：

   - **检查选举轮次**：如果收到的投票的选举轮次大于自己投票的选举轮次，那么就需要进行投票变更。
   - **优先检查ZXID**：ZXID比较大的服务器优先作为Leader。因为ZXID越大，越能保证数据的恢复。
   - **如果ZXID相同，那么就比较myid**：myid较大的服务器作为Leader服务器。

   对于Server1而言，它的投票是(1, 0)，接收到的Server2的投票为(2, 0)，首先会比较两者的ZXID，均为0，再比较myid，此时Server2的myid最大，于是更新自己的投票为(2, 0)，然后重新将投票发出去，对于Server2而言，其无须更新自己的投票，只是再次向集群中所有机器发出上一次投票信息即可。

7. **统计投票**：每次投票后，服务器都会统计投票信息。如果一台机器收到了超过半数的相同的投票，那么这个投票对应的myid就是Leader。**过半是指大于或等于（n / 2+1）**。对于Server1、Server2而言，都统计出集群中已经有两台机器接受了(2, 0)的投票信息，此时便认为已经选出了Leader。

8. **改变服务器状态**：一旦确定了Leader，每个服务器就会更新自己的状态，如果是Follower，那么就变更为FOLLOWING，如果是Leader，就变更为LEADING。这个步骤不是立即的，而是会等待一段时间，默认200ms，来确定是否有更优的投票。

##### 服务器运行时的Leader选举

在Zookeeper运行期间，Leader与非Leader服务器各司其职，**即便当有非Leader服务器宕机或新加入，此时也不会影响Leader，但是一旦Leader服务器挂了，那么整个集群将暂停对外服务，进入新一轮Leader选举**。

假设正在运行的有Server1、Server2、Server3三台服务器，当前Leader是Server2，若某一时刻Leader挂了，此时便开始Leader选举。选举过程如下：

1. **变更状态**：Leader挂后，余下的非Observer服务器都会讲自己的服务器状态变更为LOOKING，然后开始进入Leader选举过程。
2. **自增选举轮次**。Zookeeper规定所有有效的投票都必须在同一轮次中，在开始新一轮投票时，会首先对logicalclock进行自增操作。
3. **初始化选票**。在开始进行新一轮投票之前，每个服务器都会初始化自身的选票，并且在初始化阶段，每台服务器都会将自己推举为Leader。
4. **发送初始化选票**：在这个过程中，需要生成投票信息（myid，ZXID）。因为是运行期间，每个服务器上的ZXID可能不同，此时假定Server1的ZXID为123，Server3的ZXID为122；**在第一轮投票中，由于还无法检测到集群中其他机器的状态信息，因此每台机器都是先将自己作为被推举的对象来进行投票。**即Server1和Server3都会投自己，产生投票(1, 123)，(3, 122)，然后各自将投票发送给集群中所有机器。
5. **接收来自各个服务器的投票**：与启动时过程相同。
6. **处理投票**：与启动时过程相同，此时，Server1将会成为Leader。
7. **统计投票**：与启动时过程相同。
8. **改变服务器的状态**：与启动时过程相同。

#### 数据同步

当集群完成Leader选举之后，Learner会向Leader服务器进行注册。在注册的最后阶段，Learner服务器会发送给Leader服务器一个ACKEPOCH数据包，Leader会从这个数据包中解析出该Learner的currentEpoch和lastZxid。

首先，leader会从zookeeper的内存数据库中提取出事务请求对应的提议缓存队列：proposals，同时完成对以下三个ZXID值的初始化。

- peerLastZxid:该learner服务器最后处理的ZXID。
- minCommittedLog:leader服务器提议缓存队列committedLog中的最小ZXID。
- maxCommittedLog:leader服务器提议缓存队列committedLog中的最大ZXID。


数据同步通常分为四类，分别是直接差异化同步（DIFF同步），先回滚在差异化同步（TRUNC+DIFF同步），仅回滚同步（TRUNC同步）和全量同步（SNAP同步）。会根据leader和learner服务器间的数据差异情况来决定最终的数据同步方式。

##### 直接差异化同步
场景：peerLastZxid介于minCommittedLog和maxCommittedLog间。 
leader首先会向这个learner发送一个DIFF指令，用于通知“learner即将把一些proposal同步给自己”。实际同步过程中，针对每个proposal，leader都会通过发送两个数据包来完成，分别是PROPOSAL内容数据包和COMMIT指令数据包——这和zookeeper运行时leader和follower间的事务请求的提交过程是一致的。 
Learner服务器接收到一个DIFF指令后，便进入DIFF同步阶段。之后Learner依次将收到的数据包同步到内存数据库中。紧接着，Learner还会收到来自Leader的NEWLEADER指令，此时Learner就会反馈给Leader一个ACK指令，表明自己确实完成了对提议缓存队列中Proposal的同步。

举例，某时刻leader的提议缓存队列对应的ZXID依次是： 
0x500000001,0x500000002,0x500000003,0x500000004,0x500000005 
而learner最后处理的ZXID为0x500000003，于是leader依次将0x500000004和0x500000005两个提议同步给learner。

##### 先回滚在差异化同步
场景：A,B,C三台机器，某一时刻B是leader，此时leader_epoch为5，同时当前已被集群大部分机器都提交的ZXID包括：0x500000001,0x500000002。此时leader正处理ZXID：0x500000003,并且已经将事务写入到了leader本地的事务日志中去——就在leader恰好要将该proposal发给其他follower进行投票时，leader挂了，proposal没被同步出去。此时集群进行新一轮leader选举，假设此次选的leader为A，leader_epoch变更为6，之后A和C又提交了0x600000001,0x600000002两个事务。此时B再次启动并开始数据同步。 
简单讲，上面场景就是leader在已经将事务记录到本地事务日志中，但没有成功发起proposal流程时就挂了。 
当leader发现某个learner包含一条自己没的事务记录，就让该learner进行事务回滚——回滚到leader上存在的，最接近peerLastZxid的ZXID，上面例子中leader会让learner回滚到ZXID为0x500000002的事务记录。

##### 仅回滚同步
场景：peerLastZxid大于maxCommittedLog 
这种场景就是上述先回滚再差异化同步的简化模式，leader会要求learner回滚到ZXID值为maxCommitedLog对应的事务操作。

##### 全量同步（SNAP同步）
场景1：peerLastZxid小于minCommittedLog 
场景2：leader上没有提议缓存队列，peerLastZxid不等于lastProcessedZxid（leader服务器数据恢复后得到的最大ZXID）

这两种场景下，只能进行全量同步。leader首先向learner发送一个SNAP指令，通知learner进行全量同步，随后leader会从内存数据库中获取到全量的数据节点和会话超时时间记录器，将它们序列化后传输给learner，learner接收到后对其反序列化后再入内存数据库中。

##### 收尾阶段
leader在完成完差异数据后，就会将该learner加入到forwardingFollowers或observingLearners队列中，这俩队列在运行期间的事务请求处理过程中会使用到。随后leader发送一个NEWLEADER指令，用于通知learner已经将提议缓存队列中的proposal都同步给自己了。 
当learner收到leader的NEWLEADER指令后会反馈给leader一个ack消息，表明自己完成了对提议缓存队列中proposal的同步。

leader收到来自learner的ack后，进入“过半策略”等待阶段，知道集群中有过半的learner机器响应了leader这个ack消息。 
一旦满足“过半策略”后，leader会向所有已完成数据同步的learner发送一个UPTODATE指令，用来通知learner已完成数据同步，同时集群已有过半机器完成同步，集群已具有对外服务的能力了。

learner在收到leader的UPTODATE指令后，会终止数据同步流程，然后向leader再反馈一个ACK消息。



### 常见问题



#### 为什么要求 可用节点数量 > 集群总结点数量 / 2 

可用节点数量 > 总节点数量/2  。注意 是 > , 不是 ≥。

如果不这样限制，在集群出现脑裂的时候，可能会出现多个子集群同时服务的情况（即子集群各组选举出自己的leader）， 这样对整个zookeeper集群来说是紊乱的。

换句话说，如果遵守上述规则进行选举，即使出现脑裂，集群最多也只能回出现一个子集群可以提供服务的情况（能满足节点数量> 总结点数量/2 的子集群最多只会有一个）。

#### 为什么采用奇数个节点

1、防止由脑裂造成的集群不可用。

首先，什么是脑裂？集群的脑裂通常是发生在节点之间通信不可达的情况下，集群会分裂成不同的小集群，小集群各自选出自己的master节点，导致原有的集群出现多个master节点的情况，这就是脑裂。

下面举例说一下为什么采用奇数台节点，就可以防止由于脑裂造成的服务不可用：

(1) 假如zookeeper集群有 5 个节点，发生了脑裂，脑裂成了A、B两个小集群： 

     (a) A ： 1个节点 ，B ：4个节点 ， 或 A、B互换

     (b) A ： 2个节点， B ：3个节点  ， 或 A、B互换

    可以看出，上面这两种情况下，A、B中总会有一个小集群满足 可用节点数量 > 总节点数量/2 。所以zookeeper集群仍然能够选举出leader ， 仍然能对外提供服务，只不过是有一部分节点失效了而已。

(2) 假如zookeeper集群有4个节点，同样发生脑裂，脑裂成了A、B两个小集群：

    (a) A：1个节点 ，  B：3个节点，   或 A、B互换 

    (b) A：2个节点 ， B：2个节点

    可以看出，情况(a) 是满足选举条件的，与（1）中的例子相同。 但是情况(b) 就不同了，因为A和B都是2个节点，都不满足 可用节点数量 > 总节点数量/2 的选举条件， 所以此时zookeeper就彻底不能提供服务了。

综合上面两个例子可以看出： 在节点数量是奇数个的情况下， zookeeper集群总能对外提供服务（即使损失了一部分节点）；如果节点数量是偶数个，会存在zookeeper集群不能用的可能性（脑裂成两个均等的子集群的时候）。

在生产环境中，如果zookeeper集群不能提供服务，那将是致命的 ， 所以zookeeper集群的节点数一般采用奇数个。

2、在容错能力相同的情况下，奇数台更节省资源。

举两个例子：

(1) 假如zookeeper集群1 ，有3个节点，3/2=1.5 ,  即zookeeper想要正常对外提供服务（即leader选举成功），至少需要2个节点是正常的。换句话说，3个节点的zookeeper集群，允许有一个节点宕机。

(2) 假如zookeeper集群2，有4个节点，4/2=2 , 即zookeeper想要正常对外提供服务（即leader选举成功），至少需要3个节点是正常的。换句话说，4个节点的zookeeper集群，也允许有一个节点宕机。

那么问题就来了， 集群1与集群2都有 允许1个节点宕机 的容错能力，但是集群2比集群1多了1个节点。在相同容错能力的情况下，本着节约资源的原则，zookeeper集群的节点数维持奇数个更好一些。








## Zookeeper的典型应用场景

Zookeeper是一个典型的发布/订阅模式的分布式数据管理与协调框架。基于对ZAB算法的实现，该框架能够很好的保证分布式环境中数据的一致性。

Zookeeper的典型应用场景主要包括：**数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master选举、分布式锁、分布式队列等。**

### 1. 数据发布与订阅（配置中心）

顾名思义就是发布者将数据发布到ZK的节点上，订阅者对数据进行订阅，进而达到动态获取数据的目的，实现配置信息的集中式管理和动态更新。例如全局的配置信息，服务式服务框架的服务地址列表等就非常适合使用。

发布/订阅一般有两种设计模式：推模式和拉模式，服务端主动将数据更新发送给所有订阅的客户端称为推模式；客户端主动请求获取最新数据称为拉模式，Zookeeper采用了推拉相结合的模式，客户端向服务端注册自己需要关注的节点，一旦该节点数据发生变更，那么服务端就会向相应的客户端推送Watcher事件通知，客户端接收到此通知后，主动到服务端获取最新的数据。

实现方式：

1. 配置存储：在Zookeeper中创建一个持久节点，并将初始化配置存储到该节点。
2. 配置获取：客户端节点在启动初始化阶段，首先会从创建的配置节点上读取数据库的配置信息。
3. 注册监听：客户端节点在创建的配置节点上注册一个数据变更的Watcher监听
4. 配置变更：借助Zookeeper的Watcher机制，当配置信息发生变更时，Zookeeper会将数据变更的通知发送到各个客户端。每个客户端在收到这个变更通知后，就可以重新进行最新数据的获取。

### 2. 命名服务（全局唯一ID生成）

利用Zookeeper中顺序节点的特性，在一个指定的节点下面创建一个顺序节点，并获取完整的子节点名。

### 3. 分布式锁服务

分布式锁是控制分布式系统中同步访问共享资源的一种方式。分布式锁通常用来保证分布式系统的一致性。

#### 排他锁的获取与释放

1. 获取排他锁，所有客户端都会试图**在同一个节点下（如/exclusive_lock）创建一个指定名称的临时子节点**。Zookeeper会保证在所有的客户端中，最终只有一个客户端能创建成功，那就可以认为该客户端获取了锁。同时，没有获取到锁的客户端会对/exclusive_lock节点注册一个子节点变更的Watcher监听。**会有惊群效应。**
2. 释放排他锁，由于创建的子节点类型为临时节点，所以会在两种情况下释放锁：
   - 当前获取锁的客户端发生宕机，客户端被动断开与Zookeeper的连接，该临时子节点就会被移除。
   - 当前获取锁的客户端正常执行完业务逻辑，客户端主动断开与Zookeeper的连接，该临时子节点就会被移除。

#### 共享锁的获取与释放

1. 获取共享锁：所有客户端都会在同一个节点下（如/shared_lock）创建一个临时顺序节点，如果是读请求，就创建/shared_lock/192.168.1.1-R的节点，如果是写请求，就创建/shared_lock/192.168.1.1-W的节点。

2. 创建完节点后，获取/shared_lock节点下的所有子节点的列表。

3. 如果无法获取共享锁，那么就调用exist()来对比自己小的那个节点注册Wahtcher。比自己小的节点是指：读请求向比自己序号小的最后一个写请求节点注册Watcher监听。写请求向比自己序号小的最后一个节点注册Watcher监听。

4. 等待Watcher通知，继续进入步骤2。

   







## ZAB协议与Paxos算法的异同

ZAB协议并不是 Paxos 算法的一个典型实现，在讲解 ZAB和 Paxos之间的区别之前, 我们首先看下两者的联系， 

1. 两者都有一个类似于Leader进程的角色，由其负责协调多个Follower运行 
2. Leader进程都会等待超过半数的Follower做出正确的反馈后，才会将一个提案进行提交。 
3. 在ZAB协议中，每个Proposal都包含了一个epoch值，用来代表当前的Leader 周期，在Paxos算法中，同样存在这样的一个标识，只是名字变成了Ballot。
   
   

在Paxos算法中，一个新选举产生的主进程会进行两个阶段的工作，第一阶段被称为读阶段，在这个阶段中，这个新的主进程会通过和所有其他进程进行通信的方式来收集上一个—个主进程提出的提案，并将它们提交.第二阶段被称为写阶段，在这个阶段，与前主进程开始提出自己的提案。

在Paxos算法设计的基础上， ZAB协议额外添加了一 个同步阶段。在同步阶段之前，ZAB协议也存在一个和Paxos算法中的读阶段I类似的过程，被称为发现（ Discovery)阶段，在同步阶段中，新的 Leader会确存在过半数的Follower已经提交了之前Leader周期中的所有事物的Proposal，在这一同步阶段的引人，能够有效地保证Leader在新的周期中提出事务Proposal之前，所有的进程都已经完成了对之前所有事物的Proposal的提交。 

一旦完成同步阶段后，那么 ZAB就会执行和 Paxos算法类似 的写阶段。 

总的来汫， ZAB协议和 Paxos本质区别在于，两者的设计目标不太一样。 ZAB 协议主要用于构建一个高可用的分布式数椐主备系统，例如ZooKeeper, 而Paxos算法 则是用于构建一个分布式的一致性状态机系统，
