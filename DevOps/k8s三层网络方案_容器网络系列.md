# k8s的三层网络方案
## flannel host-gw
把宿主机做为网关，与其它主机通信。由flannel维护容器与主机的路由规则,将每个flannel子网的“下一跳”，设置成宿主机IP地址。   
	[![](https://github.com/flysnow911/Blogs/blob/master/imgs/flannel_host_gw.png)](https://github.com/flysnow911/Blogs/blob/master/imgs/flannel_host_gw.png)
在cni中，通过目的容器有ip，在路由规则中匹配到目的主机ip，出口设备。   
如下所示。通往10.233.1.0子网的数据包，目标主机地址是10.168.0.3，依赖设备eth0。   

		$ ip route
		...
		10.244.1.0/24 via 10.168.0.3 dev eth0
	
主机的ip信息，flannel子网信息都存在etcd中，flannel能够watch到变化。

### 优势：
    相对于vxlan，host-gw的优势是减少了额外的封包，拆包的过程。性能只损失了约10%，而vxlan的模式，性能损失在20%-30%左右。
### 要求：
	flannel host-gw要求主机之间能进行二层同信。


## calico
###   基本原理
与flannel host-gw原理一样，也是配置容器子网与宿主机的路由规则。
		[![](https://github.com/flysnow911/Blogs/blob/master/imgs/calico_bgp.png)](https://github.com/flysnow911/Blogs/blob/master/imgs/calico_bgp.png)
		< 目的容器 IP 地址段 > via < 网关的 IP 地址 > dev eth0

但是calico维护路由规则不依赖etcd，而是BGP。
	
cali的cni是veth pair模式，数据包由caliXXXX出容器，经过路由规则，找到目标主机ip与目标容器子网段。再经过主机网卡出去。路由由进程Felix维护。路由规则通过BGP Client使用BGP协议传输。BGP协议消息，伪代码如下：
		
		[BGP 消息]
		我是宿主机 192.168.1.2
		10.233.2.0/24 网段的容器都在我这里
		这些容器的下一跳地址是我
### calico缺陷：
calico的路由信息是实时发现的。BGP client之间一直维护了着通信连接，那么N个结点的网络之间有N²的连接数，这给集群网络的压力是比较大的。所以当集群规模到达100的时候，就需要使用Route Reflector模式。

###  Route Reflector模式
  在部分结点中跑BGP Client，由这些结点之间相互通信，学习路由规则，由其它结点与BGP Client交换由规则，以得到整个集群的规则。
  
###  Calico IPIP 模式
  当集群中主机之前二层不能通信，就要用ipip模式。
  如下图,ipip模式数据流图。
  [![](https://github.com/flysnow911/Blogs/blob/master/imgs/calico_ipip.png)](https://github.com/flysnow911/Blogs/blob/master/imgs/calico_ipip.png)
  
  Node1中，会有以下路由
  
  	10.233.2.0/24 via 192.168.2.2 tunl0
  发向子网10.233.2.0/24子网的信息，需要发给ip是192.168.2.2的主机，依赖设备T-U—N-L-0。注意区分flannel的tun0。
  calico的TUNL0设备是IP隧道设备，数据进入TUNL0后，会被额外封装一层,把目标ip地址加入数据包。这样，整个包就被伪装成N1发送给N2的包。   
  
  [![](https://github.com/flysnow911/Blogs/blob/master/imgs/calico_ipip_payload.png)](https://github.com/flysnow911/Blogs/blob/master/imgs/calico_ipip_payload.png)
###   IPIP劣势
     1.额外的一次封包拆包，性能损失与vxlan差不多。所以建议适合主机在同一网段的场景使用。   
     2.在公有云的情况下，宿主机之间的网关是不允许用户干涉，所以这不能使用。
## 总结
	1.三层网络方案要维护的路由规则比较多，故障概率高，排查困难。
	2.在公有云，用更简单的flannel host ws模式。
	3.私有云，更推荐calico，因为calico能够覆盖更多的场景。
  
  
  
  
	
