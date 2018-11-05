# `hyperledger fabric` 建立你的第一个网络
演示`hyperledger fabric`的第一个网络示例，帮助了解换`hyperledger fabric`的运行流程原理和运行环境。

构建您的第一个网络（`BYFN`）方案提供了一个示例`Hyperledger Fabric`网络，该网络由**两个组织**组成，每个组织维护**两个对等节点**，以及一个“**独立**”**订购服务**。

# 环境

在开始之前，如果还没有这样做，可能希望检查是否已在将要开发区块链应用程序和运行`Hyperledger Fabric`的平台上安装了所有[环境工具](hyperledger%20fabric%20入门.md)。

还需要[安装示例代码，二进制文件和Docker镜像](https://hyperledger-fabric.readthedocs.io/en/latest/install.html)。会注意到`fabric-samples` 存储库中包含许多示例。将使用该`first-network`样本。现在打开那个子目录。

```sh
$ cd fabric-samples/first-network
```

> **注意**：本文档中提供的命令**必须**从存储库`fabric-samples`克隆的子目录运行`first-network`。如果选择从其他位置运行命令，则各种提供的脚本将**无法找到二进制文件**。

# 运行示例

提供一个完全注释的脚本`byfn.sh`利用这些`Docker`镜像快速引导`Hyperledger Fabric`网络，网络由代表**两个**不同组织的**4个对等体**和**一个`orderer`节点**组成。它还将**启动一个容器**来运行脚本执行，该执行将**对等方节点**连接到一个**通道**，**部署和实例化链代码**并根据部署的链代码驱动交易执行。

`byfn.sh`脚本的帮助文本：

```sh
Usage:
  byfn.sh <mode> [-c <channel name>] [-t <timeout>] [-d <delay>] [-f <docker-compose-file>] [-s <dbtype>] [-l <language>] [-i <imagetag>] [-v]
    <mode> - one of 'up', 'down', 'restart', 'generate' or 'upgrade'
      - 'up' - bring up the network with docker-compose up
      - 'down' - clear the network with docker-compose down
      - 'restart' - restart the network
      - 'generate' - generate required certificates and genesis block
      - 'upgrade'  - upgrade the network from version 1.2.x to 1.3.x
    -c <channel name> - channel name to use (defaults to "mychannel")
    -t <timeout> - CLI timeout duration in seconds (defaults to 10)
    -d <delay> - delay duration in seconds (defaults to 3)
    -f <docker-compose-file> - specify which docker-compose file use (defaults to docker-compose-cli.yaml)
    -s <dbtype> - the database backend to use: goleveldb (default) or couchdb
    -l <language> - the chaincode language: golang (default) or node
    -i <imagetag> - the tag to be used to launch the network (defaults to "latest")
    -v - verbose mode
  byfn.sh -h (print this message)

Typically, one would first generate the required certificates and
genesis block, then bring up the network. e.g.:

        byfn.sh generate -c mychannel
        byfn.sh up -c mychannel -s couchdb
        byfn.sh up -c mychannel -s couchdb -i 1.2.x
        byfn.sh up -l node
        byfn.sh down -c mychannel
        byfn.sh upgrade -c mychannel

Taking all defaults:
        byfn.sh generate
        byfn.sh up
        byfn.sh down
```

如果选择不提供通道名称，则脚本将使用默认名称`mychannel`。`CLI`客户端超时参数（使用`-t`标志指定）是可选值。如果选择不设置它，那么`CLI`将放弃在默认设置`10`秒后进行的查询请求。

## 生成网络工件

执行以下命令，即可生成网络共件（网络证书、通道配置、创世块等）

```sh
$ ./byfn.sh generate
```

执行后将看到有关将发生什么的简要说明，以及是否继续执行命令行提示。响应`y`或按回车键执行所描述的操作。

```sh
$ ./byfn.sh generate
Generating certs and genesis block for channel 'mychannel' with CLI timeout of '10' seconds and CLI delay of '3' seconds
Continue? [Y/n] y
proceeding ...
/opt/gopath/src/github.com/hyperledger/fabric/release/linux-amd64/bin/cryptogen

##########################################################
##### Generate certificates using cryptogen tool #########
##########################################################
+ cryptogen generate --config=./crypto-config.yaml
org1.example.com
org2.example.com
+ res=0
+ set +x

/opt/gopath/src/github.com/hyperledger/fabric/release/linux-amd64/bin/configtxgen
##########################################################
#########  Generating Orderer Genesis block ##############
##########################################################
+ configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
2018-11-05 02:27:17.379 UTC [common/tools/configtxgen] main -> WARN 001 Omitting the channel ID for configtxgen for output operations is deprecated.  Explicitly passing the channel ID will be required in the future, defaulting to 'testchainid'.
2018-11-05 02:27:17.402 UTC [common/tools/configtxgen] main -> INFO 002 Loading configuration
2018-11-05 02:27:17.655 UTC [common/tools/configtxgen] doOutputBlock -> INFO 003 Generating genesis block
2018-11-05 02:27:17.658 UTC [common/tools/configtxgen] doOutputBlock -> INFO 004 Writing genesis block
+ res=0
+ set +x

#################################################################
### Generating channel configuration transaction 'channel.tx' ###
#################################################################
+ configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mychannel
2018-11-05 02:27:17.785 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-11-05 02:27:17.844 UTC [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2018-11-05 02:27:17.958 UTC [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 003 Writing new channel tx
+ res=0
+ set +x

#################################################################
#######    Generating anchor peer update for Org1MSP   ##########
#################################################################
+ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP
2018-11-05 02:27:18.083 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-11-05 02:27:18.136 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-11-05 02:27:18.137 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
+ res=0
+ set +x

#################################################################
#######    Generating anchor peer update for Org2MSP   ##########
#################################################################
+ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID mychannel -asOrg Org2MSP
2018-11-05 02:27:18.244 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-11-05 02:27:18.283 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-11-05 02:27:18.285 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
+ res=0
+ set +x
```

上面的命令将生成各种网络实体所有**证书和密钥**，用于引导订购服务`genesis block`，以及配置[Channel](https://hyperledger-fabric.readthedocs.io/en/latest/glossary.html#channel)所需的一组交易配置文件。

## 启动网络

接下来，可以使用以下命令之一启动网络： 

```sh
$ ./byfn.sh up
```

上面的命令将编译`Golang`链代码图像并启动相应的容器。`Go`是默认的链码语言，但是也支持[Node.js](https://fabric-shim.github.io/)和[Java](https://fabric-chaincode-java.github.io/) 链码。如果想通过`Node.js`链码运行本教程，请改为使用以下命令：

```sh
# we use the -l flag to specify the chaincode language
# forgoing the -l flag will default to Golang

$ ./byfn.sh up -l node
```

> **注意**：有关`Java Shim`的更多信息，请参阅其 [文档](https://fabric-chaincode-java.github.io/org/hyperledger/fabric/shim/Chaincode.html)。有关`Node Shim`的更多信息，请参阅其 [文档](https://fabric-shim.github.io/fabric-shim.ChaincodeInterface.html)。

如果使用Java链代码运行示例，则必须指定如下：`-l java`

```sh
$ ./byfn.sh up -l java
```

> **注意**：不要运行这两个命令。除非**关闭并重新**创建网络，否则每次启动只能尝试一种语言。

执行命令后，系统将提示是继续还是中止。回复`y`或按回车键： 

```sh
Starting with channel 'mychannel' and CLI timeout of '10'
Continue? [Y/n]
proceeding ...
Creating network "net_byfn" with the default driver
Creating peer0.org1.example.com
Creating peer1.org1.example.com
Creating peer0.org2.example.com
Creating orderer.example.com
Creating peer1.org2.example.com
Creating cli


 ____    _____      _      ____    _____
/ ___|  |_   _|    / \    |  _ \  |_   _|
\___ \    | |     / _ \   | |_) |   | |
 ___) |   | |    / ___ \  |  _ <    | |
|____/    |_|   /_/   \_\ |_| \_\   |_|

Channel name : mychannel
Creating channel...
```

运行命令后，系统日志将是上的样子。这将启动所有容器，然后启动完整的端到端应用程序方案。成功完成后，它应在终端窗口中报告以下内容：

```sh
Query Result: 90
2017-05-16 17:08:15.158 UTC [main] main -> INFO 008 Exiting.....
===================== Query successful on peer1.org2 on channel 'mychannel' =====================

===================== All GOOD, BYFN execution completed =====================


 _____   _   _   ____
| ____| | \ | | |  _ \
|  _|   |  \| | | | | |
| |___  | |\  | | |_| |
|_____| |_| \_| |____/
```

可以滚动浏览这些日志以查看各种交易。如果没有得到这个结果，请跳到[故障排除](https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html#troubleshoot)部分，看看我们是否可以帮助发现问题所在。

## 卸载网络

最后，把它全部停下来，这样就可以一步一步地探索网络设置。以下内容将终止容器，删除加密材料和四个工件，并从`Docker Registry`中删除链码图像：

```sh
$ ./byfn.sh down
```

再一次，系统将提示继续，回复`y`或按回车键： 

```sh
Stopping with channel 'mychannel' and CLI timeout of '10'
Continue? [Y/n] y
proceeding ...
WARNING: The CHANNEL_NAME variable is not set. Defaulting to a blank string.
WARNING: The TIMEOUT variable is not set. Defaulting to a blank string.
Removing network net_byfn
468aaa6201ed
...
Untagged: dev-peer1.org2.example.com-mycc-1.0:latest
Deleted: sha256:ed3230614e64e1c83e510c0c282e982d2b06d148b1c498bbdcc429e2b2531e91
.....
```

如果想了解有关底层工具和引导机制的更多信息，请继续阅读。在接下来的部分中，将介绍构建全功能`Hyperledger Fabric`网络的各种步骤和要求。

> **注意**：下面概述的手动步骤假定`CORE_LOGGING_LEVEL`在`cli`容器被设置为`DEBUG`。可以通过修改`first-network`目录中的`docker-compose-cli.yaml`文件来设置此项。例如
>
> ```
> cli:
>   container_name: cli
>   image: hyperledger/fabric-tools:$IMAGE_TAG
>   tty: true
>   stdin_open: true
>   environment:
>     - GOPATH=/opt/gopath
>     - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
>     - CORE_LOGGING_LEVEL=DEBUG
>     #- CORE_LOGGING_LEVEL=INFO
> ```

# 加密生成器

将使用`cryptogen`工具为各种网络实体生成加密材料（`x509`证书和签名密钥）。这些证书代表身份，它们允许在实体进行通信和交易时进行签名、验证身份验证。

## 工作原理

`Cryptogen`使用一个文件 `crypto-config.yaml` 它包含拓扑网络，并允许我们为组织和属于这些组织的组件生成一组**证书和密钥**。每个组织都配置了一个**唯一的根证书**（`ca-cert`），**用于将特定组件（同行和订购者）绑定到该组织**。通过为每个组织分配唯一的`CA`证书，我们模仿典型的网络，其中参与的成员将使用其自己的证书颁发机构。`Hyperledger Fabric`中的事务和通信由**实体的私钥（密钥库）签名**，然后通过**公钥（`signcerts`）进行验证**。

注意到`count`此文件中的变量。我们用它来**指定每个组织的对等数量**。在例子中，每个`Org`有两个对等体。现在不会深入研究[`x.509`证书和公钥基础设施](https://en.wikipedia.org/wiki/Public_key_infrastructure)的细节 。如果有兴趣，可以在自己的时间内仔细阅读这些主题。

在运行该工具之前，快速浏览一下`crypto-config.yaml`中的一个片段。特别注意`OrdererOrgs`标题下的`“Name”，“Domain”`和`“Specs”`参数：

```yml
OrdererOrgs:
#---------------------------------------------------------
# Orderer
# --------------------------------------------------------
- Name: Orderer
  Domain: example.com
  CA:
      Country: US
      Province: California
      Locality: San Francisco
  #   OrganizationalUnit: Hyperledger Fabric
  #   StreetAddress: address for org # default nil
  #   PostalCode: postalCode for org # default nil
  # ------------------------------------------------------
  # "Specs" - See PeerOrgs below for complete description
# -----------------------------------------------------
  Specs:
    - Hostname: orderer
# -------------------------------------------------------
# "PeerOrgs" - Definition of organizations managing peer nodes
 # ------------------------------------------------------
PeerOrgs:
# -----------------------------------------------------
# Org1
# ----------------------------------------------------
- Name: Org1
  Domain: org1.example.com
  EnableNodeOUs: true
```

网络实体的命名约定如下 `“{{.Hostname}}.{{.Domain}}”`。因此，使用我们的订购节点作为参考点，我们留下了一个名为 `orderer.example.com`的订购节点，它与`Orderer`的`MSP ID`相关联。该文件包含有关定义和语法的大量文档。还可以参考[会员服务提供商（`MSP`）](https://hyperledger-fabric.readthedocs.io/en/latest/msp.html)文档，深入了解`MSP`。

运行该`cryptogen`工具后，生成的证书和密钥将保存到名为`crypto-config`的文件夹中。 

# 配置交易生成器

`configtxgen`工具用于创建四个配置工件：

- `orderer`服务创世块配置 `genesis block`
- `channel`通道交易配置 `configuration transaction`
- 两个对等交易锚点 `anchor peer transactions`  一个组织 `Peer Org`

有关此工具功能的完整说明，请参阅[configtxgen](https://hyperledger-fabric.readthedocs.io/en/latest/commands/configtxgen.html)

`orderer`块是订购服务的[Genesis Block](https://hyperledger-fabric.readthedocs.io/en/latest/glossary.html#genesis-block)，并且通道配置交易文件在[Channel](https://hyperledger-fabric.readthedocs.io/en/latest/glossary.html#channel)创建时**广播到订购者**。正如名称所暗示的那样，锚点对等事务在此通道上指定每个`Org`的[Anchor Peer](https://hyperledger-fabric.readthedocs.io/en/latest/glossary.html#anchor-peer)。 

## 工作原理

`Configtxgen`使用文件 `configtx.yaml`包含示例网络的定义。有三个成员，一个`Orderer Org`（`OrdererOrg`）和两个`Peer Orgs` （`Org1`＆`Org2`），每个成员管理和维护两个对等节点。该文件还指定了一个联盟 `SampleConsortium`  由我们的两个`Peer Orgs`组成。请特别注意此文件顶部的“配置文件”部分。会注意到我们有两个唯一标头，一个用于`orderer genesis`块： `TwoOrgsOrdererGenesis` ，一个用于我们的通道：`TwoOrgsChannel`。

这些头文件很重要，因为我们将在创建工件时将它们作为参数传递。 

> **注意**：`SampleConsortium`在系统级配置文件中定义，然后由通道级配置文件引用。通道存在于一个联盟的范围内，所有联盟必须在整个网络的范围内定义。

此文件还包含两个值得注意的其他规范。首先，我们为每个`Peer Org`（`peer0.org1.example.com`＆`peer0.org2.example.com`）指定**锚点对等体**。其次，我们指向每个成员的`MSP`目录的位置，从而允许我们在`orderer genesis`块中存储每个`Org`的根证书。这是一个关键概念。现在，与订购服务通信的任何网络实体都可以验证其数字签名。

# 运行工具

可以使用`configtxgen`和`cryptogen`命令手动生成**证书/密钥**的各种配置工件。或者，可以尝试调整`byfn.sh`脚本来实现手动生成加密的配置文件。

## 手动生成工件

可以在`byfn.sh`脚本中引用`generateCerts`函数，以获取生成将用于网络配置的证书所需的命令，如`crypto-config.yaml`文件中所定义。但是，为方便起见，也将在此提供参考。

首先**运行加密工具**，**二进制**文件位于`bin`目录中，因此需要提供工具所在位置的相对路径。

```sh
../bin/cryptogen generate --config=./crypto-config.yaml
```

应该在终端中看到以下内容：

```sh
org1.example.com
org2.example.com
```

证书和密钥（即`MSP`材料）将输出到第一个网络目录根目录下的目录：`crypto-config`。

接下来，需要告诉`configtxgen`工具在哪里查找它需要摄取的`configtx.yaml`文件。将在当前的工作目录中告诉它：

```sh
# configtxgen 会读取这个变量的值，然后找到配置文件 configtx.yaml
export FABRIC_CFG_PATH=$PWD
```

然后，将调用`configtxgen`工具来创建`orderer genesis`块：

```sh
../bin/configtxgen -profile TwoOrgsOrdererGenesis -channelID byfn-sys-channel -outputBlock ./channel-artifacts/genesis.block
```

执行上面的命令后，应该在终端中看到类似于以下内容的输出：

```sh
2017-10-26 19:21:56.301 EDT [common/tools/configtxgen] main -> INFO 001 Loading configuration
2017-10-26 19:21:56.309 EDT [common/tools/configtxgen] doOutputBlock -> INFO 002 Generating genesis block
2017-10-26 19:21:56.309 EDT [common/tools/configtxgen] doOutputBlock -> INFO 003 Writing genesis block
```

> **注意**：`orderer genesis`块和即将创建的后续工件将输出到该项目根目录的`channel-artifacts`目录中。上述命令中的`channelID`是系统通道的名称。

## 创建通道交易配置

