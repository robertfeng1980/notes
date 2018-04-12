# 在windows下搭建hyperledger fabric 开发环境

> hyperledger fabric 开发环境搭建是一项大工程，这里主要描述windows下的环境搭建。其他linux和mac不做描述。我的机器是win7 64位系统。



## 一、准备工作

### 1、命令终端工具的安装

### 2、Go 语言环境的安装

### 3、vagrant工具

> vagrant 工具就是一个用命令行和脚本的方式，帮你创建虚拟机和安装虚拟机里面的软件的一个工具。并且它支持把安装好的环境进行打包成`*.box`然后在其他机器上再通过vagrant很方便的添加到机器上。

#### 安装 vagrant

#### vagrant的基本命令

1、创建或初始化一个虚拟机环境，和`git`创建仓库有些类似

```shell
$ cd /box/work/mybox
$ vagrant init ubuntu/trusty64
```

> 执行上面的`init`命令后其实就是在是在当前目录下创建了一个`Vagrantfile`的文件。打开这个文件看看，里面有当前虚拟机的名称和一些注释掉的配置。
>
> `Vagrant`还会在`Vagrantfile`所在同级目录下创建一个`.vagrant`隐藏文件夹，该文件夹包含了在本地运行虚拟机的一些信息。如果使用了代码库管理（比如Git），这个`.vagrant`文件夹应该被`ignore`掉。同时这个目录还存在一个私钥`private_key`



2、如果已经有一个现有的`box`，那可以这样操作

```shell
$ vagrant box add my-box-file.box  --name mybox
```

> 上面的命令是在当前目录里有一个`my-box-file.box`的文件，添加到当前环境中，它的名称叫`mybox`

```shell
$ vagrant box add --name mybox http://pan.baidu.com/AsFS24D/my-box-file.box
```

> 添加一个box在远程服务器上，程序会自动到这个地址下载。然后添加这个box到虚拟机上名称为`mybox`

3、添加一个远程现有的`box`

```shell
$ vagrant init mybox https://boxes.company.com/my-project.box
```

> 上面的`mybox`是虚拟机名称，后面的网址是远程的虚拟机文件



4、启动虚拟机 `vagrant up`就可以启动当前虚拟机了，此时`vagrant`会检查本地是否有`ubuntu/trusty64`这个虚拟机的`trusty`的`ubuntu`版本（`Ubuntu Server 14.04 LTS (Trusty Tahr) daily builds`），如果没有就会从官网`https://atlas.hashicorp.com`来下载`ubuntu/trusty64`这个box，这个过程的速度看网络好坏。

```shell
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Box 'ubuntu/trusty64' could not be found. Attempting to find and install...
    default: Box Provider: virtualbox
    default: Box Version: >= 0
==> default: Loading metadata for box 'ubuntu/trusty64'
    default: URL: https://vagrantcloud.com/ubuntu/trusty64
==> default: Adding box 'ubuntu/trusty64' (v20180409.0.0) for provider: virtualbox
    default: Downloading: https://vagrantcloud.com/ubuntu/boxes/trusty64/versions/20180409.0.0/providers/virtualbox.box
    default: Download redirected to host: cloud-images.ubuntu.com
    default: Progress: 0% (Rate: 15769/s, Estimated time remaining: 10:00:35)
```

> 从上面的日志可以看到，程序会先去找`'ubuntu/trusty64'`没有找到，于是就去匹配`vagrant`虚拟镜像配置
> `URL: https://vagrantcloud.com/ubuntu/trusty64`，它会配置虚拟机的名称和使用的虚拟机软件。
> ```shell
> Vagrant.configure("2") do |config|
>   config.vm.box = "ubuntu/trusty64"
>   config.vm.box_version = "20180409.0.0"
> end
>
> 1 provider for this version.
> virtualbox Externally hosted (cloud-images.ubuntu.com)
> ```
> 下载`vagrant`的`box`文件：` Downloading: https://vagrantcloud.com/ubuntu/boxes/trusty64/versions/20180409.0.0/providers/virtualbox.box`
> 云镜像：` cloud-images.ubuntu.com`
> 上面的网址都可以打开看看，你就明白了`vagrant`是如何去下载、匹配文件的。
>
> `vagrant up`这个命令会默认使用`Virtualbox`，如果需要使用其他的虚拟机产品可以设置`vagrant up --provider hyperv` 这里的hyperv是一个虚拟机产品，如：`VMware`、`Virtualbox`



5、`ssh`连到虚拟机，`vagrant ssh`就可以直接连接到当前虚拟机。

