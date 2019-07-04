Systemd是啥？
管理系统所有进程、服务，启动项的软件成为系统管理器。      
d是守护的缩写，systemd字面意思是守护整个系统。   
CentOS7之前，用system v管理。   
从 CentOS 7 开始，Systemd 成为新的系统管理器。我认为它最大的优点就是支持进服务并行启动，从而使效率大大提高；同时它还具有日志管理、快照备份与恢复、挂载点管理等多种实用功能，
功能甩 System V 几条街！   
而且 systemd 进程的 PID 是 1 ，也就是说 Systemd 掌管着一切进程！   

使用 
   思维的转变：  
	service serviceName start --> systemctl start serviceName  
	各种命令：
	 systemctl start serviceName
	 systemctl stop serviceName
	 systemctl restart serviceName
	 systemctl status serviceName
	 systemctl try-restart serviceName
	 systemctl is-active serviceName
	 systemctl disable serviceName
	 systemctl enable serviceName
	 systemctl is-enable serviceName
	 

