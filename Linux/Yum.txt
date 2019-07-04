Yum (Yellow Dog Updater,Modifier):
redhat,centos,fedora软件管理包。能从指定服务器下载软件安装包，自动处理安装包依赖关系，并且可以一次性安装所有软件。
主要命令：
	1.yum list installed
	2.yum list tomcat 
	3.yum -y install tomcat (
		-y可以自动应答所有yes
	4.yum remove tomcat
	5.yum deplist tomcat
	6.yum info tomcat 
	7.yum update tomcat
		不加tomcat，表示更新所有软件
	8.yum check-update tomcat
主要yum源：
国外：
	EPEL Repository
	  yum localinstall --nogpgcheck http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm
	REMI Repository
	
	详细参考：
		https://www.tecmint.com/yum-thirdparty-repositories-for-centos-rhel/
	
国内：
    网易，阿里
	