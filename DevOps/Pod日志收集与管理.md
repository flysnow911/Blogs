pod日志管理有多种，一般主流是下面这种：   
host中以DaemonSet的模式运行logging-agent,pod的容器都通过stdout,stderr打印到日志文件中log_file.log，   
host上的存储日志文件目录挂载到容器中。logging-agent负责将日志转发出去，日志最后存在es中。   
并且在host中，常启动一个logrotate，日志文件size达到10M后，就rotate一把。   
优点：
	简单，对容器没有侵入。
	每个host只需一个logging-agent。
	kubectl logs 也可以用。


