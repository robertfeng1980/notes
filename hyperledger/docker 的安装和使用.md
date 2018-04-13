# docker 的安装和使用

[TOC]

## docker 概述

docker是开发人员和系统管理员 使用容器**开发、部署和运行**应用程序的平台。使用Linux容器来部署应用程序称为*装箱化*。容器不是新的，但它们用于轻松部署应用程序。

装箱化越来越受欢迎，因为装箱是：

- **灵活：**即使是最复杂的应用程序也可以进行集装箱化。
- **轻量级：**容器利用并共享主机内核。
- **可互换：**您可以即时部署更新和升级。
- **便携式：**您可以在本地构建，部署到云中并在任何地方运行。
- **可扩展性：**您可以增加和自动分发容器副本。
- **可堆叠：**您可以垂直堆叠服务并即时堆叠服务。



## docker 的安装

docker版本众多，这里介绍Windows下如何下载安装。在Win7系统上，docker安装需要使用docker toolbox，而Win10则直接使用docker CE，还有企业版的 docker EE。



### 下载

Win10 docker CE下载地址：https://docs.docker.com/docker-for-windows/install/

Win 7 Docker Toolbox 下载地址：https://docs.docker.com/toolbox/overview/

以上选择你的平台的下载链接即可下载



docker toolbox 是旧版系统使用的docker安装包，toolbox包含以下工具：

