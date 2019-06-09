# zookeeper不保证读到最新数据
zookeeper虽然zk是高速，强一致性，属于cp模型，但并不保证读到最新数据。   
zk毕竟是分布式的，leader写成功后，最新数据在同步给follower,observer的过程中   
需要时间，并且如果遇到网络延时，导致短时间的不一致，所以并不保证每次读到最新数据，   
只不过zk读多写少，所以我们平时用的时候并不关注这个问题。
在zk的开发者文档也作出了解释：
https://zookeeper.apache.org/doc/r3.1.2/zookeeperProgrammers.html#ch_zkGuarantees   
在Consistency Guarantees章节中，原文如下：

		`Sometimes developers mistakenly assume one other guarantee that ZooKeeper does not in fact make. This is:   

		Simultaneously Conistent Cross-Client Views
		ZooKeeper does not guarantee that at every instance in time, two different clients will have identical views of ZooKeeper data. Due to factors like network delays, one client may perform an update before another client gets notified of the change. Consider the scenario of two clients, A and B. If client A sets the value of a znode /a from 0 to 1, then tells client B to read /a, client B may read the old value of 0, depending on which server it is connected to. If it is important that Client A and Client B read the same value, Client B should should call the sync() method from the ZooKeeper API method before it performs its read.   

		So, ZooKeeper by itself doesn't guarantee that changes occur synchronously across all servers, but ZooKeeper primitives can be used to construct higher level functions that provide useful client synchronization. (For more information, see the ZooKeeper Recipes. [tbd:..]).`
