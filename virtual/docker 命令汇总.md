# docker 常用命令

```shell
# image
# 建立一个图像
$ docker image build --rm=true .
# 安装图像
$ docker image pull ${IMAGE}
#已安装图像的列表
$ docker image ls
# 已安装图像列表（详细列表）
$ docker image ls --no-trunc
# 删除图像
$ docker image rm ${IMAGE_ID}
# 删除未使用的图像
$ docker image prune
# 删除所有图像
$ docker image rm $(docker image ls -aq)

#container
# 运行一个容器
$ docker container run
# 正在运行的容器列表
$ docker container ls
# 所有容器的列表
$ docker container ls -a
# 停止一个容器
$ docker container stop ${CID}
# 停止所有运行的容器
$ docker container stop $(docker container ls -q)
# 列出状态为1的所有退出的容器
$ docker container ls -a --filter "exited=1"
# 取出一个容器
$ docker container rm ${CID}
# 通过正则表达式删除容器
$ docker container ls -a | grep wildfly | awk '{print $1}' | xargs docker container rm -f
# 删除所有已退出的容器
$ docker container rm -f $(docker container ls -a | grep Exit | awk '{ print $1 }')
# 删除所有容器
$ docker container rm $(docker container ls -aq)
# 查找容器的IP地址
$ docker container inspect --format '{{ .NetworkSettings.IPAddress }}' ${CID}
# 附加到一个容器
$ docker container attach ${CID}
# 进入一个容器，打开一个shell
$ docker container exec -it ${CID} bash
# 通过正则表达式获取图像的容器标识
$ docker container ls | grep wildfly | awk '{print $1}'
```





## docker VM

```shell
$ docker-machine -h
Usage: docker-machine.exe [OPTIONS] COMMAND [arg...]
创建和管理运行Docker的机器.

Options:
  --debug, -D                                                   启用调试模式
  --storage-path, -s "C:\Users\Administrator\.docker\machine"   配置存储路[$MACHINE_STORAGE_PATH]
  --tls-ca-cert                                                 CA远程验证[$MACHINE_TLS_CA_CERT]
  --tls-ca-key                                                生成证书的私钥[$MACHINE_TLS_CA_KEY]
  --tls-client-cert                                用于TLS的客户端证书[$MACHINE_TLS_CLIENT_CERT]
  --tls-client-key                                用于客户端TLS认证的私钥[$MACHINE_TLS_CLIENT_KEY]
  --github-api-token                            令牌用于请求Github API[$MACHINE_GITHUB_API_TOKEN]
  --native-ssh                                    使用本地（基于Go）的SSH实现[$MACHINE_NATIVE_SSH]
  --bugsnag-api-token                  用于崩溃报告的BugSnag API令牌[$MACHINE_BUGSNAG_API_TOKEN]
  --help, -h                                                    帮助
  --version, -v                                                 打印版本

Commands:
  active                打印哪台机器处于活动状态
  config                打印机器的连接配置
  create                创建一台机器
  env                   显示设置Docker客户端环境的命令
  inspect               检查检查有关机器的信息
  ip                    获取一台机器的IP地址
  kill                  杀死一台机器
  ls                    列出机器
  provision             准备重新调配现有机器
  regenerate-certs      重新生成证书为机器重新生成TLS证书
  restart               重新启动重新启动机器
  rm                    删除一台机器
  ssh                   使用SSH登录或在机器上运行命令.
  scp                   在机器之间复制文件
  mount                 使用SSHFS挂载或卸载机器上的目录.
  start                 开始启动一台机器
  status                获取机器的状态
  stop                  停止一台机器
  upgrade               将计算机升级到最新版本的Docker
  url                   获取一台机器的URL
  version               Show the Docker Machine version or a machine docker version
  help                  Shows a list of commands or help for one command
```



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

## docker compose

```shell
$ docker-compose -h
使用Docker定义和运行多容器应用程序

Usage:
  docker-compose [-f <arg>...] [options] [COMMAND] [ARGS...]
  docker-compose -h|--help

Options:
  -f, --file FILE             指定一个备用的compose file
                              (默认: docker-compose.yml)
  -p, --project-name NAME     指定一个替代项目名称
                              (默认: 目录名称)
  --verbose                   显示更多输出
  --log-level LEVEL           设置日志级别 (DEBUG, INFO, WARNING, ERROR, CRITICAL)
  --no-ansi                   不要打印ANSI控制字符
  -v, --version               打印版本并退出
  -H, --host HOST             用于连接到的HOST守护程序套接字

  --tls                       使用TLS;--tlsverify
  --tlscacert CA_PATH         仅由此CA签署的信任证书
  --tlscert CLIENT_CERT_PATH  证书文件的路径
  --tlskey TLS_KEY_PATH       密钥文件的路径
  --tlsverify                 使用TLS并验证远程
  --skip-hostname-check       不要检查守护进程的主机名
                              在客户端证书中指定的名称
  --project-directory PATH    指定一个备用工作目录
                              (默认: Compose file的路径)
  --compatibility             如果设置，Compose将尝试转换部署
                              将v3文件中的密钥添加到其非Swarm等效项

Commands:
  build              构建或重建服务
  bundle             从Compose文件中生成一个Docker bundle
  config             验证并查看Compose file
  create             创建服务
  down               停止并移除容器，网络，图像和卷
  events             接收来自容器的实时事件
  exec               在正在运行的容器中执行命令 
  images             镜像列表
  kill               杀死容器
  logs               查看容器的日志输出
  pause              暂停服务
  port               打印端口绑定的公共端口
  ps                 容器列表
  pull               拉取服务镜像
  push               推送服务镜像
  restart            重启服务
  rm                 移除停止的容器
  run                运行一次性命令
  scale              设置服务的容器数量
  start              开始服务
  stop               停止服务
  top                显示正在运行的进程
  unpause            暂停服务
  up                 创建并启动容器 
```



```shell
# 安装 docker-compose
$ sudo curl -L https://github.com/docker/compose/releases/download/1.21.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
# 授权
$ sudo chmod +x /usr/local/bin/docker-compose
$ docker-compose migrate-to-labels				# 升级compose
$ docker container rm -f -v myapp_web_1 myapp_db_1   # 删除容器
$ docker-compose up							# 启动应用
$ docker-compose up -d					# 后台模式启动应用
$ docker-compose stop					# 停止应用
$ docker-compose down					# 卸载应用
$ docker-compose ps						# 查看应用状态
$ docker-compose run web env			# 查看应用服务环境变量
$ docker-compose down --volumes			# 卸载应用并删除data数据
```

