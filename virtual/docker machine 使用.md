# docker machine 

`docker machine` 如何在本地虚拟机中创建、使用和管理`Docker`主机。



要运行`Docker`容器需要：

- 创建一个新的（或启动一个现有的）`Docker`虚拟机
- 将您的环境切换到新的`VM`
- 使用`docker`客户端创建，加载和管理容器

一旦你创建一台机器，你可以随时重复使用它。像任何`VirtualBox VM`一样，它在使用之间保持其配置。

这里的示例显示了如何创建和启动计算机，运行`Docker`命令以及使用容器。



# 创建虚拟机

## 终端命令窗口

命令窗口可以是`Bash shell`、`xshell`、`powershell`，只要能运行`shell`即可。

## 查看机器

使用`docker-machine ls`列出可用的机器，如果创建了机器会显示已有的机器。

```shell
$ docker-machine ls
NAME           ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
default        -        virtualbox   Running   tcp://192.168.99.100:2376           v18.04.0-ce
my-vm-node-1   *        virtualbox   Running   tcp://192.168.99.101:2376           v18.04.0-ce
my-vm-node-2   -        virtualbox   Running   tcp://192.168.99.102:2376           v18.04.0-ce
my-vm-node-3   -        virtualbox   Running   tcp://192.168.99.103:2376           v18.04.0-ce
```

## 创建VM

运行该`docker-machine create`命令，将相应的驱动程序`virtualbox`传递给该 `--driver`标志并提供一个机器名称。如果这是您的第一台机器，请将其命名`default`为示例中所示。如果您已有“default”机器，请为此新机器选择另一个名称。

```shell
$ docker-machine create --driver virtualbox default
Running pre-create checks...
Creating machine...
(staging) Copying /Users/ripley/.docker/machine/cache/boot2docker.iso to /Users/ripley/.docker/machine/machines/default/boot2docker.iso...
(staging) Creating VirtualBox VM...
(staging) Creating SSH key...
(staging) Starting the VM...
(staging) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Machine is running, waiting for SSH to be available...
Detecting operating system of created instance...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect Docker to this machine, run: docker-machine env default
```

