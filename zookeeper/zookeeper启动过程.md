zookeeper代码下载后怎么debug呢？嘿嘿，本文就是介绍如何debug 并讲解zk的主流程。

# 调试zookeeper源码  
### 首先找入口
#####  zookeeper启动服务的命令是啥？ 
###### zkServer.sh start
那就从**zkServer.sh**中下手!  很容易就找到入口类，QuorumPeerMain.java。

找到QuorumPeerMain.main方法， 从代码中可以看出，main方法中做的第一件事就是加载配置文件，zookeeper配置文件**zoo.cfg**.所以直接在idea中把zoo.cfg的路径写入启动参数，就能正常跑起来。

# zookeeper的主流程
