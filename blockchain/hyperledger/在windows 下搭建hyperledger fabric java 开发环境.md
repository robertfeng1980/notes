# 搭建Hyperledger Fabric Java SDK 开发环境

在前一篇 **在windows下搭建Hyperledger Fabric 开发环境**的基础上，搭建Java SDK开发环境。首先需要下载 `fabric-ca`源码进行安装挂载到虚拟机。

```sh
$ cd /d/GoPath/src/github.com/hyperledger
$ git clone https://github.com/hyperledger/fabric-ca.git

$ git checkout v1.1.0
```

打开文件`Vagrantfile`并验证是否设置了以`config.vm.network`语句。如果不是，则添加它们：

```ruby
$ cd ../fabric/devenv/
$ vi Vagrantfile

  config.vm.network :forwarded_port, guest: 7050, host: 7050 # fabric orderer service
  config.vm.network :forwarded_port, guest: 7051, host: 7051 # fabric peer vp0 service
  config.vm.network :forwarded_port, guest: 7053, host: 7053 # fabric peer event service
  config.vm.network :forwarded_port, guest: 7054, host: 7054 # fabric-ca service
  config.vm.network :forwarded_port, guest: 5984, host: 15984 # CouchDB service
  ### Below are probably missing.....
  config.vm.network :forwarded_port, guest: 7056, host: 7056
  config.vm.network :forwarded_port, guest: 7058, host: 7058
  config.vm.network :forwarded_port, guest: 8051, host: 8051
  config.vm.network :forwarded_port, guest: 8053, host: 8053
  config.vm.network :forwarded_port, guest: 8054, host: 8054
  config.vm.network :forwarded_port, guest: 8056, host: 8056
  config.vm.network :forwarded_port, guest: 8058, host: 8058
  config.vm.network :forwarded_port, guest: 7059, host: 7059
  # docker
  config.vm.network :forwarded_port, guest: 2333, host: 2333, protocol: "tcp" # dockerd
```

添加到`Vagrantfile`文件夹以引用下面几行之间的sdk集成文件夹：

```ruby
config.vm.synced_folder "..", "/opt/gopath/src/github.com/hyperledger/fabric"
# 对应 fabric-sdk-java目录中的sdkintegration
config.vm.synced_folder "/home/<<user>>/fabric-sdk-java/src/test/fixture/sdkintegration", "/opt/gopath/src/github.com/hyperledger/fabric/sdkintegration"
```

`ssh`链接到开发机器后设置 `dockerd` 守护进程链接的套接字地址，这样本地就可以访问虚拟机中的`docker` 容器

```sh
# 停止docker守护进程
$ sudo service docker stop

# 启动 docker守护进程，在后台常驻
$ sudo dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2333 &

# 启动 并开启debug模式
$ sudo dockerd --debug -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2333

$ ps -ef | grep docker
$ ps aux | grep dockerd
$ kill -9 {pid}
```

