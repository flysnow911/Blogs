## CNI
容器网络接口。在介绍Flannel的vxlan时，都是介绍容器请求的出口是docker0网桥。   
如果是docker run启动的容器，出口确实是docker0。   
但是k8s创建的容器，出口不是docker0，而是cni。   
下图是基于cni的数据流图，与基于docker0唯一不同的就是把docker0换成了cni0。    
	
[![data_flow_vxlan](https://github.com/flysnow911/Blogs/blob/master/imgs/cnidataflow.png "data_flow_vxlan")](https://github.com/flysnow911/Blogs/blob/master/imgs/cnidataflow.png "data_flow_vxlan")   

