helloworld项目打成jar包，上传master, docker打包成镜像，
push到私库,然后kubectl apply yamlfile，创建两个pod。
这个过程，看似简单，搞了挺久的，差点放弃。
坑1：
	registery为了简单，使用了http,而不是https,而docker默认是https的。push失败。
	解决过程：
		1.在网上搜，基本上都是写在/etc/docker/daemon.json中加入非安全模式。
			[root@node2 ~]# cat /etc/docker/daemon.json 
				{
				  "insecure-registries":["registry:5000"]
				}
		  然后执行
		  	 systemctl daemon-reload
		  	 systemctl restart docker.service
		  	 
		 但是依然push失败。
		原因：有些blog上并没有说，还需要后续步骤，需要docker login registry才行。
		 这个问题可以参考这篇文章。
		 https://blog.csdn.net/lusyoe/article/details/79587914
坑2：
	接坑1，在master上login registry后，创建pod还是失败。
	通过kubectl decsribe pod， 看到pod创建失败的原因，都是node1,node2 拉镜像失败。
	原因：这问题其实想想就能知道，用docker pull, push试下registery，肯定能看出是node1,2与registry不通，或者没logig。
	解决办法：
		1.配置域名解析，能ping通registry:5000。
		2.在node1,node2上 login registry。按照坑1里的方法。
思考问题：
	1.master上跑kubectl apply,但真正创建pod的server是谁？ 
		从我的机器上看，是node1,node2.
	  衍生问题：master,node各自工作是啥？
	2.pod网络与service网络是不同的子网，请求的接收与响应会不会太复杂？
	3.service网络怎么暴露？ 请求如何转发？
	
	

		
 docker build -t registry:5000/say-hello:0.0.0 -f ./Dockerfile .
 docker push registry:5000/say-hello:0.0.1
 kubectl apply -f ./app_say_hello.yaml