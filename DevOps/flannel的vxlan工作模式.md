VXLAN模式
Virtual Extensible Lan 虚拟可扩展局域网，linux本身就支持的一种网络虚拟技术。
-->VXLAN完全在内核态实现上述封装和解封装的工作，与upd模式类似，构建Overlay Network，工作在一个三层网络的二层overlay network。
-->发送端，数据从容器出来后，用户态进入内核，直到从网卡发出去，处理内核态，而udp模式，需要3次切换。  
需要VTEP设备来实现，取得目标主机地址，目标vtep mac地址。 

如下图container-1中数据经过进入docker0, fannel.1, 经过vxlan的两层封包，发数据。   
	
	[![vxlan数据流](https://github.com/flysnow911/Blogs/blob/master/imgs/vxlandataflow.png" “容器发出数据格式")](https://github.com/flysnow911/Blogs/blob/master/imgs/vxlandataflow.png" "vxlan数据流")      

1.docker0在路由一番，发现数据需要发送给flannel.1。参考ip route命令结果。
	出容器的数据包结构
	<img width="150" height="150" src="https://github.com/flysnow911/Blogs/blob/master/imgs/containerdata.png"/>
	[![容器发出数据格式](https://github.com/flysnow911/Blogs/blob/master/imgs/containerdata.png "容器发出数据格式")](https://github.com/flysnow911/Blogs/blob/master/imgs/containerdata.png "容器发出数据格式")      
	
2.flannel通过*目的容器ip地址*，查询到*目的VTEP mac地址*
	二层封包结果：
	[![二层封包数据](https://github.com/flysnow911/Blogs/blob/master/imgs/vtep.png "二层封包数据")](https://github.com/flysnow911/Blogs/blob/master/imgs/vtep.png "二层封包数据")   
	
3.fannel.1中维护了VTEP mac地址，可以通过vtep mac路由查到*目的主机ip*。此时数据加上目的主机ip地址。
4.这一步与vtep没关系了。就是依据目标ip地址，路由到网卡。   

从网卡发出去的数据包如下图所示： 
	[![网络数据](https://github.com/flysnow911/Blogs/blob/master/imgs/vxlan_data_format.png "网络数据")](https://github.com/flysnow911/Blogs/blob/master/imgs/vxlan_data_format.png "网络数据")   
	
以上是发送数据的过程，接收流程正好相反的过程。

以下是我k8s的环境，数据，供流程演绎参考。
[root@master ~]# ip route //对应步骤1.
default via 192.168.2.1 dev ens33 proto static metric 100 
10.244.0.0/24 dev cni0 proto kernel scope link src 10.244.0.1 
10.244.1.0/24 via 10.244.1.0 dev flannel.1 onlink 
10.244.2.0/24 via 10.244.2.0 dev flannel.1 onlink 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 
192.168.2.0/24 dev ens33 proto kernel scope link src 192.168.2.200 metric 100 


[root@master ~]# ip neigh show dev flannel.1 //对应步骤2.
10.244.2.0 lladdr 86:22:66:41:77:3d PERMANENT
10.244.1.0 lladdr 92:1b:d8:d1:19:96 PERMANENT

[root@master ~]# bridge fdb show flannel.1  //对应步骤3.
01:00:5e:00:00:01 dev ens33 self permanent
33:33:00:00:00:01 dev ens33 self permanent
33:33:ff:9e:56:25 dev ens33 self permanent
01:00:5e:00:00:01 dev ens34 self permanent
33:33:00:00:00:01 dev ens34 self permanent
33:33:00:00:00:01 dev docker0 self permanent
01:00:5e:00:00:01 dev docker0 self permanent
33:33:ff:08:59:b7 dev docker0 self permanent
02:42:70:08:59:b7 dev docker0 vlan 1 master docker0 permanent
02:42:70:08:59:b7 dev docker0 master docker0 permanent
ca:59:7d:c2:5e:0f dev flannel.1 dst 192.168.2.202 self permanent
86:22:66:41:77:3d dev flannel.1 dst 192.168.2.202 self permanent
92:1b:d8:d1:19:96 dev flannel.1 dst 192.168.2.201 self permanent
f2:21:6b:d8:af:b0 dev flannel.1 dst 192.168.2.201 self permanent
33:33:00:00:00:01 dev cni0 self permanent
01:00:5e:00:00:01 dev cni0 self permanent
33:33:ff:f7:3e:9c dev cni0 self permanent
86:61:8c:f7:3e:9c dev cni0 master cni0 permanent
86:61:8c:f7:3e:9c dev cni0 vlan 1 master cni0 permanent
a6:7d:24:57:30:ef dev veth375d278 master docker0 permanent
a6:7d:24:57:30:ef dev veth375d278 vlan 1 master docker0 permanent
33:33:00:00:00:01 dev veth375d278 self permanent
01:00:5e:00:00:01 dev veth375d278 self permanent
33:33:ff:57:30:ef dev veth375d278 self permanent
a6:c7:13:10:52:8a dev veth8ea02941 vlan 1 master cni0 permanent
7a:74:56:6b:40:e8 dev veth8ea02941 master cni0 
a6:c7:13:10:52:8a dev veth8ea02941 master cni0 permanent
33:33:00:00:00:01 dev veth8ea02941 self permanent
01:00:5e:00:00:01 dev veth8ea02941 self permanent
33:33:ff:10:52:8a dev veth8ea02941 self permanent
5a:37:b4:50:54:92 dev vethfcef72c2 master cni0 permanent
06:aa:68:89:d6:17 dev vethfcef72c2 master cni0 
5a:37:b4:50:54:92 dev vethfcef72c2 vlan 1 master cni0 permanent
33:33:00:00:00:01 dev vethfcef72c2 self permanent
01:00:5e:00:00:01 dev vethfcef72c2 self permanent
33:33:ff:50:54:92 dev vethfcef72c2 self permanent

[root@master ~]# ip route //对应步骤4.
default via 192.168.2.1 dev ens33 proto static metric 100 
10.244.0.0/24 dev cni0 proto kernel scope link src 10.244.0.1 
10.244.1.0/24 via 10.244.1.0 dev flannel.1 onlink 
10.244.2.0/24 via 10.244.2.0 dev flannel.1 onlink 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 
192.168.2.0/24 dev ens33 proto kernel scope link src 192.168.2.200 metric 100 
