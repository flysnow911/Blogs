### mac vmware k8s node安装
#### 1. centos7 mini install.
#### 2. 配置桥接模式。
#### 3. 配置网卡。
	[root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens33 
		TYPE=Ethernet
		PROXY_METHOD=none
		BROWSER_ONLY=no
		BOOTPROTO=static
		DEFROUTE=yes
		IPV4_FAILURE_FATAL=yes
		IPV6INIT=no
		IPV6_AUTOCONF=yes
		IPV6_DEFROUTE=yes
		IPV6_FAILURE_FATAL=no
		IPV6_ADDR_GEN_MODE=stable-privacy
		NAME=ens33
		UUID=316da4dd-98be-4972-b641-8b9bedfc7786
		DEVICE=ens33
		ONBOOT=yes
		PEERDNS=yes
		PEERROUTES=yes
		IPADDR=192.168.2.202
		GATEWAY=192.168.2.1
		NETMASK=255.255.255.0
		DNS1=8.8.8.83. 
#### 4.安装net-tools wget
	[root@localhost ~]# yum -y install net-tools	
	[root@localhost ~]# yum -y install wget
#### 5. 关闭，禁用防火墙 清除防火墙规则
	[root@localhost ~]# systemctl stop firewalld
	[root@localhost ~]# systemctl disable firewalld
	[root@localhost ~]# iptables -F && iptables -X && iptables -F -t nat && sudo iptables -X -t nat
    [root@localhost ~]# iptables -P FORWARD ACCEPT	
	
#### 6. 修改服务器名称
	[root@localhost ~]# hostnamectl set-hostname node2
#### 7. 修改host文件	
[root@localhost ~]# cat /etc/hosts

	127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
	::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
	192.168.2.200 master
	192.168.2.201 node1
	192.168.2.202 node2

#### 8. 同步时间
	[root@localhost ~]# yum -y install ntpdate
	[root@localhost ~]# ntpdate cn.pool.ntp.org
#### 9. 关闭swap分区 & 防止机器重启自动挂载swap 
	
	[root@localhost ~]# swapoff -a	
	[root@localhost ~]# sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab 
	 //注释  /etc/fstab  中相应的条目
	
	
#### 10.	关闭 SELinux	 & 禁用selinux
		[root@localhost ~]# setenforce 0
		[root@localhost ~]# $ vi /etc/selinux/config
		    SELINUX=disabled	
#### 11.添加docker-ce 源信息	
		[root@master ~]# wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
		[root@localhost ~]# sed -i 's@download.docker.com@mirrors.tuna.tsinghua.edu.cn/docker-ce@g' /etc/yum.repos.d/docker-ce.repo

#### 12. 配置k8s安装源
	[root@localhost yum.repos.d]# cd /etc/yum.repos.d/
	[root@localhost yum.repos.d]# vi kubernetes.repo
		[kubernetes]
		name=Kubernetes Repo
		baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
		gpgcheck=0
		enable=1	
#### 13. 更新yum仓库 
		[root@localhost yum.repos.d]# yum clean all
		[root@localhost yum.repos.d]# yum repolist		

#### 14. 安装docker
		[root@localhost yum.repos.d]# yum -y install docker-ce-17.03.2.ce	
	如果出现异常，安装下面这个，再装docker.
   		yum -y install https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.3.ce-1.el7.noarch.rpm
#### 15.查看yum可用 kube安装包
	[root@localhost yum.repos.d]# yum list | grep kube
		cockpit-kubernetes.x86_64                   176-4.el7.centos           extras   
		cri-tools.x86_64                            1.12.0-0                   kubernetes
		kubeadm.x86_64                              1.15.0-0                   kubernetes
		kubectl.x86_64                              1.15.0-0                   kubernetes
		kubelet.x86_64                              1.15.0-0                   kubernetes
		kubernetes.x86_64                           1.5.2-0.7.git269f928.el7   extras   
		kubernetes-client.x86_64                    1.5.2-0.7.git269f928.el7   extras   
		kubernetes-cni.x86_64                       0.7.5-0                    kubernetes
		kubernetes-master.x86_64                    1.5.2-0.7.git269f928.el7   extras   
		kubernetes-node.x86_64                      1.5.2-0.7.git269f928.el7   extras   
		rkt.x86_64                                  1.27.0-1                   kubernetes
		rsyslog-mmkubernetes.x86_64                 8.24.0-34.el7              base 	
		
#### 16.安装kubeadm kubectl kubelet
	[root@localhost yum.repos.d]# yum -y install kubeadm-1.15.0 kubelet-1.15.0 kubectl-1.15.0
#### 17.启动docker
	[root@localhost /]# mkdir -p /etc/docker
##### 1)添加加速器到配置文件
	[root@localhost /]# tee /etc/docker/daemon.json <<-'EOF'
		{
		 "registry-mirrors": ["https://registry.docker-cn.com"]
		}
		EOF
##### 2）启动服务
		[root@localhost /]# systemctl daemon-reload
	    [root@localhost /]# systemctl start docker
	    [root@localhost /]# systemctl enable docker.service
#### 18. 打开iptables内生的桥接相关功能，已经默认开启了，没开启的自行开启
		[root@localhost /]# cat /proc/sys/net/bridge/bridge-nf-call-ip6tables
		1
		[root@localhost /]# cat /proc/sys/net/bridge/bridge-nf-call-iptables
		1
#### 19. 配置启动kubelet服务
	修改配置文件：
	[root@localhost /]# cat /etc/sysconfig/kubelet
	KUBELET_EXTRA_ARGS="--fail-swap-on=false"
	KUBE_PROXY=MODE=ipvs	
	先设为开机自启
	[root@master ~]# systemctl enable kubelet.service
#### 20.初始化node结点
		
		[root@localhost /]# kubeadm join 192.168.2.200:6443 --token ypvpct.70w0zw75svoains7 --discovery-token-ca-cert-hash sha256:d6787edd612ebd44043d27cbdf9bab5bf8b5eaaec12a91faa2b0b4a2dc457adc --ignore-preflight-errors=Swap
		
#### 21.在master上验证集群
	   [root@master ~]# kubectl get nodes
			NAME     STATUS   ROLES    AGE    VERSION
			master   Ready    master   17h    v1.15.0
			node1    Ready    <none>   16h    v1.15.0
			node2    Ready    <none>   105s   v1.15.0