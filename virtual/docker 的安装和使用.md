# docker 的安装和使用

[TOC]

本文的是Docker*入门教程* 教你如何：

1. 设置Docker环境

2. 构建一个图像并将其作为一个容器运行

3. 扩展应用程序以运行多个容器

4. 在整个集群中分配应用程序

5. 通过添加后端数据库来堆叠服务

6. 将应用部署到生产

   ​

# docker 概述

docker是开发人员和系统管理员 使用容器**开发、部署和运行**应用程序的平台。使用Linux容器来部署应用程序称为*镜像化*。容器不是新的，但它们用于轻松部署应用程序。

镜像化越来越受欢迎，镜像化的优点有：

- **灵活：** 即使是最复杂的应用程序也可以进行镜像化。
- **轻量级：** 容器利用并共享主机内核。
- **可互换：** 您可以即时部署更新和升级。
- **便携式：** 您可以在本地构建，部署到云中并在任何地方运行。
- **可扩展性：** 您可以增加和自动分发容器副本。
- **可堆叠：** 您可以垂直堆叠服务并即时堆叠服务。




## 镜像和容器

通过运行镜像启动容器。一个**镜像**是一个可执行的包，其中包括运行应用程序代码所需的所有内容、运行时、库、环境变量和配置文件。

**容器**是镜像的运行时实例，当被执行时（即镜像的状态或者用户进程）在容器中变得可以监控查看。您可以使用该命令`docker ps`查看正在运行的镜像列表。



## 容器和虚拟机

一个**容器**中运行*原生* Linux和共享主机与其它容器的内核。它运行一个独立的进程，不占用任何其他可执行文件的内存，使其轻量化。

相比之下，**虚拟机**（VM）运行一个完整的“客户”操作系统，通过虚拟机管理程序*虚拟*访问主机资源。一般来说，虚拟机提供的环境比大多数应用程序需要的资源更多。




# docker 安装

docker版本众多，这里介绍Windows下如何下载安装。在Win7系统上，docker安装需要使用docker toolbox，而Win10则直接使用docker CE，还有企业版的 docker EE。



## 下载

Win10 docker CE下载地址：https://docs.docker.com/docker-for-windows/install/ <br/>Win 7 Docker Toolbox 下载地址：https://docs.docker.com/toolbox/overview/ <br/>

以上选择你的平台的下载链接即可下载



docker toolbox 是旧版系统使用的docker安装包，toolbox包含以下工具：

