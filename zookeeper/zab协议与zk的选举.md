# Paxos算法与Zab协议
zk是基于paxos的分布式应用协调框架。但他的选举算法用的是基于paxos改良的zab（ZK Atomic Broadcasting）协议。
**zab协议**是一种分布式系统中支持崩溃恢复的一致性协议。基于这个协议，zk实现主从架构系统中各副本数据一致性。
### 数据同步
在运行时，所有写操作都发给leader执行，完成后leader将操作写入日志，并同步给所有follower节点。
###  选主
没有leader或leader奔溃时，参与竞选的节点通过zab协议，能够快速选出leader。

### 选举的时机
1. 启动阶段
2. leader奔溃
3. 多数节点异常（脑裂）

### 选举的过程
从选票的数据结构说起。Vote.java 包含以下重要属性：
		id -- proposedLeader 选票选谁做leader
		zxid --proposedZxid	 选票所处zxid(事务id)，这个只有在leader的每次写操作成功后，才会+1.
		peerEpoch -- proposedEpoch 选票所处epoch（每次选举，都会开启一个新时代）
	
节点选举过程的三状态：
**LOOKING**: leader选举期
**FOLLOWING**: follower和leader保持同步时期
**LEADING**: leader服务器进入领导状态

基于选票的数据结构，选举的过程可以描述为：
1. 参选的节点处于LOOKING状态，相互发送选票，选出自己想选择的leader(首轮选自己)。
2. 节点在接收到选票后，都会通过一定规则pk，pk的目的是决定坚持自己的选择，还是接受收到选票的选择。把pk结果再广播出去，直到一个节点pk屡次胜出。
3. 当发现有节点得到超过一半节点的选择或者节点收到所有其他节点的票时，该节点就退出选举，把状态变为leadding或following，同时把自己的状态广播出去。这一步3.4.0版本有变化。
	##### 3.4.0之前
	选举类是AuthFastLeaderElection.java，line907-918，
当节点收到所有竞选节点的选票时，则判断自己的选择是否正确，更新自己选票结果，同时广播选票，修改状态，设置相应状态(leadding或following)，退出选举的自旋
如果没有收到所有竞选节点选票，就把已收到的选票轮询一遍，看是存在某个节点得到超过一半节点得支持，若存在，设置相应状态(leadding或following)。退出选举的自旋，否则自旋。
	##### 3.4.0之后
	选举类是FastleaderElection.java中,line：1025行开始当节点收到所有竞选节点的选票时，再次判断自己的选票是否正确，判断方法是通过轮询recvqueue中的选票，是否有比当前持有选票更优的选票，如果没有，则退出选举，设置相应状态(leadding或following)，退出选举的自旋。否则不改变状态，仍然为looking状态，再次由QuorumPeer调用FastleaderElection.lookForLeader方法，选出leader。仔细看line:1025-1049。  
			
			
		`if (voteSet.hasAllQuorums()) { //是否获得所有竞选节点选票
			// Verify if there is any change in the proposed leader
			//进入自旋, 自上次从队列取出数据时间T到T+finalizeWait时刻，队列中无数据的情况下，超时退出自旋。
		     while((n = recvqueue.poll(finalizeWait,
					TimeUnit.MILLISECONDS)) != null){ 
				if(totalOrderPredicate(n.leader, n.zxid, n.peerEpoch,
						proposedLeader, proposedZxid, proposedEpoch)){
					recvqueue.put(n);
					break;
				}
			}
		     /*
		     * This predicate is true once we don't read any new
		     * relevant message from the reception queue
		     */
		     if (n == null) { //不需要改变自己的选票
			setPeerState(proposedLeader, voteSet);
			Vote endVote = new Vote(proposedLeader,
				proposedZxid, logicalclock.get(), 
				proposedEpoch);
			leaveInstance(endVote);
			return endVote;
		     }
		 }
		break;`
