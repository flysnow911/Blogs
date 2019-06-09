zookeeper代码下载后怎么debug呢？嘿嘿，本文就是介绍如何debug 并讲解zk的主流程。

# 调试zookeeper源码  
### 首先找入口
#####  zookeeper启动服务的命令是啥？ 
  ./zkServer.sh start
那就从**zkServer.sh**中下手!  很容易就找到入口类，QuorumPeerMain.java。
找到QuorumPeerMain.main方法， 从代码中可以看出，main方法中做的第一件事就是加载配置文件，zookeeper配置文件**zoo.cfg**.
所以直接在idea中把zoo.cfg的路径写入启动参数，就能正常跑起来。

# zookeeper的主流程
如时序图所示，从main方法进入调用.initializeAndRun（），就进入启动流程 。这里只介绍流程，详细人可以  参考时序图与源码。
1. 校验启动参数 
2. 启动清理日志和镜像文件的任务 
3. 加载配置文件
4. 依靠配置文件初始化QuorumPeer(管理竞选协议)
5. 接下来就是主流程
	- 	  loadDataBase()：data/version-2/中加载相关文件, 镜像和日志路径中反序列化数据DataTree中，
			返回最大事务ID，事务所属时代，当前时代，接收的时代涉及到的文件有:acceptedEpoch,currentEpoch,
			log.400000002,snapshot.3003aa11
	- 	  startServerCnxnFactory：启动连接工厂，初始化连接池，这个连接主要用来监听客户端请求。
	- 	  adminServer.start： JettyAdminServer,启动后台管理，接收commands应用。
	- 	  startLeaderElection：开始竞选。
	- 	  run(): 内旋，依据state四种状态，决定循环什么操作。

[![zk启动主流程](https://github.com/flysnow911/Blogs/blob/master/imgs/zookeeper%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B.png "zk启动主流程")](https://github.com/flysnow911/Blogs/blob/master/imgs/zookeeper%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B.png "zk启动主流程")
