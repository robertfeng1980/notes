# 在windows下搭建hyperledger fabric 开发环境

> hyperledger fabric 开发环境搭建是一项大工程，这里主要描述windows下的环境搭建。其他linux和mac不做描述。我的机器是win7 64位系统。



## 一、准备工作

### 1、命令终端工具的安装

### 2、Go 语言

- [ ] 下载安装文件：https://golang.org/dl/
- [ ] 配置环境变量，在电脑系统变量中进行新建配置
  - `go` 安装目录环境变量配置：`GOROOT`=`E:\Go`
  - `go` 工作目录环境变量配置：`GOPATH`=`D:\GOPATH`
    - 在`D:\GOPATH`创建目录`src`，`bin`，`pkg`
    - 在`src`目录创建`github.com\hyperledger` 这个目录后面会放`hyperledger fabric`的代码

### 3、vagrant工具

> vagrant 工具就是一个用命令行和脚本的方式，帮你创建虚拟机和安装虚拟机里面的软件的一个工具。并且它支持把安装好的环境进行打包成`*.box`然后在其他机器上再通过vagrant很方便的添加到机器上。

#### 安装 vagrant

- [ ] 下载安装文件：https://www.vagrantup.com/downloads.html

- [ ] 安装完成后重启电脑

  ​

#### vagrant的基本命令

##### 基本命令

***介绍`box`的创建、启动、关闭、删除、连接***



+ **创建或初始化一个虚拟机环境，和`git`创建仓库有些类似**

    ```shell
    $ cd /box/work/mybox
    $ vagrant init ubuntu/trusty64
    ```

    > 执行上面的`init`命令后其实就是在是在当前目录下创建了一个`Vagrantfile`的文件。打开这个文件看看，里面有当前虚拟机的名称和一些注释掉的配置。
    >
    > `Vagrant`还会在`Vagrantfile`所在同级目录下创建一个`.vagrant`隐藏文件夹，该文件夹包含了在本地运行虚拟机的一些信息。如果使用了代码库管理（比如Git），这个`.vagrant`文件夹应该被`ignore`掉。同时这个目录还存在一个私钥`private_key`

    ​



+ **如果已经有一个现有的`box`，那可以这样操作**

    ```shell
    $ vagrant box add my-box-file.box  --name mybox
    ```

    > 上面的命令是在当前目录里有一个`my-box-file.box`的文件，添加到当前环境中，它的名称叫`mybox`

    ```shell
    $ vagrant box add --name mybox http://pan.baidu.com/AsFS24D/my-box-file.box
    ```

    > 添加一个box在远程服务器上，程序会自动到这个地址下载。然后添加这个box到虚拟机上名称为`mybox`

    ​



+  **添加一个远程现有的`box` **

    ```shell
    $ vagrant init mybox https://boxes.company.com/my-project.box
    ```

    > 上面的`mybox`是虚拟机名称，后面的网址是远程的虚拟机文件

    ​



+ **启动虚拟机 `vagrant up`就可以启动当前虚拟机了**，此时`vagrant`会检查本地是否有`ubuntu/trusty64`这个虚拟机的`trusty`的`ubuntu`版本（`Ubuntu Server 14.04 LTS (Trusty Tahr) daily builds`），如果没有就会从官网`https://atlas.hashicorp.com`来下载`ubuntu/trusty64`这个box，这个过程的速度看网络好坏。

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

    ​



+ **`ssh`连到虚拟机**，`vagrant ssh`就可以直接连接到当前虚拟机。

    > 如果当前目录没有`Vagrantfile`就无法进行连接，这是有两种办法，
    > 一种是进入到虚拟机目录`/box/work/mybox`(存在`Vagrantfile`文件的目录)
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



+ **关闭虚拟机 `vagrant halt`**，关闭正在运行的虚拟机，当你的工作目录不在虚拟机目录下 ` vagrant.exe halt ID`



+ **删除销毁虚拟机 `vagrant destroy` ，该操作只会删除虚拟机，不会删除虚拟机对应的`box`**





##### 高级命令

