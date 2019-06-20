## apollo集成spring的基础知识
	实现ApplicationContextInitializer,覆写initialize()方法，能在spring容器初始化的时期，嵌入自己的业务。
	spring读取配置信息，装入PropertiesSource的实例中，在bean的初始化阶段把，把配置信息赋值给bean。

在以上两点的基础上，开发apollo的客户端。
1. 创建ApolloApplicationContextinitializer，通过rest接口访问apollocConfig的服务，取得需要的所有配置，然后写入PropertySources,同时设置为最高优先级的propertySource。
2. 那么在spring初始化bean时，就利用propertySource里的配置信息赋值给bean。

以下是时序图。
[![apollo集成spring启动图](https://github.com/flysnow911/Blogs/blob/master/imgs/apollo%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B.png "apollo集成spring启动图")](https://github.com/flysnow911/Blogs/blob/master/imgs/apollo%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B.png "apollo集成spring启动图")


原理可参考
https://github.com/ctripcorp/apollo/wiki/Apollo%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E8%AE%BE%E8%AE%A1#31-%E5%92%8Cspring%E9%9B%86%E6%88%90%E7%9A%84%E5%8E%9F%E7%90%86
