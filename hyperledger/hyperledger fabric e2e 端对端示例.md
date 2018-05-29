# HyperLedger Fabric `end-to-end`示例流程

* [Hyperledger Fabric end\-to\-end示例流程](#hyperledger-fabric-end-to-end%E7%A4%BA%E4%BE%8B%E6%B5%81%E7%A8%8B)
* [准备工作](#%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C)
* [设置环境变量](#%E8%AE%BE%E7%BD%AE%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F)
* [安装Hyperledger源码](#%E5%AE%89%E8%A3%85hyperledger%E6%BA%90%E7%A0%81)
* [获取二进制文件和镜像](#%E8%8E%B7%E5%8F%96%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%96%87%E4%BB%B6%E5%92%8C%E9%95%9C%E5%83%8F)
  * [方案一、通过makefile编译生成](#%E6%96%B9%E6%A1%88%E4%B8%80%E9%80%9A%E8%BF%87makefile%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90)
    * [生成 二进制 文件](#%E7%94%9F%E6%88%90-%E4%BA%8C%E8%BF%9B%E5%88%B6-%E6%96%87%E4%BB%B6)
    * [生成 docker 镜像](#%E7%94%9F%E6%88%90-docker-%E9%95%9C%E5%83%8F)
  * [方案二、通过HTTP下载远程的文件](#%E6%96%B9%E6%A1%88%E4%BA%8C%E9%80%9A%E8%BF%87http%E4%B8%8B%E8%BD%BD%E8%BF%9C%E7%A8%8B%E7%9A%84%E6%96%87%E4%BB%B6)
    * [获取 二进制文件](#%E8%8E%B7%E5%8F%96-%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%96%87%E4%BB%B6)
    * [获取 docker 镜像](#%E8%8E%B7%E5%8F%96-docker-%E9%95%9C%E5%83%8F)
* [cryptogen 工具](#cryptogen-%E5%B7%A5%E5%85%B7)
  * [它是如何工作的？](#%E5%AE%83%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84)
  * [cryptogen 基本命令](#cryptogen-%E5%9F%BA%E6%9C%AC%E5%91%BD%E4%BB%A4)
    * [generate 生成加密文件](#generate-%E7%94%9F%E6%88%90%E5%8A%A0%E5%AF%86%E6%96%87%E4%BB%B6)
    * [extend 扩展现有网络](#extend-%E6%89%A9%E5%B1%95%E7%8E%B0%E6%9C%89%E7%BD%91%E7%BB%9C)
* [configtxgen 工具](#configtxgen-%E5%B7%A5%E5%85%B7)
  * [它是如何工作的？](#%E5%AE%83%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84-1)
  * [configtxgen 基本命令](#configtxgen-%E5%9F%BA%E6%9C%AC%E5%91%BD%E4%BB%A4)
* [运行shell脚本](#%E8%BF%90%E8%A1%8Cshell%E8%84%9A%E6%9C%AC)
  * [手动生成配置文件](#%E6%89%8B%E5%8A%A8%E7%94%9F%E6%88%90%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
    * [设置平台类型](#%E8%AE%BE%E7%BD%AE%E5%B9%B3%E5%8F%B0%E7%B1%BB%E5%9E%8B)
    * [生成加密证书](#%E7%94%9F%E6%88%90%E5%8A%A0%E5%AF%86%E8%AF%81%E4%B9%A6)
    * [生成创世块](#%E7%94%9F%E6%88%90%E5%88%9B%E4%B8%96%E5%9D%97)
    * [创建通道配置](#%E5%88%9B%E5%BB%BA%E9%80%9A%E9%81%93%E9%85%8D%E7%BD%AE)
    * [创建anchor peer配置](#%E5%88%9B%E5%BB%BAanchor-peer%E9%85%8D%E7%BD%AE)
* [使用 docker 运行 e2e示例](#%E4%BD%BF%E7%94%A8-docker-%E8%BF%90%E8%A1%8C-e2e%E7%A4%BA%E4%BE%8B)
  * [后台程序流程](#%E5%90%8E%E5%8F%B0%E7%A8%8B%E5%BA%8F%E6%B5%81%E7%A8%8B)
  * [说明了什么](#%E8%AF%B4%E6%98%8E%E4%BA%86%E4%BB%80%E4%B9%88)
  * [如何查看交易](#%E5%A6%82%E4%BD%95%E6%9F%A5%E7%9C%8B%E4%BA%A4%E6%98%93)
    * [查看docker CLI容器日志](#%E6%9F%A5%E7%9C%8Bdocker-cli%E5%AE%B9%E5%99%A8%E6%97%A5%E5%BF%97)
    * [查看智能合约链码容器日志](#%E6%9F%A5%E7%9C%8B%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%E9%93%BE%E7%A0%81%E5%AE%B9%E5%99%A8%E6%97%A5%E5%BF%97)
  * [一键运行e2e示例](#%E4%B8%80%E9%94%AE%E8%BF%90%E8%A1%8Ce2e%E7%A4%BA%E4%BE%8B)
* [了解 docker\-compose服务编排](#%E4%BA%86%E8%A7%A3-docker-compose%E6%9C%8D%E5%8A%A1%E7%BC%96%E6%8E%92)
* [手动运行命令](#%E6%89%8B%E5%8A%A8%E8%BF%90%E8%A1%8C%E5%91%BD%E4%BB%A4)
  * [退出容器](#%E9%80%80%E5%87%BA%E5%AE%B9%E5%99%A8)
  * [删除链码容器](#%E5%88%A0%E9%99%A4%E9%93%BE%E7%A0%81%E5%AE%B9%E5%99%A8)
  * [生成交易配置文件](#%E7%94%9F%E6%88%90%E4%BA%A4%E6%98%93%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
  * [修改 docker\-compose文件并启动](#%E4%BF%AE%E6%94%B9-docker-compose%E6%96%87%E4%BB%B6%E5%B9%B6%E5%90%AF%E5%8A%A8)
  * [命令语法参考](#%E5%91%BD%E4%BB%A4%E8%AF%AD%E6%B3%95%E5%8F%82%E8%80%83)
  * [创建通道](#%E5%88%9B%E5%BB%BA%E9%80%9A%E9%81%93)
  * [加入通道](#%E5%8A%A0%E5%85%A5%E9%80%9A%E9%81%93)
  * [安装链码](#%E5%AE%89%E8%A3%85%E9%93%BE%E7%A0%81)
  * [实例化链码](#%E5%AE%9E%E4%BE%8B%E5%8C%96%E9%93%BE%E7%A0%81)
  * [调用链码](#%E8%B0%83%E7%94%A8%E9%93%BE%E7%A0%81)
  * [查询链码](#%E6%9F%A5%E8%AF%A2%E9%93%BE%E7%A0%81)
* [使用 CouchDB](#%E4%BD%BF%E7%94%A8-couchdb)
  * [手动运行命令](#%E6%89%8B%E5%8A%A8%E8%BF%90%E8%A1%8C%E5%91%BD%E4%BB%A4-1)
    * [创建和加入通道](#%E5%88%9B%E5%BB%BA%E5%92%8C%E5%8A%A0%E5%85%A5%E9%80%9A%E9%81%93)
    * [安装和实例化链码](#%E5%AE%89%E8%A3%85%E5%92%8C%E5%AE%9E%E4%BE%8B%E5%8C%96%E9%93%BE%E7%A0%81)
    * [调用链码](#%E8%B0%83%E7%94%A8%E9%93%BE%E7%A0%81-1)
    * [查询与检索](#%E6%9F%A5%E8%AF%A2%E4%B8%8E%E6%A3%80%E7%B4%A2)
  * [关于数据持久性的注意事项](#%E5%85%B3%E4%BA%8E%E6%95%B0%E6%8D%AE%E6%8C%81%E4%B9%85%E6%80%A7%E7%9A%84%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)
* [故障排除](#%E6%95%85%E9%9A%9C%E6%8E%92%E9%99%A4)
  * [启动失败问题排除](#%E5%90%AF%E5%8A%A8%E5%A4%B1%E8%B4%A5%E9%97%AE%E9%A2%98%E6%8E%92%E9%99%A4)
  * [启动网络服务](#%E5%90%AF%E5%8A%A8%E7%BD%91%E7%BB%9C%E6%9C%8D%E5%8A%A1)



端到端验证规定了一个由**两个组织**组成的样本Fabric网络，**每个组织都维护两个对等点**以及一个基于`Kafka`的**共识**服务。

本网络示例需要两个基本工具的使用，这两个工具主要用于**数字签名认证和访问权限控制**，这在**交易网络**中是必须的：

+ **cryptogen** 生成用于识别和验证网络中各种组件的x509证书
+ **configtxgen** 为**`Ordered` 共识排序**引导程序和**创建通道**生成必要的配置文件

这两个工具都有自己独立的`yaml`配置，在这个配置文件中我们指定了我们网络的拓扑结构（`cryptogen`）以及我们用于各种配置操作（`configtxgen`）的证书的位置。 要想启动 `e2e`，必须利用上面的工具和配置生成相关的证书和文件，方可启动 `e2e` 网络示例。

# 准备工作

在开始本示例之前，需要你安装好以下软件：

+ `Git` 下载代码和相关组件
+ `Git Bash` Windows 用户需要
+ `Docker` v1.12或更高版本 
+ `Docker Compose`  v1.8或更高版本 
+ `Docker Toolbox` 仅限Windows用户 
+ `Go` 1.9或更高版本

具体操作安装可以参考文档：[在windows下搭建hyperledger fabric 开发环境](https://github.com/hooj0/notes/blob/master/hyperledger/%E5%9C%A8windows%E4%B8%8B%E6%90%AD%E5%BB%BAhyperledger%20fabric%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83.md)

如果你的机器不是在本地开发的环境，而是远程的虚拟机，则需要单独到虚拟机上安装以上列举的部分软件。

# 设置环境变量

设置 `Go`环境变量，编辑文件 `vi /etc/profile`，加入配置。

```sh
export GOPATH="/opt/gopath"
export GOROOT="/opt/go"
export PATH=$GOROOT/bin:$GOPATH/bin:$PATH
```

随后执行` sudo source /etc/profile `，让配置生效。

# 安装`Hyperledger`源码

创建目录，进入`gopath`，使用`Git`下载源代码。已经安装下载的跳过本步骤。

```sh
$ mkdir -p $GOPATH/src/github.com/hyperledger
$ cd $GOPATH/src/github.com/hyperledger

$ git clone https://github.com/hyperledger/fabric.git
$ git tag
$ git checkout v1.1.0
```

# 获取二进制文件和镜像

上面提到了`cryptogen`和`configtxgen`这两个工具，由于下载的源码中没有这两个文件，需要利用源码中的`Makefile`进行编译生成这两个工具。

## 方案一、通过`makefile`编译生成

### 生成 二进制 文件

---

现在为`cryptogen`和`configtxgen`制作特定于平台的二进制文件。

```sh
$ cd $GOPATH/src/github.com/hyperledger/fabric
# 生成的文件将在release目录中
$ make release
```

安装 `tree`工具查看目录结构，可以看到`linux-amd64` 目录下的工具文件已经生成。

```sh
$ sudo apt-get install -y tree

$ tree /opt/gopath/src/github.com/hyperledger/fabric/release
/opt/gopath/src/github.com/hyperledger/fabric/release
├── linux-amd64  ## 生成的目录
│   └── bin
│       ├── configtxgen	#--> 生成交易文件和区块文件
│       ├── configtxlator	#--> 支持独立于SDK来进行重新配置、修改交易和区块文件
│       ├── cryptogen	#--> 生成身份证书
│       ├── get-docker-images.sh	#--> 下载docker 镜像
│       ├── orderer		  #--> orderer 共识排序服务
│       └── peer		  #--> peer 节点操作
└── templates
    └── get-docker-images.in
```

可以看到上面生成了不止`cryptogen`和`configtxgen`文件，这些文件非常重要。

### 生成 docker 镜像

---

接下来，制作Fabric图像。这通常需要5到10分钟，所以耐心等待：

```sh
$ cd $GOPATH/src/github.com/hyperledger/fabric
$ make docker
```

如果您未能编译docker镜像，则可以执行`make clean` 然后再次执行`make docker`。 

生成好的镜像后，执行 `docker images`命令会有如下镜像。

```sh
$ docker images
hyperledger/fabric-ca                                                                                  latest              72617b4fa9b4        2 months ago        299MB
hyperledger/fabric-ca                                                                                  x86_64-1.1.0        72617b4fa9b4        2 months ago        299MB
hyperledger/fabric-tools                                                                               latest              b7bfddf508bc        2 months ago        1.46GB
hyperledger/fabric-tools                                                                               x86_64-1.1.0        b7bfddf508bc        2 months ago        1.46GB
hyperledger/fabric-orderer                                                                             latest              ce0c810df36a        2 months ago        180MB
hyperledger/fabric-orderer                                                                             x86_64-1.1.0        ce0c810df36a        2 months ago        180MB
hyperledger/fabric-peer                                                                                latest              b023f9be0771        2 months ago        187MB
hyperledger/fabric-peer                                                                                x86_64-1.1.0        b023f9be0771        2 months ago        187MB
hyperledger/fabric-javaenv                                                                             latest              82098abb1a17        2 months ago        1.52GB
hyperledger/fabric-javaenv                                                                             x86_64-1.1.0        82098abb1a17        2 months ago        1.52GB
hyperledger/fabric-ccenv                                                                               latest              c8b4909d8d46        2 months ago        1.39GB
hyperledger/fabric-ccenv                                                                               x86_64-1.1.0        c8b4909d8d46        2 months ago        1.39GB
hyperledger/fabric-baseimage                                                                           latest              dbe6787b5747        3 months ago        1.37GB
hyperledger/fabric-baseimage                                                                           x86_64-0.4.6        dbe6787b5747        3 months ago        1.37GB
hyperledger/fabric-zookeeper                                                                           latest              92cbb952b6f8        3 months ago        1.39GB
hyperledger/fabric-zookeeper                                                                           x86_64-0.4.6        92cbb952b6f8        3 months ago        1.39GB
hyperledger/fabric-kafka                                                                               latest              554c591b86a8        3 months ago        1.4GB
hyperledger/fabric-kafka                                                                               x86_64-0.4.6        554c591b86a8        3 months ago        1.4GB
hyperledger/fabric-couchdb                                                                             latest              7e73c828fc5b        3 months ago        1.56GB
hyperledger/fabric-couchdb                                                                             x86_64-0.4.6        7e73c828fc5b        3 months ago        1.56GB
hyperledger/fabric-baseos                                                                              latest              220e5cf3fb7f        3 months ago        151MB
hyperledger/fabric-baseos                                                                              x86_64-0.4.6        220e5cf3fb7f        3 months ago        151MB
```

如果无法生成镜像，可以执行刚刚生成的`get-docker-images.sh`脚本

```sh
$ cd /opt/gopath/src/github.com/hyperledger/fabric/release/linux-amd64/bin
$ sudo ./get-docker-images.sh
```

## 方案二、通过HTTP下载远程的文件

### 获取 二进制文件

---

进入到 `fabric/scripts`目录，查看`bootstrap.sh`脚本可得到下载二进制文件的代码，下载的版本号设置为 `1.1.0`。查看 `bootstrap.sh`文件内容，发现下载二进制文件的url。

```sh
echo "===> Downloading platform specific fabric binaries"
curl https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric/hyperledger-fabric/${ARCH}-${VERSION}/hyperledger-fabric-${ARCH}-${VERSION}.tar.gz | tar xz

echo "===> Downloading platform specific fabric-ca-client binary"
curl https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric-ca/hyperledger-fabric-ca/${ARCH}-${VERSION}/hyperledger-fabric-ca-${ARCH}-${VERSION}.tar.gz
```

将上面的 `${ARCH}-${VERSION}`变量用 `linux-amd64-1.1.0` 替换即可。这里的 `linux-amd64-1.1.0` 是系统类型和当前`fabric`代码的版本。

最终得到的下载二进制文件URL：[点击下载](https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric/hyperledger-fabric/linux-amd64-1.1.0/hyperledger-fabric-linux-amd64-1.1.0.tar.gz) <br/>通过这个地址就可以下载到一个压缩包，解压后得到二进制文件 `bin` 和 `yaml`配置 `config`。

CA 证书客户端二进制文件URL：[点击下载](https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric-ca/hyperledger-fabric-ca/linux-amd64-1.1.0/hyperledger-fabric-ca-linux-amd64-1.1.0.tar.gz) <br/>这个文件目前认为是CA客户端文件，后期在 CA方面应该有需要使用。

### 获取 docker 镜像

---

如果觉得上面的太繁琐，可以直接运行下面的命令即可。

```sh
$ cd /opt/gopath/src/github.com/hyperledger/fabric/scripts
$ sudo ./bootstrap.sh 1.1.0
```

执行上面的脚本后，不仅在当前 `scripts`目录生成我们需要的二进制文件还会下载好需要的 docker 镜像文件。

# cryptogen 工具

`cryptogen`工具为网络实体生成秘钥资料（`x509`证书）。这些证书基于标准的`PKI`实现，通过**达成共同的信任**实现`anchor`节点验证。 

## 它是如何工作的？

`cryptogen` 使用包含网络拓扑示例的文件  `crypto-config.yaml`，并允许我们为**组织**和属于这些组织的组件**生成证书库**。每个组织都配备了一个**唯一的根证书**（`ca-cert`），将特定组件（`Peer` 和`Ordered`）绑定到该组织。`fabric`中的交易和通信由实体的私钥（`keystore`）进行签名，然后通过公钥（`signcerts`）进行验证。你会注意到这个文件中有一个`count`变量，我们用它来指定每个组织的**对等节点Peer数量**。

```yaml
# ---------------------------------------------------------------------------
# "OrdererOrgs" - 管理orderer节点的组织的定义
# ---------------------------------------------------------------------------
OrdererOrgs:
  # ---------------------------------------------------------------------------
  # Orderer
  # ---------------------------------------------------------------------------
  - Name: Orderer
    Domain: example.com
    CA:
        Country: US
        Province: California
        Locality: San Francisco
    # ---------------------------------------------------------------------------
    # "Specs" - 请参阅下面的PeerOrgs以获取完整说明
    # ---------------------------------------------------------------------------
    Specs:
      - Hostname: orderer
# ---------------------------------------------------------------------------
# "PeerOrgs" - 管理peer对等节点的组织的定义
# ---------------------------------------------------------------------------
PeerOrgs:
  # ---------------------------------------------------------------------------
  # Org1
  # ---------------------------------------------------------------------------
  - Name: Org1
    Domain: org1.example.com
    EnableNodeOUs: true
    CA:
        Country: US
        Province: California
        Locality: San Francisco
    # ---------------------------------------------------------------------------
    # "Specs"
    # ---------------------------------------------------------------------------
    # 取消注释此部分以启用配置中主机的显式定义。
    # 大多数用户都希望使用Template，下面的Specs是一组Spec条目。
    # 每个Spec条目由两个字段组成：
    #   - Hostname:  （必需）所需的主机名称，无域名。
    #   - CommonName: （可选）指定CN的模板或显式覆盖。默认情况下，这是模板："{{.Hostname}}.{{.Domain}}"
    # ---------------------------------------------------------------------------
    # Specs:
    #   - Hostname: foo # 隐式地“foo.org1.example.com”
    #     CommonName: foo27.org5.example.com # 覆盖上面设置的基于主机名的foo
    #   - Hostname: bar
    #   - Hostname: baz
    # ---------------------------------------------------------------------------
    # "Template"
    # ---------------------------------------------------------------------------
    # 允许定义从模板开始依次创建的一个或多个主机。
    # 默认情况下，这看起来像从0到Count-1的“peer％d”。
    # 您可以覆盖节点数（Count），起始索引（Start）或用于构造名称的模板（Hostname）。
    #  
    # 注意：模板和规格不是互斥的。您可以定义这两个部分，并为您创建聚合节点。注意名称的碰撞
    # ---------------------------------------------------------------------------
    Template:
      Count: 2
      # Start: 5
      # Hostname: {{.Prefix}}{{.Index}} # default 隐式地“{{.Prefix}}{{.Index}}.org1.example.com”
    # ---------------------------------------------------------------------------
    # "Users"
    # ---------------------------------------------------------------------------
    # Count: The number of user accounts _in addition_ to Admin
    # ---------------------------------------------------------------------------
    Users:
      Count: 1
  # ---------------------------------------------------------------------------
  # Org2: See "Org1" for full specification
  # ---------------------------------------------------------------------------
  - Name: Org2
    Domain: org2.example.com
    EnableNodeOUs: true
    CA:
        Country: US
        Province: California
        Locality: San Francisco
    Template:
      Count: 2
    Users:
      Count: 1
```

运行该工具后，证书将被放在一个名为`crypto-config`的文件夹中。使用命令`tree crypto-config`查看目录结构。

## cryptogen 基本命令

```sh
$ cryptogen --help
usage: cryptogen [<flags>] <command> [<args> ...]
用于生成Hyperledger Fabric密钥材料的实用程序

Commands:
  generate [<flags>] 	# 生成密钥材料   
  showtemplate			# 显示默认的配置模板   
  extend [<flags>]		# 扩展存在的网络    
```

### generate 生成加密文件

---

```sh
# 生成密钥材料
$ cryptogen generate --help
usage: cryptogen generate [<flags>]

Flags:  
  --output="crypto-config"  # 放置工件的输出目录
  --config=CONFIG           # 要使用的配置模板文件
```

下面生成一个简单的加密文件，配置模板文件 `crypto_config.yaml`，输出目录 `crypto_out`。`crypto_config.yaml`模板文件内容如下：

```yaml
OrdererOrgs:
- Name: Orderer
  Domain: hoojo.cnblogs.com
  Specs:
  - Hostname: orderer_host
PeerOrgs:
- Name: Org1
  Domain: org1.hoojo.cnblogs.com
  EnableNodeOUs: false
  Template:
    Count: 1
  Users:
    Count: 1
- Name: Org2
  Domain: org2.hoojo.cnblogs.com
  EnableNodeOUs: false
  Template:
    Count: 1
  Users:
    Count: 1
```

如果不知道模板文件格式，可以利用命令查看模板文件格式 

```sh
$ cryptogen showtemplate
```

利用命令生成配置文件，再用 `tree crypto_out -d -L 4` 命令查看`crypto_out` 输出目录5层结构如下：

```sh
$ rm -rf crypto_out/*
$ cryptogen generate --output=crypto_out --config=crypto_config.yaml
$ tree crypto_out -d -L 4
crypto_out
├── ordererOrganizations
│   └── hoojo.cnblogs.com			# --> 对应 OrdererOrgs 下的 Domain 属性值
│       ├── ca
│       ├── msp
│       │   ├── admincerts
│       │   ├── cacerts
│       │   └── tlscacerts
│       ├── orderers
│       │   └── orderer_host.hoojo.cnblogs.com	# --> 对应 OrdererOrgs 下的 Specs.Hostname 属性值
│       ├── tlsca
│       └── users
│           └── Admin@hoojo.cnblogs.com #--> @ 后面会关联上面的 Domain 属性值
└── peerOrganizations
    ├── org1.hoojo.cnblogs.com	#--> org1 关联上面的 PeerOrgs.Domain 属性值
    │   ├── ca
    │   ├── msp
    │   │   ├── admincerts
    │   │   ├── cacerts
    │   │   └── tlscacerts
    │   ├── peers
    │   │   └── peer0.org1.hoojo.cnblogs.com	# --> 关联上面org1的 PeerOrgs.Template.Count 属性值
    │   ├── tlsca
    │   └── users
    │       ├── Admin@org1.hoojo.cnblogs.com
    │       └── User1@org1.hoojo.cnblogs.com	# --> PeerOrgs.{Orgs}.Users.Count #Orgs=org1
    └── org2.hoojo.cnblogs.com	#--> org2 关联上面的 PeerOrgs.Domain 属性值
        ├── ca
        ├── msp
        │   ├── admincerts
        │   ├── cacerts
        │   └── tlscacerts
        ├── peers
        │   └── peer0.org2.hoojo.cnblogs.com
        ├── tlsca
        └── users
            ├── Admin@org2.hoojo.cnblogs.com
            └── User1@org2.hoojo.cnblogs.com
```

也可以调整配置文件，生成不同数量的`Order、Peer、User`的加密文件配置。

### extend 扩展现有网络

---

扩展网络其实就是在之前的网络的目录中生成新加入的配置加密文件数据

```sh
# 扩展存在的网络
$ cryptogen extend --help
usage: cryptogen extend [<flags>]

Flags:
  --help                   Show context-sensitive help (also try --help-long and --help-man).
  --input="crypto-config"  # 现有网络所在的输入目录
  --config=CONFIG          # 要使用的配置模板文件
```

创建一个模板配置文件 `crypto_config_extend.yaml` 内容如下：

```yaml
OrdererOrgs:
  - Name: Orderer
    Domain: hoojo.csdn.com
    Specs:
      - Hostname: orderer_host_csdn

PeerOrgs:
  - Name: Org10
    Domain: org10.hoojo.csdn.net
    EnableNodeOUs: false
    Template:
      Count: 1
    Users:
      Count: 1

  - Name: Org11
    Domain: org11.hoojo.csdn.net
    EnableNodeOUs: false
    Template:
      Count: 2
      Start: 10
      Hostname: "{{.Prefix}}.extend.{{.Index}}"
    Users:
      Count: 1
```

执行 `extend`命令生成新加入的配置的加密文件，随后查看目录发现之前的配置没有改变，并且加入新的配置文件。

```sh
cryptogen extend --input=crypto_out --config=crypto_config_extend.yaml
org10.hoojo.csdn.net
org11.hoojo.csdn.net

$ tree crypto_out -d -L 4 
```

# configtxgen 工具

`configtxgen` 工具用于创建四个文件： **orderer 创世块**、fabric **交易通道配置**和两个**anchor 节点**交易配置（每个`Peer Org`都有一个`anchor`）。

`orderer `区块是`orderer`服务的创世块，并且**通道交易配置文件**在通道**创建时被广播给`orderer`**。正如名称所示，`anchor`对等节点交易指定了此通道上的每个组织的`anchor peer`。

## 它是如何工作的？

`configtxgen`使用一个`configtx.yaml`包含端对端`e2e`示例网络定义的文件。有三个成员：一个`Orderer Org`（`OrdererOrg`）和两个`Peer Orgs` （`Org1`＆`Org2`），每个成员管理和维护两个对等节点。该文件还指定了由两个对等节点组织组成的联盟`SampleConsortium`。<br/>

```yaml
--
################################################################################
#   Profile 顶级配置
#
#   可以在此编码不同的profiles以进行指定作为configtxgen工具的参数
################################################################################
Profiles:

    TwoOrgsOrdererGenesis:
        Capabilities:
            <<: *ChannelCapabilities
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        # 联盟组合体
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *Org1
                    - *Org2
    TwoOrgsChannel:
        # 联盟
        Consortium: SampleConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
                - *Org2
            Capabilities:
                <<: *ApplicationCapabilities

################################################################################
#   Section: Organizations 组织
#
#   这部分定义了不同的组织标识稍后在配置中引用
################################################################################
Organizations:

    # SampleOrg使用sampleconfig定义一个MSP。它不应该被使用在生产中，但可以用作其他定义的模板
    - &OrdererOrg
        # DefaultOrg定义在fabric.git开发环境的sampleconfig中使用的组织
        Name: OrdererOrg
        # ID来加载MSP定义
        ID: OrdererMSP
        # MSPDir是包含MSP配置的文件系统路径
        MSPDir: crypto-config/ordererOrganizations/example.com/msp

    - &Org1
        Name: Org1MSP
        ID: Org1MSP
        MSPDir: crypto-config/peerOrganizations/org1.example.com/msp
        AnchorPeers:
            # AnchorPeers定义了可用于跨组织沟通的对等点的位置。
            # 请注意，此值仅在“应用程序”部分上下文中的起始块中编码
            - Host: peer0.org1.example.com
              Port: 7051

    - &Org2
        Name: Org2MSP
        ID: Org2MSP
        MSPDir: crypto-config/peerOrganizations/org2.example.com/msp
        AnchorPeers:
            - Host: peer0.org2.example.com
              Port: 7051

################################################################################
#   SECTION: Orderer
#
#   此部分定义要编码到Orderer相关参数的配置交易或生成块中的值
################################################################################
Orderer: &OrdererDefaults

    # Orderer类型：订阅者实现启动可用的类型是“solo”和“kafka”
    # solo 是基于单机硬盘模式的队列方式，可用于开发环境
    OrdererType: kafka

    Addresses:
        - orderer.example.com:7050

    # 批量超时：创建批次之前等待的时间量
    BatchTimeout: 2s

    # 批量大小：控制批量成块的消息数量
    BatchSize:

        # 最大消息数：允许的最大消息数
        MaxMessageCount: 10

        # 绝对最大字节数：允许批量中的序列化消息的绝对最大字节数。
        AbsoluteMaxBytes: 98 MB

        # 首选最大字节数：允许批量中的序列化消息的首选字节数。
        # 大于首选最大字节的消息将导致批量大于首选最大字节数。
        PreferredMaxBytes: 512 KB

    Kafka:
        # orderer连接的kafka名单
        Brokers:
            - kafka0:9092
            - kafka1:9092
            - kafka2:9092
            - kafka3:9092

    # 组织是在网络的订购者一侧定义为参与者的组织列表
    Organizations:

################################################################################
#   SECTION: Application
#
#   此部分定义要编码到应用程序相关参数的交易配置或创建块中的值
################################################################################
Application: &ApplicationDefaults

    # 组织是在网络应用程序端定义为参与者的组织列表
    Organizations:
    
################################################################################
#   SECTION: Capabilities
#   本节定义了fabric 网络的功能。
#
#   这是v1.1.0 以来的一个新概念，不应该在具有v1.0.x peer 和orderer 的混合网络中使用。
#   功能定义了FABRIC二进制中必须存在的功能，以便该二进制安全地参与FABRIC网络。
#   例如，如果添加了新的MSP类型，则较新的二进制文件可能会识别并验证来自此类型的签名，
#   而较旧的二进制文件则无法验证这些事务。这可能会导致具有不同世界状态的不同版本的结构二进制文件。
#   相反，为通道定义capability会通知那些没有这种capability的二进制文件，它们必须停止处理事务直到升级完成。
#   对于v1.0.x，如果定义了任何功能（包括关闭所有功能的地图），则v1.0.x对等将故意崩溃。
################################################################################
Capabilities:
    # 通道功能适用于orderers和peers，并且必须由两者支持。
    # 将capability的值设置为true 则需要它
    Global: &ChannelCapabilities
        # V1.1 for Global是行为的一个标志性标志，
        # 已被确定为所有运行v1.0.x的orderers和peers所期望的行为，但其修改会导致不兼容。
        # 用户应该将此标志设置为true。
        V1_1: true

    # Orderer的capabilities仅适用于orderers，并且可以安全地操纵，而不用担心升级同伴。
    # 将capability的值设置为true 则需要它
    Orderer: &OrdererCapabilities
        # V1.1 for Order是行为的一个catchall标志，
        # 已被确定为所有运行v1.0.x的orderers所期望的行为，但其修改会导致不兼容。
        # 用户应该将此标志设置为true。
        V1_1: true

    # 应用程序功能仅适用于peer对等网络，并且可以安全地操作而不用担心升级orderers。
    # 将capability的值设置为true 则需要它。
    Application: &ApplicationCapabilities
  
        # V1.1 for Application是行为的一个catchall标志，
        # 已被确定为运行v1.0.x的所有peers所期望的行为，但其修改会导致不兼容。 
        # 用户应该将此标志设置为true
        V1_1: true

```

请特别注意`configtx.yaml`文件顶部的配置文件`Profiles`部分，会看到我们有两个独立的配置。一个用于`orderer`创世区块 `TwoOrgsOrdererGenesis`， 另一个用于交易通道 `TwoOrgsChannel`。这些顶级配置文件很重要，因为我们将在创建配置文件时将它们作为**参数**传递给二进制文件。<br/>`configtx.yaml`文件还包含两个值得注意的格式规范。首先，我们为**每个`Peer Org`指定`anchor peer`锚点**（`peer0.org1.example.com`＆`peer0.org2.example.com`）。其次，我们指向**每个成员的MSP目录的位置**，这反过来又允许我们将每个组织的**根证书存储在`orderer`创世块中**。现在任何与`orderer`共识服务通信的网络实体都可以**验证其数字签名**。

官方提供了脚本`generateArtifacts.sh`，该脚本将生成加密材料和上面最开始提到的四个配置文件，这些文件保存到当前目录下的`channel-artifacts`文件夹中。

用`tree`命令查看文件目录结构如下：

```sh
$ sudo tree channel-artifacts/
channel-artifacts/
├── channel.tx				#--> 交易通道配置文件
├── genesis.block			#--> orderer 创世块
├── Org1MSPanchors.tx		#--> org1 anchor节点 交易配置
└── Org2MSPanchors.tx		#--> org2 anchor节点 交易配置
```

## configtxgen 基本命令

```sh
$ configtxgen --help
  -asOrg #  将配置生成作为特定组织（按名称）执行，仅包含org（可能）有权设置的写入集中的值
  -channelID # 在configtx中使用的通道ID（默认“testchainid”）
  -inspectBlock # 打印指定路径中块中包含的组态
  -inspectChannelCreateTx # 以指定的路径打印交易中包含的配置
  -outputAnchorPeersUpdate # 创建配置更新以更新锚点对等点（仅适用于默认渠道创建，仅适用于第一次更新）
  -outputBlock # 写入生成块的路径（如果设置）
  -outputCreateChannelTx # 写入通道创建configtx的路径（如果设置）
  -printOrg #  将组织的定义显示为JSON。 （可用于手动添加组织到频道）
  -profile # 用于生成的configtx.yaml配置文件。 （默认“SampleInsecureSolo”）
```

# 运行shell脚本

确保位于脚本所在的目录`examples/e2e_cli`中。确定创建的通道的名称是唯一的，并用你选择的名称替换`<channel-ID>`参数。如果不提供名称，该脚本将使用默认名称`mychannel`。

```sh
$ cd examples / e2e_cli 
$ ./generateArtifacts.sh <channel-ID>
```

脚本的输出有点冗长，因为它会生成加密库和多个文件。执行完成后，会在终端中看到五个截然不同的注释消息。他们内容如下（`generateArtifacts.sh` 文件经过我一些修改）：

```sh
##########################################################
##### Generate certificates using cryptogen tool #########
##########################################################
==> cryptogen generate --config=./crypto_config_extend.yaml --output=./crypto_out
org10.hoojo.csdn.net
org11.hoojo.csdn.net
==> cp docker-compose-e2e-template.yaml docker-compose-e2e.yaml

Using configtxgen -> /opt/gopath/src/github.com/hyperledger/fabric/test_example/../release/linux-amd64/bin/configtxgen

##########################################################
#########  Generating Orderer Genesis block ##############
##########################################################
==> cryptogen -profile TwoOrgsOrdererGenesis -outputBlock ./channel_out/genesis.block
2018-05-25 10:36:25.787 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-05-25 10:36:25.866 UTC [common/tools/configtxgen] doOutputBlock -> INFO 002 Generating genesis block
2018-05-25 10:36:25.868 UTC [common/tools/configtxgen] doOutputBlock -> INFO 003 Writing genesis block

#################################################################
### Generating channel configuration transaction 'channel.tx' ###
#################################################################
==> cryptogen -profile TwoOrgsChannel -outputCreateChannelTx ./channel_out/channel.tx -channelID tx_channel
2018-05-25 10:36:25.986 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-05-25 10:36:26.011 UTC [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2018-05-25 10:36:26.094 UTC [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 003 Writing new channel tx

#################################################################
#######    Generating anchor peer update for Org1MSP   ##########
#################################################################
==> cryptogen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel_out/Org1MSPanchors.tx -channelID tx_channel -asOrg Org1MSP
2018-05-25 10:36:26.240 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-05-25 10:36:26.267 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-05-25 10:36:26.268 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update

#################################################################
#######    Generating anchor peer update for Org2MSP   ##########
#################################################################
==> cryptogen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel_out/Org2MSPanchors.tx -channelID tx_channel -asOrg Org2MSP
2018-05-25 10:36:26.388 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-05-25 10:36:26.412 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-05-25 10:36:26.414 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
```

这些交易配置文件将绑定参与成员及其网络组件的加密材料，并输出`orderer`创世区块和三个通道交易配置文件。这些文件需要成功引导Fabric网络并创建一个要处理的通道。 

## 手动生成配置文件

可参考`generateArtifacts.sh`脚本中的命令，分解其中的步骤，我们手动处理配置文件生成，帮助理解脚本中的流程步骤。

### 设置平台类型
---
设置平台架构环境变量。该命令将检测当前操作系统并为后续步骤需要使用的二进制文件：

```sh
# for power or z
$ os_arch=$(echo "$(uname -s)-$(uname -m)" | awk '{print tolower($0)}') && echo $os_arch
linux-x86_64

# for linux, osx or windows
os_arch=$(echo "$(uname -s)-amd64" | awk '{print tolower($0)}') && echo $os_arch
linux-amd64
```

现在可以运行二进制工具。不同平台的二进制文件位于`release` 目录中，因为当前系统的`$os_arch`为`linux-amd64`。

### 生成加密证书
---
确保你在`examples/e2e_cli`目录中，因此工具所在的路径： 

```sh
$ ../../release/linux-amd64/bin/cryptogen generate --config=./crypto-config.yaml
org1.example.com
org2.example.com
```

这样就成功生成了`crypto` 文件，默认文件夹在当前目录下的`crypto-config`中。

### 生成创世块
---
接下来，需要告诉`configtxgen`工具在哪里查找它需要获取的`configtx.yaml`文件。先看看我们目前的工作目录：

```sh
FABRIC_CFG_PATH=$PWD
```

生成创世块，其中 `TwoOrgsOrdererGenesis`这个参数关联配置文件 `configtx.xml`中的 `Profiles.TwoOrgsOrdererGenesis` 

```sh
$ ../../release/linux-amd64/bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/man-genesis.block
2018-05-28 02:32:38.654 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-05-28 02:32:38.748 UTC [msp] getMspConfig -> INFO 002 Loading NodeOUs
2018-05-28 02:32:38.781 UTC [msp] getMspConfig -> INFO 003 Loading NodeOUs
2018-05-28 02:32:38.786 UTC [common/tools/configtxgen] doOutputBlock -> INFO 004 Generating genesis block
2018-05-28 02:32:38.789 UTC [common/tools/configtxgen] doOutputBlock -> INFO 005 Writing genesis block
```

### 创建通道配置
---
创建交易通道配置文件，其中 `TwoOrgsChannel`这个参数关联配置文件 `configtx.xml`中的 `Profiles.TwoOrgsChannel` ：

```sh
# channel-ID 是一个参数，可以进行指定其他数据
$ ../../release/linux-amd64/bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/man-channel.tx -channelID tx_channel
2018-05-28 03:25:56.227 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-05-28 03:25:56.263 UTC [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2018-05-28 03:25:56.305 UTC [msp] getMspConfig -> INFO 003 Loading NodeOUs
2018-05-28 03:25:56.352 UTC [msp] getMspConfig -> INFO 004 Loading NodeOUs
2018-05-28 03:25:56.426 UTC [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 005 Writing new channel tx
```

### 创建`anchor peer`配置
---
在`channel`上为`Org1`定义`anchor peer`，其中 `Org1MSP`这个参数关联配置文件 `configtx.xml`中的 `Profiles.Organizations.Org*.ID`对应的值 （`org*` 对应其中的一个`Org`配置）：

```sh
$ ../../release/linux-amd64/bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID tx_channel -asOrg Org1MSP

2018-05-28 03:28:57.429 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-05-28 03:28:57.477 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-05-28 03:28:57.478 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
```

同上，在`channel`上创建`Org2`的`anchor peer`

```sh
$ ../../release/linux-amd64/bin/configtxgen  -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID tx_channel -asOrg Org2MSP
```

# 使用 docker 运行 `e2e`示例

首先进入目录`examples/e2e`中，然后使用`docker-compose`生成网络实体进行测试。请注意，可以设置一个 `TIMEOUT`变量（以秒为单位），以便在脚本完成后`cli`容器不会退出。

```sh
# channel_name 传入刚才使用的 channel_id 值 tx_channel
# CHANNEL_NAME=<channel-id> TIMEOUT=<pick_a_value> docker-compose -f docker-compose-cli.yaml up -d
$ CHANNEL_NAME=tx_channel TIMEOUT=10000 sudo docker-compose -f docker-compose-cli.yaml up -d
WARNING: The CHANNEL_NAME variable is not set. Defaulting to a blank string.
WARNING: The TIMEOUT variable is not set. Defaulting to a blank string.
Creating network "e2ecli_default" with the default driver
Creating zookeeper2 ...
Creating peer1.org1.example.com ...
Creating zookeeper0 ...
Creating peer0.org1.example.com ...
Creating zookeeper1 ...
Creating peer0.org2.example.com ...
Creating peer1.org2.example.com ...
Creating zookeeper2
Creating peer1.org1.example.com
Creating peer0.org1.example.com
Creating peer0.org2.example.com
Creating zookeeper0
Creating zookeeper1
Creating zookeeper1 ... done
Creating kafka1 ...
Creating kafka0 ...
Creating kafka2 ...
Creating kafka1
Creating kafka3 ...
Creating kafka0
Creating peer0.org2.example.com ... done
Creating kafka0 ... done
Creating orderer.example.com ...
Creating orderer.example.com ... done
Creating cli ...
Creating cli ... done
```

 这样 `docker-compose`就启动了在`docker-compose-cli.yaml`的编排服务。

等待`60`秒左右。在后台中有交易配置被发送给其他同伴`anchor peer`节点。执行`docker ps`查看活动容器。应该看到与以下内容相同的输出：

```sh
$ sudo docker ps
CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS              PORTS                                                                       NAMES
ed81b87194d2        hyperledger/fabric-tools       "/bin/bash -c './scr…"   2 minutes ago       Up 2 minutes                                                                                    cli
dc33f34cc803        hyperledger/fabric-kafka       "/docker-entrypoint.…"   2 minutes ago       Up 2 minutes        9093/tcp, 0.0.0.0:32780->9092/tcp                                           kafka1
cfdd263d72d5        hyperledger/fabric-kafka       "/docker-entrypoint.…"   2 minutes ago       Up 2 minutes        9093/tcp, 0.0.0.0:32779->9092/tcp                                           kafka2
2fecdb0462d6        hyperledger/fabric-kafka       "/docker-entrypoint.…"   2 minutes ago       Up 2 minutes        9093/tcp, 0.0.0.0:32778->9092/tcp                                           kafka0
af0dfda450bf        hyperledger/fabric-kafka       "/docker-entrypoint.…"   2 minutes ago       Up 2 minutes        9093/tcp, 0.0.0.0:32777->9092/tcp                                           kafka3
7579935f13b2        hyperledger/fabric-zookeeper   "/docker-entrypoint.…"   2 minutes ago       Up 2 minutes        0.0.0.0:32776->2181/tcp, 0.0.0.0:32775->2888/tcp, 0.0.0.0:32773->3888/tcp   zookeeper2
18ac2164ad60        hyperledger/fabric-zookeeper   "/docker-entrypoint.…"   2 minutes ago       Up 2 minutes        0.0.0.0:32774->2181/tcp, 0.0.0.0:32772->2888/tcp, 0.0.0.0:32771->3888/tcp   zookeeper0
7c882f4fe67a        hyperledger/fabric-zookeeper   "/docker-entrypoint.…"   2 minutes ago       Up 2 minutes        0.0.0.0:32770->2181/tcp, 0.0.0.0:32769->2888/tcp, 0.0.0.0:32768->3888/tcp   zookeeper1
```

## 后台程序流程

+ 脚本 `script.sh` 在CLI容器内进行运行。脚本根据提供的通道名称`CHANNEL_NAME=tx_channel`执行方法`createChannel`，在该方法中运行创建通道命令`peer channel create`，并使用`channel.tx`文件进行通道配置。

  ```sh
  # 在文件 docker-compose-cli.yaml 中配置
  command: /bin/bash -c './scripts/script.sh ${CHANNEL_NAME}; sleep $TIMEOUT'
  		
  $ peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile $ORDERER_CA >&log.txt
  ```

+ `createChannel`方法输出是一个创世块，它存储在同级的文件系统`<your_channel_name>.block`中，并包含从`channel.tx`指定的通道配置。

+ 所有四个同等节点都执行`joinChannel`命令，其将以前生成的创世块作为输入。该命令指示对等节点加入`<your_channel_name>`并创建一个以`<your_channel_name>.block`开始的链。

  ```sh
  
  $ peer channel join -b $CHANNEL_NAME.block  >&log.txt
  ```

+ 现在有一个由四个对等节点和两个组织组成的通道。`TwoOrgsChannel`是顶级配置文件下的。

+ `peer0.org1.example.com`和`peer1.org1.example.com`属于`Org1`，`peer0.org2.example.com`和`peer1.org2.example.com`属于`Org2`

+ 这些关系是通过`crypto-config.yaml`定义的，MSP路径是在我们的`docker-compose`文件中指定的。

+ 更新`Org1MSP`（`peer0.org1.example.com`）和`Org2MSP`（`peer0.org2.example.com`）的`anchor peer`节点。通过将`Org1MSPanchors.tx`和`Org2MSPanchors.tx`文件与通道名称一起传递到`orderer`服务来实现此目的。

  ```sh
  $ peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/${CORE_PEER_LOCALMSPID}anchors.tx --tls --cafile $ORDERER_CA >&log.txt
  ```

+  **chaincode_example02** 安装在`peer0.org1.example.com`和 `peer0.org2.example.com`节点上

  ```sh
  $ peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 >&log.txt
  ```

+ 链码在`peer0.org2.example.com`上实例化，实例化将链码添加到通道，启动目标对等节点的容器，并初始化与链码关联的键值对。这个例子的初始值是`["a","100" "b","200"]`。这个实例化以`dev-peer0.org2.example.com-mycc-1.0`的名称启动一个容器。

  ```sh
  $ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "OR	('Org1MSP.peer','Org2MSP.peer')" >&log.txt
  ```

+ 实例化也通过了背书策略论据，该策略被定义为 `-P "OR ('Org1MSP.member','Org2MSP.member')"` ，这意味着任何交易都必须由与`Org1`或`Org2` 对Peer等节点进行认可。

+ 针对`a`值的查询将发送到`peer0.org1.example.com`。链代码先前已安装在`peer0.org1.example.com`上，因此它将以`dev-peer0.org1.example.com-mycc-1.0`的名称为`Org1 peer0`启动一个容器。查询的结果也被返回。没有发生写操作，因此对`a`的查询仍会返回`100`值。

  ```sh
  $ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}' >&log.txt
  ```

+ 将调用发送到`peer0.org1.example.com`以将`10`从`a`移动到`b`

  ```sh
  $ peer chaincode invoke -o orderer.example.com:7050  --tls --cafile $ORDERER_CA -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}' >&log.txt
  ```

+ 在 `peer1.org2.example.com` 安装链码，同上安装**chaincode_example02** ，但需要切换变量到 `peer1.org2`

+ 查询发送到`peer1.org2.example.com`获取`a`的值。这将以名称`dev-peer1.org2.example.com-mycc-1.0`启动第三个链码容器。返回值`90`，正确地反映前一个事务，在此期间键`a`的值被修改了`10`。

## 说明了什么

`Chaincode` **必须安装在对等节点上**，以便它成功地对账本执行`读/写`操作。此外，直到对该链码**执行`初始`或传`统交易—读取/写入`（例如，查询“a”的值）**，链码容器才被**启动**用于**对等节点**。**交易将导致容器启动**。此外，通道中的所有对等节点方都会**维护账本**的精确**副本**，其中包含区块链以将块中**不可变的顺序**记录存储起来，以及用于维护当前结构状态的**状态数据库**。这包括那些没有安装`chaincode`的同等节点（如上例中的`peer1.org1.example.com`）。最后，链码在安装后可以访问（如上例中的`peer1.org2.example.com`），因为它已经被实例化。

## 如何查看交易

### 查看`docker CLI`容器日志

```sh
$ docker logs -f cli
```

### 查看智能合约链码容器日志

检查个别链码容器以查看针对每个容器执行的单独交易。以下是每个容器的日志输出：

```sh
$ docker logs dev-peer0.org2.example.com-mycc-1.0
04:30:45.947 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
ex02 Init
Aval = 100, Bval = 200

$ docker logs dev-peer0.org1.example.com-mycc-1.0
04:31:10.569 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
ex02 Invoke
Query Response:{"Name":"a","Amount":"100"}
ex02 Invoke
Aval = 90, Bval = 210

$ docker logs dev-peer1.org2.example.com-mycc-1.0
04:31:30.420 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
ex02 Invoke
Query Response:{"Name":"a","Amount":"90"}
```

## 一键运行`e2e`示例

使用单个`shell`脚本来运行示例，可以生成配置文件和加密证书配置。`cryptogen`，`configtxgen`和`docker-compose`命令嵌入在`shell`脚本中。如果在执行`shell`选择不提供通道ID，则脚本将使用默认名称`mychannel`。`cli`超时参数是一个可选值，如果不设置它，那么`cli`容器将在脚本结束时**退出**。

```sh
$ ./network_setup.sh up

$ ./network_setup.sh up <channel-ID> <timeout-value>
```

# 了解 `docker-compose`服务编排

`e2e_cli`文件夹提供了两种类型的`docker-compose`文件，这两种文件都是从`docker-compose-base.yaml`扩展而来的。我们的第一个编排示例是`docker-compose-cli.yaml`，它为我们提供了一个`CLI`容器，以及一个`Orderer`节点，四个对等节点和可选的`couchDB`容器。我们使用此`docker-compose`来完成本页面上的所有说明。

第二个编排示例是`docker-compose-e2e.yaml`构建为使用`Node.js SDK`运行端到端测试。除了SDK的功能之外，它的主要区别在于有`fabric-ca`服务器的容器。因此，我们能够将**REST请求**发送到CA组织以进行**用户注册和登记**。

如果希望在没有优先运行脚本情况下下使用`docker-compose-e2e.yaml`，那么我们需要进行四项轻微修改。我们需要指出本组织CA的私钥。你可以在你的`crypto-config`文件夹中找到这些值。例如，要找到`Org1`的私钥，我们将遵循此路径`crypto-config/peerOrganizations/org1.example.com/ca/ `。私钥是一个长的哈希值，后跟`_sk`。Org2的路径是 -  `crypto-config/peerOrganizations/org2.example.com/ca/ `。

在`docker-compose-e2e.yaml`中更新`ca0`和`ca1`的`FABRIC_CA_SERVER_TLS_KEYFILE`变量。还需要编辑命令中提供的路径以启动`ca`服务器。为每个CA容器提供两次相同的私钥。

# 手动运行命令

## 退出容器
退出当前正在运行的容器：  

```sh
$ docker rm -f $(docker ps -aq)
```

## 删除链码容器
在终端中执行`docker images`命令以查看链码镜像。它们看起来类似于以下内容：

```sh
$ docker images
REPOSITORY                            TAG                              IMAGE ID            CREATED             SIZE
dev-peer1.org2.example.com-mycc-1.0   latest                           4bc5e9b5dd97        5 seconds ago       176 MB
dev-peer0.org1.example.com-mycc-1.0   latest                           6f2aeb032076        22 seconds ago      176 MB
dev-peer0.org2.example.com-mycc-1.0   latest                           509b8e393cc6        39 seconds ago      176 MB
```

删除这些链码镜像

```sh
$ docker rmi -f $(docker images dev-* -qa)
```

## 生成交易配置文件
确保你有交易和创世块配置文件。如果您删除了它们，请再次运行shell脚本：

```sh
$ rm -rf channel-artifacts/* crypto-config/*
# ./generateArtifacts.sh <channel-ID>
$ ./generateArtifacts.sh my_tx_channel
```

## 修改 `docker-compose`文件并启动
打开`docker-compose-cli.yaml`文件并注释掉运行`script.sh`的命令。找到文件中的`cli`容器，并注释掉该行代码。例如：

```yaml
working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
#command: /bin/bash -c './scripts/script.sh ${CHANNEL_NAME}; sleep $TIMEOUT'
volumes:
```

现在重新启动编排服务：

```sh
$ pwd
/opt/gopath/src/github.com/hyperledger/fabric/examples/e2e_cli

$ CHANNEL_NAME=my_tx_channel TIMEOUT=1000 docker-compose -f docker-compose-cli.yaml up -d
```

如果想查看编排服务的实时日志，请不要提供`-d`标志。如果让日志滚动输出，那么将需要打开第二个终端来执行CLI调用。

## 命令语法参考
语法需要参考`script.sh`文件中的 `create`和`join`命令。

针对`peer0.org1.example.com`的CLI命令的工作方式，需要使用下面给出的四个环境变量来运行命令。这些用于`peer0.org1.example.com`的变量被传递到CLI容器中，因此我们可以在不传递它们的情况下进行操作。但是，如果想将请求发送给其他对等节点和`orderer`服务，则需要相应地提供这些变量值。查看`docker-compose-base.yaml`中的特定路径：

```sh
# Environment variables for PEER0

CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
CORE_PEER_LOCALMSPID="Org1MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

## 创建通道
运行以下命令，在CLI容器交互模式下运行

```sh
$ docker exec -it cli bash

root@b754923d6c72:/opt/gopath/src/github.com/hyperledger/fabric/peer#
```

执行成功后发现工作目录被切换

用`-c`标志指定通道名称。用`-f`标志指定通道交易配置。当前情况下它是`channel.tx`，但是可以使用不同的名称来挂载自己的交易配置。

```sh
# channel.tx文件安装在cli容器中的channel-artifacts目录中
# 因此，我们传递文件的完整路径
# 我们还通过orderer ca-cert的路径来验证TLS握手
# 请务必正确替换 $CHANNEL_NAME变量

CORE_PEER_LOCALMSPID="Org1MSP" \
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp \
CORE_PEER_ADDRESS=peer0.org1.example.com:7051

$ peer channel create -o orderer.example.com:7050 -c my_tx_channel -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

这些手动命令的其余部分将保留在CLI容器中。在定位除`peer0.org1.example.com`以外的对等节点时，还必须记住在所有命令前加上相应的环境变量。

## 加入通道
加入特定的对等节点到通道

```sh
# Org1
CORE_PEER_LOCALMSPID="Org1MSP" \
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp \
CORE_PEER_ADDRESS=peer1.org1.example.com:7051
#CORE_PEER_ADDRESS=peer0.org1.example.com:7051

# Org2
CORE_PEER_LOCALMSPID="Org2MSP" \
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp \
CORE_PEER_ADDRESS=peer0.org2.example.com:7051
#CORE_PEER_ADDRESS=peer1.org2.example.com:7051

$ peer channel join -b my_tx_channel.block
```

通过对四个环境变量进行适当更改，可以根据需要让其他同伴加入该频道。

## 安装链码
将智能合约链码安装到远程对等节点上

```sh
$ peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
```

## 实例化链码

实例化链码并定义相互认可政策，在对等节点上实例化链码`mycc`。这将为目标对象启动链码容器并设置链码的认证策略。在这个片段中，我们将策略定义为需要来自属于`Org1`或`Org2`的一个对等节点的认证。该命令是：

```sh
# 如果你没有安装mycc的chaincode，那么也修改该参数
# 初始化账户 a=100，b=200
$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C my_tx_channel -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"
```

## 调用链码
```sh
# a 为 b 交易转账
$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C my_tx_channel -n mycc -c '{"Args":["invoke","a","b","10"]}'
```

## 查询链码

```sh
# 查询a的账户数据
$ peer chaincode query -C my_tx_channel -n mycc -c '{"Args":["query","a"]}'

Query Result: 90
```

# 使用 `CouchDB`

状态数据库可以从默认（`goLevelDB`）切换到`CouchDB`。`CouchDB`可以使用相同的链码函数，但是，根据链码数据建模为`JSON`，还可以对状态数据库数据内容执行丰富而复杂的查询。

要使用`CouchDB`而不是默认数据库（`goLevelDB`），请遵循**手动练习命令**部分中的相同步骤，除了在启动编排服务时也要通过`docker-compose`启动`couchdb `。确保位于`docker-compose`脚本所在的`examples/e2e_cli`目录中：

```sh
$ CHANNEL_NAME=my_tx_channel TIMEOUT=1000 docker-compose -f docker-compose-cli.yaml -f docker-compose-couch.yaml up -d
```

**chaincode_example02** 现在应该在下面使用CouchDB。

> **注意**
>
> 如果选择将`fabric-couchdb`容器端口映射到主机端口，请确保知道安全隐患。在开发环境中映射端口使`CouchDB REST API`可用，并允许通过`CouchDB Web`界面（`Fauxton`）对数据库进行可视化。生产环境可能不会实施端口映射，以**限制对`CouchDB`容器的外部访问**。

可以使用上面概述的步骤将`chaincode_example02`链接代码用于`CouchDB`状态数据库，但是为了使用`CouchDB`查询功能，需要使用将数据建模为`JSON`的链式代码（例如`marbles02`）。您可以在`fabric/examples/chaincode/go` 目录中找到**marbles02** `chaincode`。

## 手动运行命令

手动执行命令，需要设置通用变量，进入`CLI`交互模式下

```sh
# CLI容器交互模式下运行，执行成功后发现工作目录被切换
$ docker exec -it cli bash
root@b754923d6c72:/opt/gopath/src/github.com/hyperledger/fabric/peer#
```

在交互默认下继续运行后续的命令行操作

```sh
# 设置变量值，通道名称
CHANNEL_NAME=my_tx_channel
# CA证书
CC_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

### 创建和加入通道

---

我们将按照上面（**手动运行命令**章节）相同的流程创建和加入通道。

```sh
# 创建通道
CORE_PEER_LOCALMSPID="Org1MSP" \
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp \
CORE_PEER_ADDRESS=peer0.org1.example.com:7051

$ peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile $CC_CA

# 加入通道---------------------------------------------
# Org1
CORE_PEER_LOCALMSPID="Org1MSP" \
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp \
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
#CORE_PEER_ADDRESS=peer1.org1.example.com:7051

# Org2
CORE_PEER_LOCALMSPID="Org2MSP" \
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp \
CORE_PEER_ADDRESS=peer0.org2.example.com:7051
#CORE_PEER_ADDRESS=peer1.org2.example.com:7051

$ peer channel join -b $CHANNEL_NAME.block
```

### 安装和实例化链码

---

将对等节点加入频道后，请使用以下步骤与`marbles02`链码进行互动：

```sh
$ peer chaincode install -o orderer.example.com:7050 -n marbles -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/marbles02

$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $CC_CA -C $CHANNEL_NAME -n marbles -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org2MSP.member','Org1MSP.member')"
```

### 调用链码

---

创建一些弹珠并移动它们：

```sh
# 下面引用变量的地方会用上面的变量声明值进行替换
$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile $CC_CA -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble1","blue","35","tom"]}'

$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile $CC_CA -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble2","red","50","tom"]}'

$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile $CC_CA -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble3","blue","60","tom"]}'

$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile $CC_CA -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarble","marble3","jerry"]}'

$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile $CC_CA -C $CHANNEL_NAME -n marbles -c '{"Args":["readMarble","marble3"]}'

$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile $CC_CA -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarblesBasedOnColor","blue","jerry"]}'

$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile $CC_CA -C $CHANNEL_NAME -n marbles -c '{"Args":["delete","marble1"]}'

$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile $CC_CA -C $CHANNEL_NAME -n marbles -c '{"Args":["readMarble","marble1"]}'

Marble does not exist: marble1
```

如果选择在`docker-compose`中映射`CouchDB`端口，则可以通过打开浏览器并浏览URL`http://localhost:5984/_utils`，通过`CouchDB Web`界面（`Fauxton`）查看状态数据库：

```sh
services:
  couchdb0:
    container_name: couchdb0
    image: hyperledger/fabric-couchdb
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    # Comment/Uncomment the port mapping if you want to hide/expose the CouchDB service,
    # for example map it to utilize Fauxton User Interface in dev environments.
    ports:
      - "5984:5984"           
```

应该看到名为`mychannel`的数据库（或唯一通道名称）以及其中的`document`。

### 查询与检索

---

从`CLI`运行查询，读取 `marble2`：

```sh
$ peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["readMarble","marble2"]}'

Query Result: {"color":"red","docType":"marble","name":"marble2","owner":"tom","size":50}
```

可以检索特定的`marble1 `历史记录 ：

```sh
$ peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["getHistoryForMarble","marble1"]}'

Query Result: [{"TxId":"01d5f0d0d544718273bfebe572e79bef0373f9151537d287fc22f8fe3c9eba99", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"tom"}, "Timestamp":"2018-05-29 03:42:36.621539573 +0000 UTC", "IsDelete":"false"},{"TxId":"63a3fb67598d7f48cc60d5f4033766485ebc0f2a08aefc078d24e7ec36e67baa", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"jerry"}, "Timestamp":"2018-05-29 03:44:29.204375425 +0000 UTC", "IsDelete":"false"},{"TxId":"32d3d01488f89cf52bad0de96629978820d74a35c6f68062d83589a5321c66b0", "Value":null, "Timestamp":"2018-05-29 03:44:48.029558642 +0000 UTC", "IsDelete":"true"}]
```

还可以对数据内容执行其他的查询，例如所有者`jerry`查询：

```sh
$ peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarblesByOwner","jerry"]}'

Query Result: [{"Key":"marble3", "Record":{"color":"blue","docType":"marble","name":"marble3","owner":"jerry","size":60}}]
```

## 关于数据持久性的注意事项

如果在对等节点容器或`CouchDB`容器上需要**数据持久性**，则一种选择是将`docker-host`中的目录**挂载到容器中**的相关目录中。例如，可以在`docker-compose-base.yaml`文件的对等节点容器的服务编排文件中添加以下两行：

```yaml
volumes:
 - /var/hyperledger/peer0:/var/hyperledger/production
```

将主机中的`/var/hyperledger/peer0` 挂载在容器中的 `/var/hyperledger/production` 目录位置上。

对于`CouchDB`容器，可以在`CouchDB`容器服务编排文件中添加以下两行：

```yaml
volumes:
 - /var/hyperledger/couchdb0:/opt/couchdb/data
```

# 故障排除

## 启动失败问题排除

在调用 `./network_setup.sh up`的时候出现以下情况

```sh
2018-05-23 08:33:44.586 UTC [grpc] Printf -> DEBU 006 grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: Error while dialing dial tcp: lookup orderer.example.com on 127.0.0.11:53: read udp 127.0.0.1:32807->127.0.0.11:53: i/o timeout"; Reconnecting to {orderer.example.com:7050 <nil>}
Error: Error connecting due to  rpc error: code = Unavailable desc = grpc: the connection is unavailable
```

`SERVICE_UNAVAILABLE` 服务没有成功启动

```sh
2018-05-24 06:59:14.597 UTC [channelCmd] readBlock -> DEBU 00a Got status: &{SERVICE_UNAVAILABLE}
Error: can't read the block: &{SERVICE_UNAVAILABLE}
```

解决方案，执行获取docker 镜像脚本，重新获取docker镜像。确保脚本中的`版本`和`源代码版本`对应。

```sh
$ cd /opt/gopath/src/github.com/hyperledger/fabric
$ git tag
v1.1.0

$ cd /opt/gopath/src/github.com/hyperledger/fabric/scripts
$ sudo ./bootstrap.sh 1.1.0
```

## 启动网络服务

- 建议清理并新建网络。使用以下命令删除配置文件、加密证书、容器和链码镜像：

  ```sh
  $ ./network_setup.sh down
  $ ./network_setup.sh up
  ```

- 如果看到docker错误，请首先检查版本（应该是1.12或更高版本），然后尝试重启docker进程。Docker的问题通常无法立即识别。例如，可能会看到由于无法访问容器中安装的加密文件而导致的错误。

- 如果错误持续，建议删除镜像并从头开始：

  ```sh
  $ make clean
  $ make docker
  $ make release
  ```

- 如果看到下面的错误：

  ```sh
  Error: Error endorsing chaincode: rpc error: code = 2 desc = Error installing chaincode code mycc:1.0(chaincode /var/hyperledger/production/chaincodes/mycc.1.0 exits)
  ```

  之前的运行可能包含链码镜像（例如，`dev-peer1.org2.example.com-mycc-1.0`或`dev-peer0.org1.example.com-mycc-1.0`）。删除它们，然后重试。

  ```sh
  $ docker rmi -f $(docker images | grep peer[0-9]-peer[0-9] | awk '{print $3}')
  ```

- 如果看到类似于以下内容的内容：

  ```sh
  Error connecting: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure
  Error: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure
  ```

  确保在生成配置时指向`release`文件夹中的二进制文件是正确的，并且当前代码版本分支标签`tag`和`docker images`是对应的。

- 如果你看到下面的错误：

  ```sh
  [configtx/tool/localconfig] Load -> CRIT 002 Error reading configuration: Unsupported Config Type ""
  panic: Error reading configuration: Unsupported Config Type ""
  ```

  没有正确设置`FABRIC_CFG_PATH`环境变量。`configtxgen`工具需要这个变量才能找到`configtx.yaml`。返回并重新创建通道配置文件。