- **添加和查看当前机器上安装或下载的`box`**，在当前用户`Administrator`目录`C:\Users\Administrator\.vagrant.d\boxes\`

    ```shell
    $ cd ~/.vagrant.d/boxes
    ```



- **列举出本地所有下载好的`box`文件信息**

    ```shell
    $ vagrant box list
    ubuntu/xenial64 (virtualbox, 20180410.0.0)
    ```



- **删除本地的`box`信息** `vagrant box remove box-name`


- **查看正在运行的虚拟机** `vagrant global-status`





##### 配置选项

+ **端口映射转发**，多用于在宿主主机或外部主机访问虚拟机不能访问的情况，解决方法如下，在`Vagrantfile`中加入配置：

    ```shell
    Vagrant.configure("2") do |config|
      config.vm.network "forwarded_port", guest: 8080, host: 8888
    end
    ```

    > 在默认情况下，`Vagrant`所创建的`Virtualbox`虚拟机使用的是`NAT`网络类型，即外界是不能直接访问你的虚拟机的，就连宿主（装`Virtualbox`软件的这台机器）机器也访问不了。此时，如果你在虚拟机中启动的一个`Tomcat`来部署网站的测试环境，而又想外界能够访问的话，你需要使用端口转发。
    >
    > 通过上面的配置后，宿主主机的`8888`端口转发到了虚拟机的`8080`端口，这样你便可以通过在宿主主机上访问 http://localhost:8888 来访问虚拟机的`Tomcat`了。对于`Virtualbox`来说，只有NAT类型的网络类型支持端口转发，这也是为什么`Vagrant`创建的`Virtualbox`虚拟机默认都有一个支持NAT的虚拟网卡，原因就是要能够支持`Vagrant`级别的端口转发。另外，`Vagrant`在第一次尝试连接虚拟机时使用的也是NAT。



+ **共享挂载目录到虚拟机上**，在`Vagrantfile`中加入配置：

    ```shell
    Vagrant.configure("2") do |config|
      config.vm.synced_folder "images/", "/home/files/images"
    end
    ```

    > 上面的配置是将当前`Vagrantfile`所在目录下的`images`挂载在虚拟机`/home/files/images`上



+ `provision` 规定虚拟机运行一段特定的 **命令或脚本**。简单地说，`provision`即通过使用某些工具自动地、批量地为机器安装软件以及配置系统，它省去了人工安装和配置系统时的重复性和易错性，当然还享受了计算机与生俱来的速度。Vagrant提供多种方式对虚拟机进行`provision`，包括`Shell`、`Chef`、`Puppet`和`Ansible`等。以`Shell`为例，既可以通过直接在`Vagrantfile`中编写`Shell`脚本的方式，也可以通过引用外部Shell文件的方式。

    ```shell
    Vagrant.configure("2") do |config|
      config.vm.provision "shell", inline: "echo hello"
    end

    # 多行命令
    config.vm.provision "shell", inline: <<-SHELL
         apt-get update
         apt-get install -y apache2
    SHELL
    ```

    > 上面的配置是系统在执行`vagrant provision`的时候就会输出 `hello`

    ```shell
    Vagrant.configure("2") do |config|
      config.vm.provision "shell", path: "script.sh"
    end
    ```
    > 引用外部脚本 ` "script.sh"`


​    

- 下面介绍下  **`ansible` 批量自动化任务**

  > ansible是新出现的自动化运维工具，基于Python开发，集合了众多运维工具（puppet、cfengine、chef、func、fabric）的优点，实现了批量系统配置、批量程序部署、批量运行命令等功能。
  >
  > ansible是基于模块工作的，本身没有批量部署的能力。真正具有批量部署的是ansible所运行的模块，ansible只是提供一种框架。主要包括：
  >
  > (1)、连接插件connection plugins：负责和被监控端实现通信；
  >
  > (2)、host inventory：指定操作的主机，是一个配置文件里面定义监控的主机；
  >
  > (3)、各种模块核心模块、command模块、自定义模块；
  >
  > (4)、借助于插件完成记录日志邮件等功能；
  >
  > (5)、playbook：剧本执行多个任务时，非必需可以让节点一次性运行多个任务；
  >
  > ​
  >
  > ansible 使用指南：http://www.ansible.com.cn/index.html
  >
  > ​
  >
  > ansible 文档：https://yq.aliyun.com/articles/86760?t=t1

  ​

  在使用`ansible`批量自动化任务时，有两种方式：

  （1）在宿主主机机器上安装`ansible`

  （2）采用`ansible local`的方式，即在虚拟机自身上安装`ansible`

  对于第（1）种方法，我们需要保证宿主主机机器上已经安装了`ansible`，然后进行配置：

  ```shell
  Vagrant.configure("2") do |config|

  	# Run Ansible from the Vagrant Host
  	config.vm.provision "ansible" do |ansible|
  		ansible.playbook = "playbook.yml"
  	end
  end
  ```

  在使用第（2）种方法时，`Vagrant`会首先检查box中是否已经安装了`ansible`，如果没有，则会自动安装到虚拟机上，然后再运行`provision`：

  ```shell
  Vagrant.configure("2") do |config|
  	# Run Ansible from the Vagrant VM
  	config.vm.provision "ansible_local" do |ansible|
  		ansible.playbook = "playbook.yml"
  	end
  end
  ```

  > 当我们多次执行`vagrant up`启动虚拟机时，`provison`并不会每次都执行，只有在这三种情况下provision才会运行：
  >
  > 首次执行`vagrant up`
  >
  > 执行`vagrant provision`
  >
  > 执行`vagrant reload --provision`

  ​

  当然，你也可以在`Vagrantfile`中配置成每次执行`vagrant up`时都运行`provision`：

  ```shell
  Vagrant.configure("2") do |config|
  	config.vm.provision "shell", inline: "echo hello",
  	run: "always"
  end
  ```

  ​


+ 配置私有固定`ip`网络

    ```shell
    Vagrant.configure("2") do |config|
      config.vm.network "private_network", ip: "192.168.33.77"
    end
    ```

+ 配置动态`ip`使用`DHCP`的方式动态分配`ip`

  ```shell
  Vagrant.configure("2") do |config|
    config.vm.network "private_network", type: "dhcp"
  end
  ```

+ 配置共有的固定`ip`网络

    ```shell
    Vagrant.configure("2") do |config|
      config.vm.network "public_network", ip: "192.168.1.15"
    end
    ```

+ 配置虚拟机主机源`Provider`

    不同的`Provider`有不同的特性，也存在不同的配置方式。以`Virtualbox`为例，`Vagrant`默认会给虚拟机指定一个不具备可读性的名字，比如`mybox_default_1523517449396_15013`，我们可以对此进行配置予以更改：

    ```shell
    config.vm.provider "virtualbox" do |v|
      v.name = "mybox"
    end
    ```

     

    `Provider`的特定配置也可以覆盖`Vagrant`原来的配置：

    ```shell
    Vagrant.configure("2") do |config|
      config.vm.box = "precise64"

      config.vm.provider "vmware_fusion" do |v, override|
        override.vm.box = "precise64_fusion"
      end
    end
    ```



##### 打包`box`

> 打包`box`可以让之前安装好的环境重复使用，比如将自己机器的工作环境打包成一个`box`，以后去其他公司或者网络封闭的场所时，把这个打包好的box直接添加到新机器上直接就可以使用了。而不需要重复安装一些软件，这样避免了错误也省去了大量的时间精力，毕竟安装软件是一些苦力活，对网络的也有要求。

```shell
$ vagrant halt
# 这里的mybox是虚拟机的名称，在 Vagrantfile 有指定虚拟机的情况下可以这样使用
$ vagrant package --base mybox
```

> 这样就创建了一个打包，打包完成后可以拷贝到其他地方使用。注意在打包之前要关闭虚拟机。

如果当前打包的目录是在`Vagrantfile`所在目录(这样当前命令就直接指向了当前配置对应的虚拟机)，那就可以直接打包

```shell
$ vagrant halt
$ vagrant package
```



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