FastleaderElection的算法有个好处，减少了节点超半数的判断，需要拿到全部选票，才做判断。这样的好处是更严谨。收敛速度不一定快。   
**最重要的变化是**：前者通信用的是**UDP协议（WorkerReceiver中用的是DatagramSocket）**，后者是**TCP协议(Socket)**，更稳定。
##### pk的规则   
依次对比三个属性epoch,zxid,sid，出现不相等的情况，大者立刻胜出。
源码：

	/*
         * We return true if one of the following three cases hold:
         * 1- New epoch is higher
         * 2- New epoch is the same as current epoch, but new zxid is higher
         * 3- New epoch is the same as current epoch, new zxid is the same
         *  as current zxid, but server id is higher.
         */

        return ((newEpoch > curEpoch) ||
                ((newEpoch == curEpoch) &&
                ((newZxid > curZxid) || ((newZxid == curZxid) && (newId > curId)))));
##### FLE演算示意图
[![FLE选举演算示意图](https://github.com/flysnow911/Blogs/blob/master/imgs/FLE%E9%80%89%E4%B8%BE%E6%BC%94%E7%AE%97%E5%9B%BE.png "FLE选举演算示意图")](https://github.com/flysnow911/Blogs/blob/master/imgs/FLE%E9%80%89%E4%B8%BE%E6%BC%94%E7%AE%97%E5%9B%BE.png "FLE选举演算示意图")   

从上图可以看出，选举的步骤如下：
1. 启动阶段,按照协议，大家都给自己投票，选自己为leader。并且广播自己的选票。
2. T1时刻，S1收到S2的选票，S1 PK失败，修改自己的选票，选择S2为leader,广播新选票。
3. T2时刻，S1收到S3的选票，S1 PK失败，修改自己的选票，选择S3为leader.然后发现自己已经获取所有竞选节点的选票，所以在T2阶段进入自旋阶段，等待剩下选票，看是否需要修改自己选票。
4. T5+${finalizeWait}时刻，S1自旋超时，发现不需要修改自己的选票，所以退出自旋，同时把QuorumPeer的状态设为Following，退出选举。 
5. S2从初始阶段到T3之间，与S1 PK胜出，不需要修改选票。
6. S2直到T3时刻,与S3 PK失败，修改自己的选票，选择S3为leader.   
7. S2发现自己已经获取所有竞选节点的选票，在T3阶段进入自旋阶段，等待剩下选票，看是否需要修改自己选票.
8. S2在T7+${finalizeWait}时刻，自旋超时，发现不需要修改自己的选票，所以退出自旋，同时把QuorumPeer的状态设为Following，退出选举。  
8. S3从初始阶段到T4之间，与S1,S2 PK胜出，不需要修改选票。  
9. S3在T4时刻,已经获取所有竞选节点的选票，在T4阶段进入自旋阶段，等待剩下选票，看是否需要修改自己选票。     
10. S3在T6+${finalizeWait}时刻，自旋超时，发现不需要修改自己的选票，所以退出自旋，同时把QuorumPeer的状态设为Leading，退出选举。

## 选举过程中异常问题的处理
异常问题的处理能看出一个系统是否健壮，同时也是一个优秀系统的必备功能。
#### 1). 选举过程中，新的Server加入
当一个Server启动时它都会发起一次选举，此时由选举线程发起相关流程，那么每个Server都会获得当前zxid最大的那个server，如果本次最大的Server没有获得n/2+1 个票数，那么下一次投票时，他将向zxid最大的Server投票，重复以上流程，最后一定能选举出一个Leader。
#### 2). 选举过程中，Server退出  
只要保证n/2+1个Server存活zk service仍可以正确工作，
如果少于n/2+1个Server 存活就没办法选出Leader。
#### 3). 选举过程中或选举完成后，Leader死亡  
当选举出Leader以后，其他server已经处于FLLOWING 状态，本次选主过程正常进行。
当选主完成后，所有的Fllower都会向Leader发送Ping消息，如果无法ping通，就改变自己的状为(FLLOWING ==> LOOKING)，发起新的一轮选举。
#### 4). 双主问题  
当Follower无法ping通Leader时就认为Leader已经出问题开始重新选举，Leader收到Follower的ping没有达到半数以上则要退出Leader重新选举。