> 工具箱包括这些Docker工具：
>
> - Docker Machine用于运行`docker-machine`命令
> - Docker Engine用于运行`docker`命令
> - Docker Compose用于运行这些`docker-compose`命令
> - Kitematic，Docker GUI
> - 一个为Docker命令行环境预配置的shell
> - Oracle VirtualBox
> - Git
>
> 例如，您可以在[Toolbox Releases](https://github.com/docker/toolbox/releases)上找到各种版本的工具，或者使用`--version`终端中的标志运行它们`docker-compose --version`。

docker toolbox使用的VM是VirtualBox，而Win 10 系统的docker使用的VM则是Hyper-V



### 安装

> 安装很简单，直接下一步，其中有一个选项则是选择安装工具，如果你的电脑安装过Git和VirtualBox的可以不选这两个选项。其他的直接下一步完成即可。



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

> 看到上面的画面就启动成功了，下面可以开始命令行操作。
>
> docker is configured to use the default machine with IP 192.168.99.100
>
> 这条信息就是docker虚拟机对应的ip地址



继续在上面的窗口输入命令行，或者打开电脑中的`dos`命令行窗口 `cmd`，或者是使用`powershell`、`git`、`xshell`都可以。这里使用`git bash`窗口进行操作。

输入命令行`docker --version`，查看版本

```shell
$ docker --version
Docker version 18.03.0-ce, build 0520e24302
```

> 看到以上版本信息说明docker安装正确，无问题。



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

> 介绍了一下docker的基本配置信息，包括重要的容器信息、镜像信息、存储目录等



## docker 的使用

### 镜像和容器

通过运行镜像启动容器。一个**镜像**是一个可执行的包，其中包括运行应用程序代码所需的所有内容、运行时、库、环境变量和配置文件。

**容器**是镜像的运行时实例，当被执行时（即镜像的状态或者用户进程）在容器中变得可以监控查看。您可以使用该命令`docker ps`查看正在运行的镜像列表。



### 容器和虚拟机

一个**容器**中运行*原生* Linux和共享主机与其它容器的内核。它运行一个独立的进程，不占用任何其他可执行文件的内存，使其轻量化。

相比之下，**虚拟机**（VM）运行一个完整的“客户”操作系统，通过虚拟机管理程序*虚拟*访问主机资源。一般来说，虚拟机提供的环境比大多数应用程序需要的资源更多。



### 基本用法

+ 运行hello world示例

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

+ 查看已经安装的镜像

  输入命令行`docker images` 或者 `docker image ls`

  ```shell
  $ docker image ls
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  hello-world         latest              e38bc07ac18e        37 hours ago        1.85kB
  ```
  ​

+ 查看在容器中运行过的镜像信息

  输入命令行`docker container ls --all`，如果查看正在运行的镜像可以用`docker container ls`

  ```shell
  $ docker container ls --all
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
  ff46aa1e0dc6        hello-world         "/hello"            9 minutes ago       Exited (0) 9 minutes ago                       priceless_vaughan

  ```
  ​

+ 查看运行过的镜像容器id信息

  输入命令行`docker container ls -aq`

  ```shell
  $ docker container ls -aq
  ff46aa1e0dc6
  ```




### 创建镜像容器

从程序最底层构建`docker`容器，当我们运行`python`程序的时候，首先需要安装`python`的sdk库用来支撑程序的运行。这个时候你的环境在安装这些程序的时候会有各种差异导致版本冲突，并且在你安装好后也许和生产环境的版本不一致或者存在差异。

使用`docker`可以很好的解决上面描述的问题，docker可以很好的将一个可以移植的python环境当成一个镜像进行运行。你只需用构建一个可以运行python和你的python代码程序，以及包含其他依赖配置的镜像就可以让这个打包程序在所有存在docker的环境进行运行。



下面将演示如何构建自己机器上可以运行python的docker镜像

+ 定义镜像容器的`Dockerfile`

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

  > 上面的文件是docker镜像需要的文件内容和依赖，其中有`requirements.txt`,`app.py`，其他的是配置信息。从上面的注释可以看到：
  >
  > 利用官方的`Python`作为父镜像
  >
  > 设置工作目录为 `/app`
  >
  > 将当前目录下的内容复制到上面的 `app`目录中
  >
  > 安装`requirements.txt`中指定的所有必需软件包
  >
  > 提供对外端口：`80`
  >
  > 定义系统变量
  >
  > 执行命令行 `python app.py`来运行`app.py`python程序脚本

  ​

+ 开始镜像中的程序内容

  由于上面的 `Dockerfile`定义好了镜像的内容，这里需要对文件中的内容进行填充

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

  > 从上面的代码可以看出程序依赖了`from flask import Flask`,`from redis import Redis`，并且访问了系统变量`os.getenv("NAME", "world")`，程序运行的端口是`app.run(host='0.0.0.0', port=80)`。这就是`requirements.txt` 和 `Dockerfile`文件内容的原因了。
  >
  > 现在我们看到`pip install -r requirements.txt`为Python安装Flask和Redis库，并且该应用程序输出环境变量`NAME`以及调用的输出`socket.gethostname()`。最后，因为Redis没有运行（因为我们只安装了Python的Redis库，而没有Redis），所以在这里会失败并发生错误异常消息。

  ​

+ 构建镜像程序

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

  > 上面的命令行`docker build -t myhello .`是构建一个`myhello`的镜像，镜像的内容就是当前目录下的文件内容。上面的提示可以看到有7个步骤，后面的信息没有贴出来。

  ​

+ 查看已经安装注册的镜像程序

  ```shell
  $ docker image ls
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  myhello             latest              75be173204aa        9 minutes ago       150MB
  hello-world         latest              e38bc07ac18e        38 hours ago        1.85kB
  python              2.7-slim            b16fde09c92c        3 weeks ago         139MB
  ```

  ​

+ 运行镜像程序

  ```shell
  $ docker run -p 4000:80 myhello
   * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
  ```

  > 上面的命令是将镜像程序发布出去，其中的80端口映射的4000端口上，通过docker关联的虚拟机的ip地址就可以访问到当前镜像应用。

  ​

  利用以下命令查看ip地址

  ```shell
   $ docker-machine ip
   192.168.99.100
  ```

  ​

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

  ​

+ 停止运行的镜像程序

  利用`docker container ls`查看正在运行的镜像，利用`docker container stop [CONTAINER ID]`停止镜像

  ```shell
  $ docker container ls
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
  77e422fa3978        myhello             "python app.py"     22 minutes ago      Up 22 minutes       0.0.0.0:4000->80/tcp   elated_ptolemy


  $ docker container stop 77e422fa3978
  77e422fa3978
  ```

+ 在后台运行镜像程序

  `docker run -d`分离模式在后台运行应用程序，命令行如下

  ```shell
  $ docker run -d -p 4000:80 myhello
  faf0d327cb8ba130fb69cd7719c8d2732ebe6f3f223e93afa68bc7b9e612b429
  ```

  > 上面返回的一段id是容器运行的id，也可以通过命令`docker container ls`查看短id

  ```shell
  $ docker container stop faf0d327cb8ba130fb69cd7719c8d2732ebe6f3f223e93afa68bc7b9e612b429
  ```




### 分享镜像

分享镜像就是将个人的镜像推送到docker云端，这样大家都可以进行拉取共享使用。docker云端和github有些类似，在使用云端之前需要先注册，注册地址：[cloud.docker.com](https://cloud.docker.com/)

+ 登陆docker 云端

  利用命令行`docker login `进行登陆，其中`-u` 代表`username`，`-p` 是密码

  ```shell
  $ docker login -p xxxx -u xxxx
  WARNING! Using --password via the CLI is insecure. Use --password-stdin.
  Login Succeeded
  ```

+ 登出docker 云端，命令行 `docker logout`


+ 建立一个镜像的标签，方便于查看和管理

  命令行`docker tag image username/repository:tag`，其中`image`代表镜像的名称、 `username`代表用户id、`repository`表示集合仓库名称、`tag`则是标签名称。示例如下：

  ```shell
  # docker tag image username/repository:tag

  $ docker tag myhello hoojo/test:my_hello_world

  $ docker image ls
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  hoojo/test          my_hello_world      75be173204aa        About an hour ago   150MB
  myhello             latest              75be173204aa        About an hour ago   150MB
  hello-world         latest              e38bc07ac18e        39 hours ago        1.85kB
  python              2.7-slim            b16fde09c92c        3 weeks ago         139MB
  ```

  > 通过上面的命令发现多了一个镜像，就是刚才标签过的。注意看`REPOSITORY`，`TAG`

+ 提交镜像到云端仓库

  使用命令行`docker push`来进行推送，整个推送可能很慢，由于是国外的网站

  ```shell
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

  推送完成后可以去docker云端看下有没有：https://cloud.docker.com/swarm/your-username/repository/list

  ​

## 参考文档

https://docs.docker.com/get-started/part2/