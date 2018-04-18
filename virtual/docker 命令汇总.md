# docker 常用命令

## docker VM

```shell
$ docker-machine ls						# 查看已安装的虚拟机
$ docker-machine create --driver virtualbox default	# 创建虚拟机 --driver virtualbox 表示虚拟机驱动提供商是virtualbox，default 表示虚拟机的名称
$ env | grep DOCKER						# 查看 环境变量
$ docker-machine env default  			# 查看指定机器环境变量
$ docker-machine env -u 				# 取消环境变量设置
$ eval $(docker-machine env default)	# 配置指定机器环境里的shell默认链接机器
$ docker-machine env --shell powershell dev  # 指定shell目录的终端工具
$ docker-machine ip default					# 获取指定机器地址
$ docker run -d -p 8000:80 nginx			# -d 表示后台运行，-p 表示发布应用
$ curl $(docker-machine ip default):8000		# 访问指定机器
$ docker-machine stop default				# 启动指定机器
$ docker-machine start default				# 停止 指定 机器

$ docker-machine scp dev:/home/docker/foo.txt .  #复制远程主机文件到本地
$ docker-machine rm baz				# 删除主机
$ docker-machine mount dev:/home/docker/foo foo		# 挂载目录

- `docker-machine config` 			# 机器配置
- `docker-machine env`				# 环境变量
- `docker-machine inspect`			# 检查
- `docker-machine ip`				# 机器 ip
- `docker-machine kill`				# 杀死机器
- `docker-machine provision`		# 重做固定的任务	
- `docker-machine regenerate-certs`  # 注册生成证书
- `docker-machine restart`			# 重启机器
- `docker-machine ssh`				# ssh 链接机器
- `docker-machine start`			# 启动
- `docker-machine status`			# 运行状态
- `docker-machine stop`				# 停止
- `docker-machine upgrade`			# 升级
- `docker-machine url`				# tcp url
```



## docker 安装

```shell
## List Docker CLI commands
docker
docker container --help

## 显示docker版本和信息
docker --version
docker version
docker info

## Execute Docker image 运行镜像
docker run hello-world

## List Docker images 查看所有镜像
docker image ls

## List Docker containers (running, all, all in quiet mode)
docker container ls					# 查看正在运行的容器
docker container ls --all					# 查看所有容器
docker container ls -aq						# 查看未运行的容器
docker-machine restart default				# 重启虚拟机

$ docker-machine ssh default 				# ssh 连接到default这台虚拟机
```



## docker 容器

```shell
docker build -t friendlyhello .  # 使用此目录的Dockerfile创建图像
docker run -p 4000:80 friendlyhello  # 运行“friendlyname”映射端口4000到80
docker run -d -p 4000:80 friendlyhello         # 同上，但处于分离模式
docker run --detach --publish 4000:80 --name webserver nginx
docker ps                               # 列出所有正在运行的容器
docker container ls                                # 列出所有正在运行的容器
docker container ls -a             	# 列出所有容器，即使那些没有运行的容器
docker container stop <hash>           # 停止指定的容器
docker container kill <hash>         # 强制关闭指定的容器
docker container rm <hash>        	# 从本机中移除指定的容器
docker container rm $(docker container ls -a -q)         # 删除所有容器
docker image ls -a                             # 列出此机器上的所有图像
docker image rm <image id>            # 从本机中删除指定的图像
docker image rm $(docker image ls -a -q)   # 从本机中删除所有图像
docker images myhello                      # 通过仓库查看镜像
docker login             			# 登录
docker tag <image> username/repository:tag  # 标签<image>用于上传到仓库
docker push username/repository:tag            # 上传标记的图像到仓库
docker run username/repository:tag                   # 从仓库运行图像，本地没有会先拉取镜像
docker push registry/username/repository:tag 
docker run registry/username/repository:tag

docker-machine ssh default                             # ssh 链接到虚拟机 default
docker-machine ip                             # 查看虚拟机ip地址
```



## docker 服务

```shell
docker stack ls                                      # 程序和堆列表
docker stack deploy -c <compose-yml-file> <appname>  # 部署compose.yml文件到堆
docker service ls                                      # 显示所有在运行的服务程序
docker service ps <service>                  		 # 显示所有程序服务的任务
docker inspect <task or container>                    # 检查任务或容器
docker container ls -q                                # 显示容器 IDs
docker stack rm <appname>                             # 关闭或删除容器程序
docker swarm leave --force                             # 强制关闭节点集群
docker swarm init --advertise-addr eth1               # 启动集群、暴露某个ip地址
```



## docker 集群

```shell
docker-machine create --driver virtualbox myvm1 	# 创建虚拟机 myvm1 (Mac, Win7, Linux)
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm1 # Win10 下创建虚拟机
docker-machine env myvm1                		 # 查看myvm1节点基本信息
docker-machine ssh myvm1 "docker node ls"         # 查看myvm1集群节点信息
docker-machine ssh myvm1 "docker node inspect <node ID>"        # 检查节点
docker-machine ssh myvm1 "docker swarm join-token -q worker"   # 查看加入集群的token
docker-machine ssh myvm1							# ssh 链接到 VM; 退出输入 "exit" 结束
docker node ls								# 查看集群中的节点（登录到管理器时）
docker-machine ssh myvm2 "docker swarm leave"  # 使 worker节点离开集群
docker-machine ssh myvm1 "docker swarm leave -f" # 主节点离开集群，并杀死集群
docker-machine ls 						# VMs 列表, 星号显示这个shell正在与哪个虚拟机通话
docker-machine start myvm1            	# 启动虚拟机，如果在没启动的情况下
docker-machine env myvm1      			# 显示myvm1的环境变量和命令
eval $(docker-machine env myvm1)         # 将 shell 链接到 myvm1
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression   # windows 将 shell 链接到 myvm1
docker stack deploy -c <file> <app>  # 部署程序; 必须设置命令shell与管理器（myvm1）交互，使用本地compose文件
docker-machine scp docker-compose.yml myvm1:~ # 将文件复制到主节点的目录（仅当您使用的ssh连接到管理器并部署应用程序时才需要）
docker-machine ssh myvm1 "docker stack deploy -c <file> <app>"   # 使用ssh的部署应用程序（您必须首先将compose文件复制到myvm1）
eval $(docker-machine env -u)     						# 从虚拟机断开shell，使用本地docker
docker-machine stop $(docker-machine ls -q)               # 停止全部运行的虚拟机
docker-machine rm $(docker-machine ls -q) 				# 删除所有运行的虚拟机，包括磁盘上的
```