> 工具箱包括这些Docker工具：
>
> - `Docker Machine`用于运行`docker-machine`命令，用来操作虚拟机
> - `Docker Engine`用于运行`docker`命令
> - `Docker Compose`用于运行这些`docker-compose`命令
> - `Kitematic`，`Docker GUI`图形界面
> - 一个为`Docker`命令行环境预配置的shell
> - `Oracle VirtualBox`
> - `Git`
>
> 例如，您可以在[Toolbox Releases](https://github.com/docker/toolbox/releases)上找到各种版本的工具，或者使用`--version`终端中的标志运行它们`docker-compose --version`。

`docker toolbox`使用的`VM`是`VirtualBox`，而`Win 10` 系统的`docker`使用的`VM`则是`Hyper-V`



## 安装

安装很简单，直接下一步，其中有一个选项则是选择安装工具，如果你的电脑安装过Git和VirtualBox的可以不选这两个选项。其他的直接下一步完成即可。

找到docker安装目录，在目录下会有一个`start.sh`，使用`Git bash`运行这个脚本。双击脚本会执行启动docker程序操作，这个过程可能有点慢。

```shell
$ cd E:\Docker Toolbox
$ sh start.sh
```

当运行`start.sh`后会自动检查`boot2docker.iso`的最新版本，会提示下载最新的`boot2docker.iso`，这个时候程序自动回去`github`上进行下载。可以看到窗口日志

```shell
Running pre-create checks...
(manager) Default Boot2Docker ISO is out-of-date, downloading the latest release...
(manager) Latest release for github.com/boot2docker/boot2docker is v1.11.2
(manager) Downloading C:\Users\Administrator\.docker\machine\cache\boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v1.18.2/boot2docker.iso...
Error with pre-create check: "Get https://github-cloud.s3.amazonaws.com/releases/ 
...

The process cannot access the file because it is being used by another process.
(default)
Error with pre-create check: "read tcp 10.3.73.28:7668->52.216.225.248:443: wsarecv: An existing connection was forcibly closed by the remote host."
Looks like something went wrong in step ´Checking if machine default exists´... 
Press any key to cont   
```

如果下载完成后还是无法启动，可以手动下载`boot2docker.iso`，下载地址在错误日志中有，我这里的地址是：https://github.com/boot2docker/boot2docker/releases/download/v1.18.2/boot2docker.iso 这个地址来自于上面的日志，每个人可能版本号不一致。所以要以日志里的为准，然后去下载这个`boot2docker.iso`。下载完成后，将文件放置在`C:\Users\Administrator\.docker\machine\cache\`目录下，这个地址也是来自于上面的错误日志。

> 网上还说有另一种解决办法，就是将docker toolbox安装目录的`boot2docker.iso`复制到`C:\Users\Administrator\.docker\machine\cache\`目录下，然后**断掉网络**在运行`start.sh`脚本就不会去检查了。



将以上步骤完成后，再运行`start.sh`应该就可以成功启动了，启动后画面类似如下

```shell
Administrator@hoojo-pc MINGW64 /e/Docker Toolbox
$ sh start.sh
Starting "default"...
(default) Check network to re-create if needed...
(default) Windows might ask for the permission to configure a dhcp server. Sometimes, such confirmation window is minimized in the taskbar.
(default) Waiting for an IP...
Machine "default" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.
Regenerate TLS machine certs?  Warning: this is irreversible. (y/n): Regenerating TLS certificates
Waiting for SSH to be available...
Detecting the provisioner...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...


                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/

docker is configured to use the default machine with IP 192.168.99.100
For help getting started, check out the docs at https://docs.docker.com


Start interactive shell
```

看到上面的画面就启动成功了，下面可以开始命令行操作。<br/>docker is configured to use the default machine with IP 192.168.99.100<br/>这条信息就是docker虚拟机对应的`	ip`地址

> 当然，你也可以使用docker GUI界面图形的方式去启动docker。在安装完成后，可以双击桌面上的`a`图标，点击启动就可以启动docker了。启动后，如果你有docker云端账号，登陆上去还可以进行云端镜像管理操作。点击左下角的`Docker CLI`就可以开启命令行窗口。



## 测试

继续在上面的窗口输入命令行，或者打开电脑中的`dos`命令行窗口 `cmd`，或者是使用`powershell`、`git`、`xshell`都可以。这里使用`git bash`窗口进行操作。

输入命令行`docker --version`，查看版本

```shell
$ docker --version
Docker version 18.03.0-ce, build 0520e24302
```

看到以上版本信息说明docker安装正确，无问题。

输入命令行`docker version`，查看更多版本信息

```shell
$ docker version
Client:
 Version:       18.03.0-ce
 API version:   1.37
 Go version:    go1.9.4
 Git commit:    0520e24302
 Built: Fri Mar 23 08:31:36 2018
 OS/Arch:       windows/amd64
 Experimental:  false
 Orchestrator:  swarm

Server:
 Engine:
  Version:      18.04.0-ce
  API version:  1.37 (minimum version 1.12)
  Go version:   go1.9.4
  Git commit:   3d479c0
  Built:        Tue Apr 10 18:23:35 2018
  OS/Arch:      linux/amd64
  Experimental: false
```

输入命令行`docker info`，查看info信息

```shell
$ docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 18.04.0-ce
Storage Driver: aufs
 Root Dir: /mnt/sda1/var/lib/docker/aufs
 Backing Filesystem: extfs
 Dirs: 0
 Dirperm1 Supported: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: 773c489c9c1b21a6d78b5c538cd395416ec50f88
runc version: 4fc53a81fb7c994640722ac585fa9ca548971871
init version: 949e6fa
Security Options:
 seccomp
  Profile: default
Kernel Version: 4.9.93-boot2docker
Operating System: Boot2Docker 18.04.0-ce (TCL 8.2.1); HEAD : b8a34c0 - Wed Apr 11 17:00:55 UTC 2018
OSType: linux
Architecture: x86_64
CPUs: 1
Total Memory: 995.6MiB
Name: default
ID: XADN:VFBU:3C4G:D3YG:BZ5P:4ZX6:TP34:55MN:PM3J:KMEL:KLYA:4OUU
Docker Root Dir: /mnt/sda1/var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Labels:
 provider=virtualbox
Experimental: false
Insecure Registries:
 127.0.0.0/8
Live Restore Enabled: false
```

介绍了一下docker的基本配置信息，包括重要的容器信息、镜像信息、存储目录等。

ssh链接到指定虚拟机，可以打开本地vbox软件，看看哪些虚拟机，然后利用`docker-machine ssh`去打开

```shell
$ docker-machine ssh default
```



## 镜像加速

### **Ubuntu系统加速**

------

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://8kzs1r91.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



### **Win 7 如何配置镜像加速器**

------

针对安装了Docker Toolbox的用户，您可以参考以下配置步骤：
创建一台安装有Docker环境的Linux虚拟机，指定机器名称为default，同时配置Docker加速器地址。

```shell
$ docker-machine create --engine-registry-mirror=https://8kzs1r91.mirror.aliyuncs.com -d virtualbox default
```

对于已经安装创建过虚拟机的，可以先`docker-machine ssh default`链接到`default`虚拟机如下处理

```shell
sudo sed -i "s|EXTRA_ARGS='|EXTRA_ARGS='--registry-mirror=https://8kzs1r91.mirror.aliyuncs.com |g" /var/lib/boot2docker/profile

# 退出
exit
# 重启虚拟机
docker-machine restart default
```



查看机器的环境配置，并配置到本地，并通过Docker客户端访问Docker服务。

```shell
docker-machine env default
eval "$(docker-machine env default)"
docker info
```

针对安装了Docker for Windows的用户，您可以参考以下配置步骤：
在系统右下角托盘图标内右键菜单选择 Settings，打开配置窗口后左侧导航菜单选择 Docker Daemon。编辑窗口内的JSON串，填写加速器地址，如下所示：

```shell
{
  "registry-mirrors": ["https://8kzs1r91.mirror.aliyuncs.com"]
}
```

编辑完成，点击 Apply 保存按钮，等待Docker重启并应用配置的镜像加速器。



## 本节命令汇总

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
docker container ls							# 查看正在运行的容器
docker container ls --all					# 查看所有容器
docker container ls -aq						# 查看未运行的容器
docker-machine restart default				# 重启虚拟机

$ docker-machine ssh default 				# ssh 连接到default这台虚拟机
```



# docker 容器

## 基本命令

### 运行hello world示例

---

从`docker hub`上拉取`hello world`示例(https://hub.docker.com/r/library/hello-world/)并运行

```shell
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9bb5a5d4561a: Pulling fs layer
9bb5a5d4561a: Verifying Checksum
9bb5a5d4561a: Download complete
9bb5a5d4561a: Pull complete
Digest: sha256:bbdaf0ed64b665f3061aeab15b946697dd00845161935d9238ed28e8cfc2581c
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

省略其他。。。。。。
```

由于docker的hello world示例镜像在docker的官网，可能需要翻墙并且速度慢，这里可以使用国内阿里镜像，打开本地的刚才安装好的虚拟机`Oracle VM VirtualBox`，找到`default`。这是`default`已经在运行，直接右键点击**显示**即可，然后输入命令

```shell
$ cd /etc/docker
$ vi daemon.json
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

> 在`daemon.json`中加入上面的配置即可加速

​

### 查看已经安装的镜像

---

输入命令行`docker images` 或者 `docker image ls`

```shell
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              e38bc07ac18e        37 hours ago        1.85kB
```
​

### 查看在容器中运行过的镜像信息

---

输入命令行`docker container ls --all`，如果查看正在运行的镜像可以用`docker container ls`

```shell
$ docker container ls --all
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
ff46aa1e0dc6        hello-world         "/hello"            9 minutes ago       Exited (0) 9 minutes ago                       priceless_vaughan

```
​

### 查看运行过的镜像容器`ID`信息

---

输入命令行`docker container ls -aq`

```shell
$ docker container ls -aq
ff46aa1e0dc6
```


## 创建镜像容器

从程序最底层构建`docker`容器，当我们运行`python`程序的时候，首先需要安装`python`的`sdk`库用来支撑程序的运行。这个时候你的环境在安装这些程序的时候会有各种差异导致版本冲突，并且在你安装好后也许和生产环境的版本不一致或者存在差异。

使用`docker`可以很好的解决上面描述的问题，`docker`可以很好的将一个可以移植的`python`环境当成一个镜像进行运行。你只需用构建一个可以运行`python`和你的`python`代码程序，以及包含其他依赖配置的镜像就可以让这个打包程序在所有存在`docker`的环境进行运行。



下面将演示如何构建自己机器上可以运行`python`的`docker`镜像

### 定义镜像容器的`Dockerfile`

---

`Dockerfile`定义容器内环境中做了些什么。在镜像环境中需要访问网络接口和磁盘驱动器等资源，这些资源与系统其余部分是隔离的，因此需要将端口映射到外部，并明确要将哪些文件“复制”到镜像环境。在完成这些之后，可以让构建 `Dockerfile`的应用程序在任何机器上都可以运行。

在目标位置`/d/docker/python_sample`创建`Dockerfile`文件，粘贴一下内容，命令如下：

```shell
$ mkdir docker
$ cd docker/
$ mkdir python_sample
$ cd python_sample/

$ vi Dockerfile
```

将下面的文本内容粘贴至 `Dockerfile`

```dockerfile
# Use an official Python runtime as a parent image
FROM python:2.7-slim
# Set the working directory to /app
WORKDIR /app
# Copy the current directory contents into the container at /app
ADD . /app
# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt
# Make port 80 available to the world outside this container
EXPOSE 80
# Define environment variable
ENV NAME World
# Run app.py when the container launches
CMD ["python", "app.py"]
```

> 上面的文件是docker镜像需要的文件内容和依赖，其中有`requirements.txt`、`app.py`，其他的是配置信息。从上面的注释可以看到：
>
> 利用官方的`Python`作为父镜像<br/>设置工作目录为 `/app`<br/>将当前目录下的内容复制到上面的 `app`目录中<br/>安装`requirements.txt`中指定的所有必需软件包<br/>提供对外端口：`80`<br/>定义系统变量<br/>执行命令行 `python app.py`来运行`app.py`python程序脚本<br/>



### 编写镜像中的程序

---

由于上面的 `Dockerfile`定义好了镜像的内容，这里需要对文件中的内容进行填充。

创建两个文件，`requirements.txt`、`app.py`将它们放在和`Dockerfile`同一个文件夹中。完成应用程序的编码部分，这个取决于你的镜像需要运行什么代码内容。当上述`Dockerfile`被内置到的图像，`app.py`和 `requirements.txt`是因为存在`Dockerfile`文件的`ADD`命令中，而端口`80`是`app.py`是通过HTTP访问需要的，这个暴露端口需要使用`EXPOSE` 命令。而系统变量`ENV NAME World`则是`app.py`中需要访问的变量值。

还是在`Dockerfile`同一个文件夹中创建`requirements.txt`文件，添加当前python程序的依赖软件

```shell
$ vi requirements.txt
Flask
Redis
```

创建`app.py`文件，内容如下

```python
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

从上面的代码可以看出程序依赖了`from flask import Flask`,`from redis import Redis`，并且访问了系统变量`os.getenv("NAME", "world")`，程序运行的端口是`app.run(host='0.0.0.0', port=80)`。这就是`requirements.txt` 和 `Dockerfile`文件内容的原因了。

现在我们看到`pip install -r requirements.txt`为Python安装Flask和Redis库，并且该应用程序输出环境变量`NAME`以及调用的输出`socket.gethostname()`。最后，因为Redis没有运行（因为我们只安装了Python的Redis库，而没有Redis），所以在这里会失败并发生错误异常消息。

​

### 构建镜像程序

---

通过上面的准备，现在可以构建应用的镜像程序工作，利用`docker build`来构建镜像，命令行如下：

```shell
$ docker build -t myhello .
Sending build context to Docker daemon  4.608kB
Step 1/7 : FROM python:2.7-slim
2.7-slim: Pulling from library/python
b0568b191983: Pulling fs layer
55a7da9473ae: Pulling fs layer
422d2e7f1272: Pulling fs layer
8fb86f1cff1c: Pulling fs layer
8fb86f1cff1c: Waiting
55a7da9473ae: Verifying Checksum
55a7da9473ae: Download complete
8fb86f1cff1c: Verifying Checksum
8fb86f1cff1c: Download complete
422d2e7f1272: Verifying Checksum
422d2e7f1272: Download complete
b0568b191983: Verifying Checksum
b0568b191983: Download complete
b0568b191983: Pull complete
55a7da9473ae: Pull complete
422d2e7f1272: Pull complete
8fb86f1cff1c: Pull complete
Digest: sha256:9e24a026a55ca1d9a7284db30ed846b7190a3d7f557edf493b454bff362ed64c
Status: Downloaded newer image for python:2.7-slim
.......
```

上面的命令行`docker build -t myhello .`是构建一个`myhello`的镜像，镜像的内容就是当前目录下的文件内容。上面的提示可以看到有7个步骤，后面的信息没有贴出来。



### 查看已经安装的镜像程序

---

```shell
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
myhello             latest              75be173204aa        9 minutes ago       150MB
hello-world         latest              e38bc07ac18e        38 hours ago        1.85kB
python              2.7-slim            b16fde09c92c        3 weeks ago         139MB
```



### 运行镜像程序

---

```shell
$ docker run -p 4000:80 myhello
 * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
```

上面的命令是将镜像程序发布出去，其中的80端口映射的4000端口上，通过docker关联的虚拟机的ip地址就可以访问到当前镜像应用。

​

### 查看`ip`地址

---

```shell
 $ docker-machine ip
 192.168.99.100
```



### 测试镜像程序

---

访问：http://192.168.99.100:4000/

```html
Hello World!
Hostname: 77e422fa3978
Visits: cannot connect to Redis, counter disabled
```

或者在虚拟机上运行命令行查看

```shell
$ curl http://localhost:4000

<h3>Hello World!</h3><b>Hostname:</b> 8fc990912a14<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
```



### 停止运行的镜像程序

---

利用`docker container ls`查看正在运行的镜像，利用`docker container stop [CONTAINER ID]`停止镜像

```shell
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
77e422fa3978        myhello             "python app.py"     22 minutes ago      Up 22 minutes       0.0.0.0:4000->80/tcp   elated_ptolemy
```

```shell
$ docker container stop 77e422fa3978
77e422fa3978
```



### 在后台运行镜像程序

---

 `docker run -d`分离模式在后台运行应用程序，命令行如下

  ```shell
$ docker run -d -p 4000:80 myhello
faf0d327cb8ba130fb69cd7719c8d2732ebe6f3f223e93afa68bc7b9e612b429
  ```

上面返回的一段id是容器运行的id，也可以通过命令`docker container ls`查看短id

  ```shell
$ docker container stop faf0d327cb8ba130fb69cd7719c8d2732ebe6f3f223e93afa68bc7b9e612b429
  ```




## 分享镜像

分享镜像就是将个人的镜像推送到docker云端，这样大家都可以进行拉取共享使用。docker云端和github有些类似，在使用云端之前需要先注册，注册地址：[cloud.docker.com](https://cloud.docker.com/)。国内阿里镜像地址：https://dev.aliyun.com/

### 登陆docker 云端

---

利用命令行`docker login `进行登陆，其中`-u` 代表`username`，`-p` 是密码

```shell
$ docker login -p xxxx -u xxxx
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Login Succeeded

# 或者登陆其他仓库
$ sudo docker login --username=xxx -p xxxx registry.cn-hangzhou.aliyuncs.com
```


### 登出docker 云端

---

```shell
$ docker logout
```



### 建立标签

---

建立一个镜像的标签，方便于查看和管理

命令行`docker tag image username/repository:tag`，其中`image`代表镜像的名称、 `username`代表用户id、`repository`表示集合仓库名称、`tag`则是标签名称。若是第三方仓库则需要带仓库地址，同时`useranme`部分在阿里仓库中被称为**命名空间**，而且需要手动创建。示例如下：

```shell
# docker tag image username/repository:tag
# 为其他仓库打标签
# docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/username_namespace/repository:[tag]

$ docker tag myhello hoojo/test:my_hello_world

$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hoojo/test          my_hello_world      75be173204aa        About an hour ago   150MB
myhello             latest              75be173204aa        About an hour ago   150MB
hello-world         latest              e38bc07ac18e        39 hours ago        1.85kB
python              2.7-slim            b16fde09c92c        3 weeks ago         139MB
```

通过上面的命令发现多了一个镜像，就是刚才标签过的。注意看`REPOSITORY`，`TAG`

### 提交镜像

---

提交镜像到云端仓库。使用命令行`docker push`来进行推送，整个推送可能很慢，由于是国外的网站。若使用国内阿里镜像则需要在前面带上仓库地址`registry.cn-hangzhou.aliyuncs.com/hoojo/test:my_hello_world`

```shell
# 为其他仓库push
# docker push registry.cn-hangzhou.aliyuncs.com/hoojo/test:my_hello_world

$ docker push hoojo/test:my_hello_world
The push refers to repository [docker.io/hoojo/test]
0c5f092b929e: Preparing
9a6b4154e4ff: Preparing
b475497029b3: Preparing
d99e7ab4a34b: Preparing
332873801f89: Preparing
2ec65408eff0: Preparing
43efe85a991c: Preparing
```

推送完成后可以去`docker`云端看下有没有：https://cloud.docker.com/swarm/your-username/repository/list



## 运行远程云端的镜像

从现在开始，您可以使用`docker run`此命令在任何机器上使用并运行远程的应用程序：

```shell
docker run -p 4000:80 username/repository:tag

$ docker run -p 4000:80 hoojo/test:my_hello_world
 * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
```

这样就把云镜像端程序运行在本地了，如果本地没有当前需要的镜像程序，则`docker`会在远程拉取程序到本地进行运行。无论在哪里`docker run`执行，它都会将会下载Python以及所有依赖项从中拉出`requirements.txt`，然后运行镜像中的代码。它们都在一个整体小包中一起运行，你不需要在主机上安装任何东西来让`docker`运行它。



## 本节命令汇总

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
docker login             					# 登录
docker tag <image> username/repository:tag  # 标签<image>用于上传到仓库
docker push username/repository:tag            # 上传标记的图像到仓库
docker run username/repository:tag                   # 从仓库运行图像，本地没有会先拉取镜像
docker push registry/username/repository:tag 
docker run registry/username/repository:tag

docker-machine ssh default 					# ssh 链接到虚拟机 default
docker-machine ip 							# 查看虚拟机ip地址
```



# docker 服务

在分布式应用程序中，应用程序的不同部分被称为“服务”。例如，如果您想象一个视频共享网站，它可能包含一个用于将应用程序数据存储在数据库中的服务，一个用于在后台进行视频转码的服务用户上传的东西，前端的服务等等。服务会更改运行该软件的容器实例的数量，从而为流程中的服务分配更多计算资源，**这就相当于在一台机器上部署了一套服务的集群，提高了服务的并发和高性能**。

幸运的是，使用Docker平台定义，运行和扩展服务非常简单 - 只需编写一个`docker-compose.yml`文件即可。



## 配置 `docker-compose.yml`文件

`docker-compose.yml`文件是一个`YAML`文件，它定义了Docker容器在生产中的配置方式。

在任意目录位置创建文件`docker-compose.yml`，并粘贴如下内容

```shell
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    # pull image 拉取文件，这里对应远程仓库中的镜像
    image: hoojo/test:my_hello_world 
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"
    networks:
      - webnet
networks:
  webnet:
```

> 该`docker-compose.yml`文件告诉Docker执行以下操作：
>
> - 从注册表中拉出[我们在步骤2中上传的图像](https://docs.docker.com/get-started/part2/)。
> - 运行该镜像的`5`个实例作为所调用的服务`web`，限制每个实例使用最多`10％`的`CPU`（跨所有核心）和`50MB`的`RAM`。
> - 如果一个失败，立即重启容器。
> - 将主机上的端口80映射到`web`端口80。
> - `web`通过称为负载平衡的网络指示容器共享端口80 `webnet`。（在内部，容器本身`web`在临时端口上发布到 80端口。）
> - `webnet`使用默认设置定义网络（这是一个负载平衡覆盖网络）。



## 运行负载平衡应用程序

### 启动集群

---

在使用`docker stack deploy`命令进行部署之前，首先运行如下命令

```shell
$ docker swarm init
Error response from daemon: could not choose an IP address to advertise since this system has multiple addresses on different interfaces (10.0.2.15 on eth0 and 192.168.99.100 on eth1) - specify one with --advertise-addr
```

出现以上问题后可以按照说明解决，解决方法如下

```shell
$ docker swarm init --advertise-addr eth1
# or
$ docker swarm init --advertise-addr 192.168.99.100
# or
$ docker swarm init --listen-addr 192.168.99.100:2377 --advertise-addr 192.168.99.100:2377
```



### 部署服务

---

现在正式部署镜像程序，将部署后的程序服务命名为：`run_hello`。创建网络`run_hello_webnet`，创建服务`run_hello_web`

```shell
$ docker stack deploy -c docker-compose.yml run_hello
Creating network run_hello_webnet
Creating service run_hello_web
```



### 查看服务

---

我们在单个堆栈上将一个镜像程序部署在一个主机上运行`5`个实例的服务。查看服务列表：

```shell
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                       PORTS
6vtacv7c9tq5        run_hello_web       replicated          5/5                 hoojo/test:my_hello_world   *:80->80/tcp
```

默认服务名称会在之前部署在堆栈`docker stack deploy -c docker-compose.yml run_hello`的时候的名称后面加上`web`，这里的服务名称应该是 `run_hello_web`。还列出了服务`ID`以及副本数，映射的名称和已公开端口。



### 查看任务

---

在服务中运行的单个容器称为**任务**。任务的的`id`是一个定量增加的数据，这个定量取决于在文件`docker-compose.yml`配置的`replicas` 的数量。查看服务的任务：

```shell
$ docker service ps run_hello_web
ID                  NAME                IMAGE                       NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
ksy1vbv0t4qf        run_hello_web.1     hoojo/test:my_hello_world   default             Running             Running 5 minutes ago
b38a3x3pjym5        run_hello_web.2     hoojo/test:my_hello_world   default             Running             Running 5 minutes ago
x8syohl5prfs        run_hello_web.3     hoojo/test:my_hello_world   default             Running             Running 5 minutes ago
z8k0wsfecw49        run_hello_web.4     hoojo/test:my_hello_world   default             Running             Running 5 minutes ago
ri68se61gfc2        run_hello_web.5     hoojo/test:my_hello_world   default             Running             Running 5 minutes ago
```



通过查看系统容器列表，也会显示所有的服务任务数据，这些任务无法被过滤

```shell
$ docker container ls -q
CONTAINER ID        IMAGE                       COMMAND             CREATED             STATUS              PORTS               NAMES
540ea1cf9981        hoojo/test:my_hello_world   "python app.py"     15 minutes ago      Up 15 minutes       80/tcp              run_hello_web.1.ksy1vbv0t4qf1g3mfd0x1xonu
1bed6aaa2541        hoojo/test:my_hello_world   "python app.py"     15 minutes ago      Up 15 minutes       80/tcp              run_hello_web.3.x8syohl5prfsyfpao6nf0xrge
97026758b279        hoojo/test:my_hello_world   "python app.py"     15 minutes ago      Up 15 minutes       80/tcp              run_hello_web.5.ri68se61gfc2fski787fdgz34
d30c0bc32ac8        hoojo/test:my_hello_world   "python app.py"     15 minutes ago      Up 15 minutes       80/tcp              run_hello_web.4.z8k0wsfecw49z9zodkbebeoum
354825ff501b        hoojo/test:my_hello_world   "python app.py"     15 minutes ago      Up 15 minutes       80/tcp              run_hello_web.2.b38a3x3pjym5k3w90qyyd2apz
```



直接访问IP地址：http://192.168.99.100 就可以看到镜像运行的结果

```shell
Hello World!
Hostname: 1bed6aaa2541
Visits: cannot connect to Redis, counter disabled
```

通过连续刷新`url`，会发现`hostname`发生变化，这里的`hostname`正是对应上面的`CONTAINER ID`，这就表明5个实例可以任意访问，负载均衡已发生作用。



## 无缝更新应用程序

当我们需要修改`docker-compose.yml`的配置信息，不需要先关闭集群、删除服务、杀死删除容器，只需用修改配置，然后重新部署发布`docker stack deploy`，这样大大的提高了效率、缩短了发布的间歇时间。

比如我们修改`docker-compose.yml`的副本`replicas `数量为`2`，然后重新发布

```shell
$ docker stack deploy -c docker-compose.yml run_hello
Updating service run_hello_web (id: 6vtacv7c9tq5b2m6i0fetan85)
```

上面的日志反馈是更新服务`run_hello_web`，通过目录查看服务容器`docker container ls -q`，会发现少了

```shell
$ docker container ls
CONTAINER ID        IMAGE                       COMMAND             CREATED             STATUS              PORTS               NAMES
540ea1cf9981        hoojo/test:my_hello_world   "python app.py"     29 minutes ago      Up 29 minutes       80/tcp              run_hello_web.1.ksy1vbv0t4qf1g3mfd0x1xonu
354825ff501b        hoojo/test:my_hello_world   "python app.py"     29 minutes ago      Up 29 minutes       80/tcp              run_hello_web.2.b38a3x3pjym5k3w90qyyd2apz

```



## 关闭服务和集群

删除发布在堆栈中的服务`docker stack rm`

```shell
$ docker stack rm run_hello
Removing service run_hello_web
Removing network run_hello_webnet
```

停掉集群

```shell
$ docker swarm leave --force
Node left the swarm.
```

用Docker轻松发布、开启并扩展您的应用程序非常简单。



## 本节命令汇总

```shell
docker stack ls                                      # 程序和堆列表
docker stack deploy -c <compose-yml-file> <appname>  # 部署compose.yml文件到堆
docker service ls                 					# 显示所有在运行的服务程序
docker service ps <service>                  		 # 显示所有程序服务的任务
docker inspect <task or container>                    # 检查任务或容器
docker container ls -q                                # 显示容器 IDs
docker stack rm <appname>                             # 关闭或删除容器程序
docker swarm leave --force      					# 强制关闭节点集群
docker swarm init --advertise-addr eth1               # 启动集群、暴露某个ip地址
```



# docker 集群

将应用程序部署到群集上，并在多台机器上运行它。多容器，多机应用程序通过连接多台机器到称为一个“Dockerized”**集群**。

Swarm是一组运行Docker并加入到集群中的机器。下面的Docker命令将由**群集管理器**在群集上执行。群体中的机器可以是物理的或虚拟的。加入群体后的机器被称为**节点**。

Swarm管理人员可以使用多种策略来运行容器，例如“最空节点” —— 它可以使用容器充分利用使用率最低的机器。它确保每台机器只获取指定容器的一个实例。指示集群管理器在Compose文件中使用这些策略，就像您已经使用的策略一样。

集群管理器是集群中唯一可以执行命令的机器，或者授权其他机器作为**工作者**加入群体。工作者只是在那里提供能力，并没有权力告诉任何其他机器可以做什么和不可以做什么。

到目前为止，您已经在本地机器上以单主机模式使用Docker。但是Docker也可以切换到**群集模式**，这就是使用群集的原因。立即启用群模式使当前的机器成为群管理器。从此，Docker将运行您在您管理的群集上执行的命令，而不仅仅是在当前机器上执行。



# 参考文档

https://docs.docker.com/get-started/part2/

https://docs.docker.com/docker-for-windows/

http://www.fecshop.com/topic/591