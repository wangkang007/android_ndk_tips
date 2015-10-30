# docker_tips
##docker学习笔记
###1.系统版本
操作系统：Ubuntu 15.10 64位

gcc：(Ubuntu 5.2.1-22ubuntu2) 5.2.1 20151010

Docker: version 1.6.2
###2.安装docker
&ensp;&ensp;从官方的Ubuntu软件库来安装：

	sudo apt-get update  
	sudo apt-get install docker.io

&ensp;&ensp;添加一个普通用户到docker组，如果你喜欢使用root用户来操作，这一步可以跳过
	
	sudo usermod -a -G docker test

&ensp;&ensp;安装完成后通过 ``` docker --version ```查看当前docker版本

###3.拉取镜像

&ensp;&ensp;1.通过一下命令从官网拉取最新版本的centos版本的镜像，我拉取的版本应该是当时最新版（CentOS Linux release 7.1.1503 (Core) ）	

	sudo docker pull centos
	#拉取指定版本的Ubuntu
	#sudo docker pull ubuntu:12.04

&ensp;&ensp;2.输入命令进入镜像，-i -t 开一个 tty 终端，保持交互模式，这两个一般共同使用。
	
	docker run -i -t centos /bin/bash 

&ensp;&ensp;3.docker run的命令解析

	-d Detached 或者 daemon mode，后台运行。
	-i -t 开一个 tty 终端，保持交互模式，这两个一般共同使用。
	-e 设置环境变量参数，参考 Install GitLab With Docker 。
	-p [host_port]:[container_port] 映射 HOST 端口到容器，方便外部访问容器内服务，host_port 可以省略，省略表示把 container_port 映射到一个动态端口。
	-v [host-path]:[container-path] 把 HOST 文件夹挂载到 Container 用以保存数据。
	--rm 自动删除已运行存在的相同 IMAGE 的容器。
	
&ensp;&ensp;根据上述命令来看，docker一般来说用户存放服务器，配置好的一些环境，但是不能用于文件的存储，一般文件的存储都会挂载到宿主的磁盘进行保存，比如说一些日志和数据库的数据文件是需要通过挂载到宿主的文件磁盘系统进行保存的，而docker配置好的一些文件信息通过commit持久化到docker 镜像中。

###4.docker网络桥接
&ensp;&ensp;网上看了一些教程，大家都是直接拉取镜像后能够直接访问网络，但是我拉取下来的版本并不能够进行访问网络，所以这里通过bridge-utils工具进行桥接：

&ensp;&ensp;切换到宿主机器：

	#centos 安装bridge-utils
	yum install bridge-utils
	#暂停docker服务
	sudo service docker stop
	#用ip命令使docker0网卡down掉
	sudo ip link set dev docker0 down
	#删除网卡
	sudo brctl delbr docker0
	#创建一个网卡 名字是bridge0
	sudo brctl addbr bridge0
	#ip地址和子网
	ip addr add 192.168.5.1/24 dev bridge0
	#启动桥接网卡
	sudo ip link set dev bridge0 up
	#写入配置
	echo 'DOCKER_OPTS="-b=bridge0"' >> /etc/default/docker
	#启动docker
	sudo service docker start

&ensp;&ensp;通过之前的命令重新进入到镜像，此时你会发觉你能访问外网了，但是又会存在一个问题，外网无法访问处于docker内的镜像，所以此时需要跟宿主机器进行端口的映射才能够访问，可以回到上面的命令解析部分，通过参数-p可以实现端口的映射：

	docker run -i -t -p 8080:80 centos /bin/bash

&ensp;&ensp;文件映射加上端口映射，组合成完善的环境。

	docker run -t -i -p 8080:80 -v /home/docker/centos:/home  centos /bin/bash

&ensp;&ensp;在现实生产环境中，可能会遇到多个端口映射的问题，docker支持多个端口的映射。```-p 8080:80 -p 808:80``` 这样的方式也是可以使用的。