`--driver virtualbox` 表示虚拟机驱动提供商是**virtualbox**。该命令下载安装了`Docker`守护程序的轻量级`Linux`发行版（[boot2docker](https://github.com/boot2docker/boot2docker)），创建并启动运行`Docker`的`VirtualBox`虚拟机。



## 查看环境变量

`env | grep DOCKER` 查看环境变量配置，下面的可以看出`shell`品类中的机器是`my-vm-node-1`

```shell
$ env | grep DOCKER
DOCKER_MACHINE_NAME=my-vm-node-1
DOCKER_CERT_PATH=C:\Users\Administrator\.docker\machine\machines\my-vm-node-1
DOCKER_TLS_VERIFY=1
DOCKER_HOST=tcp://192.168.99.101:2376
DOCKER_TOOLBOX_INSTALL_PATH=E:\Docker Toolbox
```



## 配置`shell` 连接的机器

`docker-machine create` 命令输出中提示，您需要Docker与新机器通信，你可以用`docker-machine env`命令来设置。

```shell
$ docker-machine env default
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="C:\Users\Administrator\.docker\machine\machines\default"
export DOCKER_MACHINE_NAME="default"
export COMPOSE_CONVERT_WINDOWS_PATHS="true"
# Run this command to configure your shell:
# eval $("E:\Docker Toolbox\docker-machine.exe" env default)
```

通过运行环境变量后，会提示运行`shell`，运行完后`docker-machine`就可以一直保持和对应的机器进行链接通信了。随后查看命令链接的机器后，发现是`default`节点。

```shell
$ eval $("E:\Docker Toolbox\docker-machine.exe" env default)

$ env | grep DOCKER
DOCKER_MACHINE_NAME=default
DOCKER_CERT_PATH=C:\Users\Administrator\.docker\machine\machines\default
DOCKER_TLS_VERIFY=1
DOCKER_HOST=tcp://192.168.99.100:2376
DOCKER_TOOLBOX_INSTALL_PATH=E:\Docker Toolbox
```



如果是其他`shell`则需要指定下对应的终端命令软件。

**PowerShell:**

```shell
$ docker-machine.exe env --shell powershell dev
$Env:DOCKER_TLS_VERIFY = "1"
$Env:DOCKER_HOST = "tcp://192.168.99.101:2376"
$Env:DOCKER_CERT_PATH = "C:\Users\captain\.docker\machine\machines\dev"
$Env:DOCKER_MACHINE_NAME = "dev"
# Run this command to configure your shell:
# docker-machine.exe env --shell=powershell dev | Invoke-Expression
```

 `cmd.exe`:

```shell
$ docker-machine.exe env --shell cmd dev
set DOCKER_TLS_VERIFY=1
set DOCKER_HOST=tcp://192.168.99.101:2376
set DOCKER_CERT_PATH=C:\Users\captain\.docker\machine\machines\dev
set DOCKER_MACHINE_NAME=dev
# Run this command to configure your shell: copy and paste the above values into your command prompt
```

**注意**：如果您正在使用`fish`或`Powershell`、`cmd.exe`，则上述方法不能按所述方式工作。相反，看看[该`env`命令的文档](https://docs.docker.com/machine/reference/env/) ，以了解如何设置环境变量的`shell`。

这为Docker客户端读取的当前shell设置环境变量，它指定`TLS`设置。每次打开新`shell`或重新启动计算机时都需要执行此操作。（另请参阅如何 [在当前shell中取消设置环境变量](https://docs.docker.com/machine/get-started/#unset-environment-variables-in-the-current-shell)。）



# 操作虚拟机

## 获取主机IP地址

Docker主机的`IP`地址上提供了任何公开的端口，您可以使用以下`docker-machine ip`命令获得这些端口：

```shell
 $ docker-machine ip default
 192.168.99.100
```



## 运行应用

使用以下命令在容器中运行[Nginx](https://www.nginx.com/) Web服务器，`-d` 表示后台运行，`-p` 表示发布应用：

```shell
 $ docker run -d -p 8000:80 nginx
```

`nginx`下载安装运行完成后，通过当前虚拟机ip `docker-machine ip`访问 `8080`端口，可以看到`nginx`成功运行

```shell
$ curl $(docker-machine ip default):8000
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0    612      0  0:00:01 --:--:--  0:00:01  597k<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

可以创建和管理、运行多个Docker的本地虚拟机，因为可以通过再次运行 `docker-machine create`创建新的虚拟机。所有创建的机器都出现在输出中`docker-machine ls`。



## 启动和停止虚拟机

如果需要启动或停止虚拟机，可以用停止命令`docker-machine stop` 启动虚拟机命令`docker-machine start`。

```shell
$ docker-machine stop default
$ docker-machine start default
```



## 不带指定机器名称操作`shell`

一些`docker-machine`命令的操作应该在名为`default`的机器（如果存在的话）上运行，如果没有指定机器名称的话。因为使用名为`default`的本地虚拟机是一种常见模式。

下面一些不带机器名称的命令操作，都会默认定位到`default`机器

```shell
$ docker-machine stop
Stopping "default"....
Machine "default" was stopped.

$ docker-machine start
Starting "default"...
(default) Waiting for an IP...
Machine "default" was started.
Started machines may have new IP addresses.  You may need to re-run the `docker-machine env` command.

$ eval $(docker-machine env)

$ docker-machine ip
192.168.99.100
```

还有其他命令也支持不带机器名进行操作

```shell
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



## 配置环境变量与指定机器交互

有时候可能想要使用当前`shell`连接到不同的`Docker Engine`。在这种情况下，可以选择将当前shell的环境切换到不同的Docker引擎。

### 查看环境变量

运行`env|grep DOCKER`以检查是否设置了DOCKER环境变量。

```shell
$ env | grep DOCKER
DOCKER_MACHINE_NAME=default
DOCKER_CERT_PATH=C:\Users\Administrator\.docker\machine\machines\default
DOCKER_TLS_VERIFY=1
DOCKER_HOST=tcp://192.168.99.100:2376
DOCKER_TOOLBOX_INSTALL_PATH=E:\Docker Toolbox
```

通过上面的输出信息可以看到当前环境变量配置默认链接的机器是`default`

### 取消环境变量

---

- `unset`在以下`DOCKER`环境变量上运行该命令

  ```shell
  unset DOCKER_TLS_VERIFY
  unset DOCKER_CERT_PATH
  unset DOCKER_MACHINE_NAME
  unset DOCKER_HOST
  ```

- 或者，运行快捷命令`docker-machine env -u`以显示您需要运行的命令以取消设置所有DOCKER变量：

  ```shell
  $ docker-machine env -u
  unset DOCKER_TLS_VERIFY
  unset DOCKER_HOST
  unset DOCKER_CERT_PATH
  unset DOCKER_MACHINE_NAME
  # Run this command to configure your shell:
  # eval $("E:\Docker Toolbox\docker-machine.exe" env -u)
  ```

  最后再运行上面输出的最后一行命令 `eval $("E:\Docker Toolbox\docker-machine.exe" env -u)`就取消了环境变量设置。

  取消完成后，再次查看环境变量，发现环境变量已经都取消成功。

  ```shell
  $ env | grep DOCKER
  DOCKER_TOOLBOX_INSTALL_PATH=E:\Docker Toolbox
  ```

  **如果您在`Docker Cloud`上运行群集，则可以重新运行`export` 用于连接群集的命令。**



## 机器启动时默认配置环境变量

为确保在每个shell会话开始时自动配置Docker客户端，可以`eval $(docker-machine env default)`通过将其添加到`~/.bash_profile`文件或shell的等效配置文件中来嵌入shell 配置文件。但是，如果调用的计算机`default`未运行，则会失败 。您可以配置系统以`default` 自动启动机器。



# 命令行汇总

```shell
$ docker-machine ls						# 查看已安装的虚拟机
$ docker-machine create --driver virtualbox default	# 创建虚拟机 --driver virtualbox 表示虚拟机驱动提供商是virtualbox，default 表示虚拟机的名称
$ env | grep DOCKER						# 查看 环境变量
$ docker-machine env default  			# 查看指定机器环境变量
$ docker-machine env -u 				# 取消环境变量设置
$ eval $(docker-machine env default)	# 配置指定机器环境里的shell默认链接机器
$ docker-machine.exe env --shell powershell dev  # 指定shell目录的终端工具
$ docker-machine ip default					# 获取指定机器地址
$ docker run -d -p 8000:80 nginx			# -d 表示后台运行，-p 表示发布应用
$ curl $(docker-machine ip default):8000		# 访问指定机器
$ docker-machine stop default				# 启动指定机器
$ docker-machine start default				# 停止 指定 机器

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

