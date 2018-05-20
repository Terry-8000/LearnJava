# Zookeeper选举原理

作为一个分布式应用程序协调服务，在大型网站中，其本身也是集群部署的，安装zookeeper的时候最好是单数节点，因为要选举。Zookeeper的leader节点是集群工作的核心，用来更新并保证leader和server具有相同的系统状态，Follower服务器是Leader的跟随者，用于接收客户端的请求并向客户端返回结果，在选举过程中参与投票。对于客户端来说，每个zookeeper都是一样的。

zookeeper提供了三种选择策略：

- LeaderElection
- AuthFastLeaderElection
- FastLeaderElection

这里仅介绍默认的算法：FastLeaderElection。

### 基础概念

- Sid：服务器id；
- Zxid：服务器的事务id，数据越新，zxid越大；
- epoch：逻辑时钟，在服务端是一个自增序列，每次进入下一轮投票后，就会加1；
- server状态：
  - Looking（选举状态）
  - Leading（领导者状态，表明当前server是leader）
  - Following（跟随者状态，表明当前server是Follower）
  - Observing（观察者状态、表明当前server是Observer）。

### 选举步骤

当系统启动或者leader崩溃后，就会开始leader的选举。

1. 状态变更。服务器启动的时候每个server的状态时Looking，如果是leader挂掉后进入选举，那么余下的非Observer的Server就会将自己的服务器状态变更为Looking，然后开始进入Leader的选举状态；

2. 发起投票。每个server会产生一个（sid，zxid）的投票，系统初始化的时候zxid都是0，如果是运行期间，每个server的zxid可能都不同，这取决于最后一次更新的数据。将投票发送给集群中的所有机器；

3. 接收并检查投票。server收到投票后，会先检查是否是本轮投票，是否来自looking状态的server；

4. 处理投票。对自己的投票和接收到的投票进行PK：

   - 先检查zxid，较大的优先为leader；
   - 如果zxid一样，sid较大的为leader；

      根据PK结果更新自己的投票，在次发送自己的投票；

5. 统计投票。每次投票后，服务器统计投票信息，如果有过半机器接收到相同的投票，那么leader产生，如果否，那么进行下一轮投票；

6. 改变server状态。一旦确定了Leader，server会更新自己的状态为Following或者是Leading。选举结束。

**补充说明：**

1. 在步骤2发送投票的时候，投票的信息除了**sid**和**zxid**，还有：

    - **electionEpoch**：逻辑时钟，用来判断多个投票是否在同一轮选举周期中，该值在服务端是一个自增序列，每次进入新一轮的投票后，都会对该值进行加1操作。
    - **peerEpoch**：被推举的Leader的epoch。
    - **state**：当前服务器的状态。

2. 为了能够相互投票，每两台服务器之间都会建立网络连接，为避免重复建立TCP连接，zk的server只允许sid大于自己的服务器与自己建立连接，否则断开当前连接，并主动和对方建立连接。



> 参考：
>
> https://www.cnblogs.com/felixzh/p/5869212.html
>
> https://www.cnblogs.com/leesf456/p/6107600.html
>
> https://www.cnblogs.com/ASPNET2008/p/6421571.html



