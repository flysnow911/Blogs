#  git拉取远程分支到本地

#### 1.创建文件夹
	mkdir hello-world
#### 2.初使化git
	git init 
#### 3.查看是否与远程分支建立连接
	git remote -v 
#### 4.建立连接
	git remote add origin  git@github.com:flysnow911/k8s-hello-world.git
#### 5.指定pull的分支
	git pull origin master
#### 6.指定push的分支
	git push --set-upstream origin master