> 如果当前目录没有`Vagrantfile`就无法进行连接，这是有两种办法，
>
> 一种是进入到虚拟机目录`/box/work/mybox`(存在`Vagrantfile`文件的目录)
>
> 另一种则是：
>
> ```shell
> $ vagrant.exe global-status
> id       name    provider   state   directory
> ---------------------------------------------------------------------------------------
> ccbaaf6  default virtualbox running /box/work/mybox
>
> The above shows information about all known Vagrant environments
> on this machine. This data is cached and may not be completely
> up-to-date. To interact with any of the machines, you can go to
> that directory and run Vagrant, or you can use the ID directly
> with Vagrant commands from any directory. For example:
> "vagrant destroy 1a2b3c4d"
> ```
>
> 找到当前启动虚拟机的信息，然后用`vagrant ssh ID`连入
>
> ```shell
> $ vagrant ssh ccbaaf6
> ```
> **此时vagrant将使用默认的用户vagrant以及预设的SSH公钥密钥键值对直接登录虚拟机。**



6、关闭虚拟机 `vagrant halt`



7、删除销毁虚拟机 `vagrant destroy` ，该操作只会删除虚拟机，不会删除虚拟机对应的`box`



8、添加和查看当前机器上安装或下载的`box`，在当前用户`Administrator`目录`C:\Users\Administrator\.vagrant.d\boxes\`

```shell
$ cd ~/.vagrant.d/boxes
```



9、列举出本地所有下载好的`box`文件信息

```shell
$ vagrant box list
ubuntu/xenial64 (virtualbox, 20180410.0.0)
```



10、删除本地的`box`信息 `vagrant box remove box-name`

11、端口映射转发，多用于在宿主主机或外部主机访问虚拟机不能访问的情况，解决方法如下：

```shell
Vagrant.configure("2") do |config|
  config.vm.network "forwarded_port", guest: 8080, host: 8888
end
```

> 在默认情况下，`Vagrant`所创建的`Virtualbox`虚拟机使用的是`NAT`网络类型，即外界是不能直接访问你的虚拟机的，就连宿主（装`Virtualbox`软件的这台机器）机器也访问不了。此时，如果你在虚拟机中启动的一个`Tomcat`来部署网站的测试环境，而又想外界能够访问的话，你需要使用端口转发。
>
> 通过上面的配置后，宿主主机的`8888`端口转发到了虚拟机的`8080`端口，这样你便可以通过在宿主主机上访问 http://localhost:8888 来访问虚拟机的`Tomcat`了。对于`Virtualbox`来说，只有NAT类型的网络类型支持端口转发，这也是为什么`Vagrant`创建的`Virtualbox`虚拟机默认都有一个支持NAT的虚拟网卡，原因就是要能够支持`Vagrant`级别的端口转发。另外，`Vagrant`在第一次尝试连接虚拟机时使用的也是NAT。

12、

### 4、Virtualbox 的安装

### 5、hyperledger fabric 源码的安装



## 二、搭建开发环境

> 由于上面已经准备好了环境的必要工具，下面全程用命令行的方式去安装虚拟机。

### 1、安装虚拟机系统

进入`devenv`开发环境目录下，执行命令行

```shell
cd /d/GoPath/src/github.com/hyperledger/fabric/devenv
$ vagrant.exe up
```

> 执行上述命令行，其实就是运行了`devenv`目录中的`Vagrantfile`脚本。有兴趣的可以去看看这个脚本，脚本里面会运行`setup.sh`脚本。`Vagrantfile`脚本会帮你配置网络、挂载之前配置好的`GoPath`下的`fabric`工程，也就是当前目录。还会安装`virtualbox`虚拟机、以及安装`docker`。由于脚本中安装虚拟机会默认分配内存和cpu资源，如果你的机器配置比较低，可以适当的调整这里分配的资源。
>
> ```shell
> config.vm.provider :virtualbox do |vb|
>     vb.name = "hyperledger"
>     vb.customize ['modifyvm', :id, '--memory', '4096']
>     vb.cpus = 2
> ```
>
> 后续还会对这个脚本进行修改，挂载我们自己的开发的智能合约` chaincode`代码和官方的示例。



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



如果以上程序执行过程中发生中断或错误，可以先解决错误，随后运行命令`vagrant.exe provision`

如果是下载docker太慢，也许会断掉或者龟速。这时候可能需要加速，使用国内的阿里加速下载`docker`，修改`GoPath/src/github.com/hyperledger/fabric/devenv`目录下的`setup.sh`脚本，加入如下代码

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

> 看到上面的输出信息后，表明成功链接到Ubuntu系统



### 2、下载docker镜像

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

> 如果最后一个命令不带`sudo`脚本将运行失败，因为当前用户权限不够，所以必须要。或者切换到`root`用户。



执行上面`download_docker_images.sh`命令期间，也许会断掉或者龟速。这时候可能需要加速，使用国内的阿里镜像仓库加速下载`docker images`命令行代码如下：

```shell
$ cd /etc/docker
$ vi daemon.json
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

> 在`daemon.json`中加入上面的配置即可加速



### 遇到的问题

#### 1、若执行`vagrant.exe up`命令出现以下错误提醒，你需要升级`powershell`

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



#### 2、若中途不小心关闭了终端命令行窗口，怎么再打开之前的虚拟机系统

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

