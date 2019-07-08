构建docker镜像并执行
1.构建可执行jar
2.编写Dockerfile
	示例：
		FROM openjdk:8-jre-alpine
		MAINTAINER *** <***@gmail.com>
		CMD ["nohup","java","-jar","./hello-world-1.0-SNAPSHOT.jar","&"]
3.把jar和Dockerfile放在server上一个目录
4.构建docker镜像
	docker build -t alibaba/helloworld:1.0.0 -f ./Dockerfile .
5.验证镜像是否构建成功
	docker images | grep alibaba/helloworld
6.执行镜像
	docker run -p 9191:9191 -it alibaba/helloworld:1.0.0 
ctl + P +Q 可以退出控制台，但容器正常运行。
