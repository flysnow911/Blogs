网上对于apollo与其他配置中心对比，并且选择apollo的帖子太多了，这里不赘述。
由于修改过apollo的源码，并且部门也全面应用了apollo,所以对apollo了解比较深入。
这里就说说从更深入的角度，聊聊apollo.

## 高容错,高可用
1. namespace没有配置没关系，会帮你默认加载application下的配置。
2. apollo服务宕机没关系(哪怕所有服务都宕机了，也没关系)，apollo的客户端早就帮应用缓存了一份配置在本地，并且一直是动态更新的。利用springcloud全家桶，解决了微服务化的apollo所有架构需求。
3. 减少了很多外部依赖，如etcd,zk。
4. 服务都是无状态的。
## 高性能
4C8G虚拟机，能支撑10000个连接。每个客户端一个长连接，一个configService服务可以满足一万个app节点需求。
	
	
	原理：巧妙地实现了长连接。利用spring DeferredResult实现异步化，维持一个长连接。
		一般实现长连接的方式：
		1. 	WatchDog机制，定时任务。客户端向Server发送包，告知保持连接。
		2. 	客户端向Server发送包，告知保持连接。
		3. 	服务器端需要一个检测机制，在一定时间(receiveTimeDelay)内，如果没有检测到客户端的请求，就断开连接。
		
## 配置同步延迟低
	client和server通过保持长连接，维护一套推拉模式，争取在最短的时间内保持配置的一致性。
	client与server保持长连接，单开一个线程池，里面只有一个线程，定时轮询是否有更新，这是主动拉的过程。
	推送过程：提供配置查询的接口是DeferredResult。
## 在保持幂等的前提下，设计尽量简单
	推拉模式的情况下，只有拉的情况下，是读配置。apollo推送的模式，是不推送配置的，他只是把最新发布编号推给应用，应用拿到编号后与本地对比，
	如果不一致，则说明有配置更新。调用主动查询配置的接口拉配置。这样设计的好处是考虑幂等。好处是：
	1.如果有两次连续的发布配置推送消息，消息里有配置，由于网络原因，前一次的修改在后面到达客户端，那么客户端就出错了。
	2.在保持幂等的情况下，设计尽量简单，这样的话，只有拉配置的逻辑里会有更新配置，重复的逻辑不会出现多次，查错，维护起来更方便，出错概率也低。
	
## 开源，社区活跃
	由于开源，我们加入加入或完善了不少功能，如生产配置的脱敏，配置对比，跨版本回滚，审计功能，批量导入等功能。