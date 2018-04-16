# 在windows下搭建hyperledger fabric 开发环境

[TOC]



`hyperledger fabric` 开发环境搭建是一项大工程，这里主要描述`windows`下的环境搭建。其他`linux`和`mac`不做描述。我的机器是`win7 64`位系统。



# 一、准备工作

![开发环境架构图](https://raw.githubusercontent.com/hyperledger/fabric/release-1.1/devenv/images/openchain-dev-env-deployment-diagram.png)

上面的这张图是官方的，通过这张图可以不难看出，`fabric`的开发环境分本地和虚拟机两个部分。本地需要安装`git、vagrant、vbox`，并且需要配置`GoPath`、以及自己的工作空间和`local-dev`。而`vagrant`虚拟的镜像中需要`python、docker、golang`。<br/>将本地的`$GoPath\src\github.com\hyperledger`映射到虚拟机上的`/opt/gopath/src/github.com/hyperledger`。<br/>将本地的`$GoPath\src\github.com\hyperledger`映射到虚拟机上的`hyperledger`，后期这个目录可以放置其他项目代码或示例。<br/>将本地`$LOCALDEVDIR`环境变量（默认`/local-dev`）映射到虚拟机上的`/local-dev`

```shell
default: /vagrant => D:/GoPath/src/github.com/hyperledger/fabric/devenv
default: /local-dev => D:/GoPath/src/github.com/hyperledger
default: /hyperledger => D:/GoPath/src/github.com/hyperledger
default: /opt/gopath/src/github.com/hyperledger => D:/GoPath/src/github.com/hyperledger
```



## 1、命令终端工具

命令终端工具很多，有常用的`xshell`、`Cygwin64 Terminal`、`Git`，这里推荐使用`Git`

- [ ] 下载安装文件：https://git-for-windows.github.io/
- [ ] 加入配置，防止路径太长太深出现错误
  - `git config --global core.autocrlf false`
  - `git config --global core.longpaths true`

## 2、Go 语言

- [ ] 下载安装文件：https://golang.org/dl/
- [ ] 配置环境变量，在电脑系统变量中进行新建配置
  - `go` 安装目录环境变量配置：`GOROOT`=`E:\Go`
  - `go` 工作目录环境变量配置：`GOPATH`=`D:\GOPATH`
    - 在`D:\GOPATH`创建目录`src`，`bin`，`pkg`
    - 在`src`目录创建`github.com\hyperledger` 这个目录后面会放`hyperledger fabric`的代码

## 3、Vagrant 工具

vagrant 工具就是一个用命令行和脚本的方式，帮你创建虚拟机和安装虚拟机里面的软件的一个工具。并且它支持把安装好的环境进行打包成`*.box`然后在其他机器上再通过vagrant很方便的添加到机器上。

- [ ] 下载安装文件：https://www.vagrantup.com/downloads.html

- [ ] 安装完成后重启电脑

  ​

## 4、Virtualbox 虚拟机

来自于`Oracle`的虚拟机，小巧便用、占用空间小。

- [x] 下载安装文件：https://www.virtualbox.org/wiki/Downloads
- [ ] 直接下一步，安装完成即可



## 5、Hyperledger Fabric 源码的安装

`hyperledger fabric` 平台核心代码也就是运行的基础，需要从`github`上下载

```shell
$ cd GoPath/src/github.com/hyperledger
$ git clone https://github.com/hyperledger/fabric.git
$ git checkout v1.0.3
```





# 二、搭建开发环境

由于上面已经准备好了环境的必要工具，下面全程用命令行的方式去安装虚拟机。



## 1、安装虚拟机系统

进入`devenv`开发环境目录下，执行命令行

```shell
cd /d/GoPath/src/github.com/hyperledger/fabric/devenv
$ vagrant.exe up
```

执行上述命令行，其实就是运行了`devenv`目录中的`Vagrantfile`脚本。有兴趣的可以去看看这个脚本，脚本里面会运行`setup.sh`脚本。`Vagrantfile`脚本会帮你配置网络、挂载之前配置好的`GoPath`下的`fabric`工程，也就是当前目录。还会安装`virtualbox`虚拟机、以及安装`docker`。

由于脚本中安装虚拟机会默认分配内存和cpu资源，如果你的机器配置比较低，可以适当的调整这里分配的资源。

```shell
config.vm.provider :virtualbox do |vb|
    vb.name = "hyperledger"
    vb.customize ['modifyvm', :id, '--memory', '4096']
    vb.cpus = 2
```

后续还会对这个脚本进行修改，挂载我们自己的开发的智能合约` chaincode`代码和官方的示例。



执行命令后，出现类似以下的提示

```shell
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Box 'ubuntu/xenial64' could not be found. Attempting to find and install...
    default: Box Provider: virtualbox
    default: Box Version: >= 0
==> default: Loading metadata for box 'ubuntu/xenial64'
    default: URL: https://vagrantcloud.com/ubuntu/xenial64
==> default: Adding box 'ubuntu/xenial64' (v20180410.0.0) for provider: virtualbox
    default: Downloading: https://vagrantcloud.com/ubuntu/boxes/xenial64/versions/20180410.0.0/providers/virtualbox.box
    default: Download redirected to host: cloud-images.ubuntu.com
    default:
==> default: Successfully added box 'ubuntu/xenial64' (v20180410.0.0) for 'virtualbox'!
==> default: Importing base box 'ubuntu/xenial64'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'ubuntu/xenial64' is up to date...
==> default: Setting the name of the VM: hyperledger
```

执行上面命令若没有问题，可以看到在下载`'ubuntu/xenial64'`系统。此时你的网络要保持通畅，建议在网络最好的时候执行，若网络不好可能中断或时间比较久。有可能需要翻墙软件进行翻墙，如果不能翻墙可以手动下载`ubnutu`系统。

> 以上命令行主要做的任务有：
> 下载 `ubuntu` 系统、下载`virtualbox.box`、创建`hyperledger`、配置网络、配置`ssh`、挂载`gopath`下的`hyperledger`工程、安装`couchdb`、下载安装`docker`、`docker-compose`、下载安装`go`语言、下载安装`python`语言等等，还有其他的操作，可以看日志



如果以上程序执行过程中发生中断或错误，可以先解决错误，随后运行命令`vagrant.exe provision`<br/>如果是下载docker太慢，也许会断掉或者龟速。这时候可能需要加速，使用国内的阿里加速下载`docker`，修改`GoPath/src/github.com/hyperledger/fabric/devenv`目录下的`setup.sh`脚本，加入如下代码

```shell
# ----------------------------------------------------------------
# Install Docker
# ----------------------------------------------------------------

add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu
$(lsb_release -cs) stable"
```



执行完成上面的命令后，可以打开本地的`Oracle VM VirtualBox.exe`，会发现本地的vbox软件的虚拟机列表中多出了一个`hyperledger`的虚拟机，再次执行命令`vagrant.exe ssh`成功连入虚拟机

```shell
$ vagrant.exe ssh
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-119-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.
```

看到上面的输出信息后，表明成功链接到Ubuntu系统



## 2、下载docker镜像

这里下载的docker镜像是`hyperledger fabric`必须的镜像，它是后面`fabric`运行的基础。

把下面的脚本内容编辑成一个可以执行的`shell`脚本

```shell
docker pull hyperledger/fabric-orderer:x86_64-1.0.3
docker rmi hyperledger/fabric-orderer:latest
docker tag hyperledger/fabric-orderer:x86_64-1.0.3 hyperledger/fabric-orderer:latest
docker pull hyperledger/fabric-ca:x86_64-1.0.3
docker rmi hyperledger/fabric-ca:latest
docker tag hyperledger/fabric-ca:x86_64-1.0.3 hyperledger/fabric-ca:latest
docker pull hyperledger/fabric-kafka:x86_64-1.0.3
docker rmi hyperledger/fabric-kafka:latest
docker tag hyperledger/fabric-kafka:x86_64-1.0.3 hyperledger/fabric-kafka:latest
docker pull hyperledger/fabric-zookeeper:x86_64-1.0.3
docker rmi hyperledger/fabric-zookeeper:latest
docker tag hyperledger/fabric-zookeeper:x86_64-1.0.3 hyperledger/fabric-zookeeper:latest
docker pull hyperledger/fabric-peer:x86_64-1.0.3
docker rmi hyperledger/fabric-peer:latest 
docker tag hyperledger/fabric-peer:x86_64-1.0.3 hyperledger/fabric-peer:latest
docker pull hyperledger/fabric-tools:x86_64-1.0.3
docker rmi hyperledger/fabric-tools:latest 
docker tag hyperledger/fabric-tools:x86_64-1.0.3 hyperledger/fabric-tools:latest
docker pull hyperledger/fabric-ccenv:x86_64-1.0.3
docker rmi hyperledger/fabric-ccenv:latest 
docker tag hyperledger/fabric-ccenv:x86_64-1.0.3 hyperledger/fabric-ccenv:x86_64-1.0.3
docker pull hyperledger/fabric-couchdb:x86_64-1.0.3
docker rmi hyperledger/fabric-couchdb:latest 
docker tag hyperledger/fabric-couchdb:x86_64-1.0.3 hyperledger/fabric-couchdb:latest
docker rmi hyperledger/fabric-baseos:x86_64-0.3.2 
docker pull hyperledger/fabric-baseos:x86_64-0.3.2 
docker tag hyperledger/fabric-baseos:x86_64-0.3.2 hyperledger/fabric-baseos:latest
docker rmi hyperledger/fabric-baseimage:x86_64-0.3.2 
docker pull hyperledger/fabric-baseimage:x86_64-0.3.2 
docker tag hyperledger/fabric-baseimage:x86_64-0.3.2 hyperledger/fabric-baseimage:latest
#https://registry-1.docker.io/v2/hyperledger/fabric-baseos
#docker rmi fabric-baseimage:latest
```

执行的命令行`vi download_docker_images.sh` 然后把上面的内容粘贴`shift + insert`到当前`vi`编辑的文件中,保存并退出。当你执行该脚本的时候，发现没有权限，这时候需要授权，输入命令行代码`chmod +x download_docker_images.sh`，最后执行该脚本 `sudo sh download_docker_images.sh`。

完整命令行如下：

```shell
$ vi download_docker_images.sh
$ chmod +x download_docker_images.sh
$ sudo sh download_docker_images.sh
```

如果最后一个命令不带`sudo`脚本将运行失败，因为当前用户权限不够，所以必须要。或者切换到`root`用户。



执行上面`download_docker_images.sh`命令期间，也许会断掉或者龟速。这时候可能需要加速，使用国内的阿里镜像仓库加速下载`docker images`命令行代码如下：

```shell
$ cd /etc/docker
$ vi daemon.json
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

在`daemon.json`中加入上面的配置即可加速



## 遇到的问题

### 1、若执行`vagrant.exe up`命令出现以下错误提醒，你需要升级`powershell`
---
```shell
$ vagrant.exe up
The version of powershell currently installed on this host is less than
the required minimum version. Please upgrade the installed version of
powershell to the minimum required version and run the command again.

  Installed version: 2

  Minimum required version: 3
```

这时候你需要安装更新，这个需要对应系统版本和当前`framework`的版本，我这里使用下载地址是：
https://www.microsoft.com/en-us/download/details.aspx?id=40855
下载`Windows6.1-KB2819745-x64-MultiPkg.msu`



### 2、若中途不小心关闭了终端命令行窗口，怎么再打开之前的虚拟机系统
---
> 这时当我们打开命令终端后，输入命令`vagrant ssh`发现有一段提示信息

```shell
$ vagrant.exe ssh
A Vagrant environment or target machine is required to run this
command. Run `vagrant init` to create a new Vagrant environment. Or,
get an ID of a target machine from `vagrant global-status` to run
this command on. A final option is to change to a directory with a
Vagrantfile and to try again.
```

> 这时候我们可以根据提示去做，下面开始演示



**方法1、** 执行命令行`vagrant global-status`，找到所有在运行的虚拟机系统

```shell
$ vagrant global-status
id       name    provider   state   directory
---------------------------------------------------------------------------------------
ccbaaf6  default virtualbox running D:/GoPath/src/github.com/hyperledger/fabric/devenv

The above shows information about all known Vagrant environments
on this machine. This data is cached and may not be completely
up-to-date. To interact with any of the machines, you can go to
that directory and run Vagrant, or you can use the ID directly
with Vagrant commands from any directory. For example:
"vagrant destroy 1a2b3c4d"
```

从上面的命令提示可以看到`vbox`的状态是`运行中`,已经当前虚拟机的目录位置
然后这些命令行`vagrant.exe ssh ccbaaf6` 和之前的不同是后面带了虚拟机的id，这个表示你需要进入那个虚拟机，执行命令后提示信息如下：

```shell
$ vagrant.exe ssh ccbaaf6
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-119-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.


Last login: Wed Apr 11 13:46:04 2018 from 10.0.2.2
```

> 看到以上信息表明成功连入之前的虚拟机系统



**方法2、** 直接进入虚拟机安装的位置`cd GoPath/src/github.com/hyperledger/fabric/devenv`，执行命令行`vagrant.exe ssh`

```shell
$ cd GoPath/src/github.com/hyperledger/fabric/devenv

$ vagrant.exe ssh
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-119-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.


Last login: Thu Apr 12 01:42:29 2018 from 10.0.2.2
```

