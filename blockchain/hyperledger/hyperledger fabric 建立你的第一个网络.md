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

接下来，需要告诉`configtxgen`工具在哪里查找它需要摄取的`configtx.yaml`文件。将在当前的工作目录中告诉它，**`configtxgen` 会读取这个变量的值，然后找到配置文件 `configtx.yaml`**：

```sh
# configtxgen 会读取这个变量的值，然后找到配置文件 configtx.yaml
export FABRIC_CFG_PATH=$PWD
```

然后，将**调用`configtxgen`工具来创建`orderer genesis`块**：

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

接下来，需要**创建通道交易配置**。请务必替换`$CHANNEL_NAME`或将`CHANNEL_NAME`设置为可在整个操作说明中使用的环境变量：

```sh
# The channel.tx artifact contains the definitions for our sample channel

export CHANNEL_NAME=mychannel  && ../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
```

应该在终端中看到类似于以下内容的输出：

```sh
2017-10-26 19:24:05.324 EDT [common/tools/configtxgen] main -> INFO 001 Loading configuration
2017-10-26 19:24:05.329 EDT [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2017-10-26 19:24:05.329 EDT [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 003 Writing new channel tx
```

接下来，将在通道上为`Org1`定义**锚点对等体**。同样，请务必替换`$CHANNEL_NAME`或为以下命令设置环境变量。终端输出将模仿通道事务工件的输出：

```sh
../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP
```

现在，将在同一个通道上为`Org2`定义**锚点对等体**：

```sh
../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP
```

# 启动网络

> **注意**：如果运行`byfn.sh`上面的示例，请确保在继续操作之前已关闭测试网络（请参阅 [“关闭网络”](https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html#bring-down-the-network)）。

将利用脚本来启动我们的网络。`docker-compose`文件引用先前下载的图像，并**使用之前生成的`genesis.block`引导订购者**。

希望手动完成命令，以便展示每个调用的语法和功能。

首先启动网络：

```sh
$ docker-compose -f docker-compose-cli.yaml up -d
```

**如果要查看网络的实时日志，请不要提供`-d`标志。如果让日志滚动，那么将需要打开第二个终端来执行`CLI`调用**。

## 环境变量

要使以下针对`peer0.org1.example.com`的`CLI`命令起作用，需要**在命令前面加上下面给出的四个环境变量**。`peer0.org1.example.com`的这些变量被拷贝到`CLI`容器中，因此可以在不传递它们的情况下进行操作。但是，如果要将**调用发送到其他对等方或订货方**，则可以**通过在启动容器之前编辑`docker-compose-base.yaml`**来相应地提供这些值。修改以下四个环境变量以使用不同的对等方和组织。

```sh
# Environment variables for PEER0

CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
CORE_PEER_LOCALMSPID="Org1MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

## 创建和加入频道

在之前，使用“[创建通道配置事务”](https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html#createchanneltx)部分中的 `configtxgen` 工具[创建了通道配置事务](https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html#createchanneltx)。可以使用`configtx.yaml`传递给`configtxgen`工具的相同或不同的配置文件重复该过程以创建其他通道配置事务。然后，可以重复本节中定义的过程，以在网络中建立其他通道。

将使用`docker exec`命令进入`CLI`容器：

```sh
$ docker exec -it cli bash
```

如果成功，应该看到以下内容：

```sh
root@0d78bb69300d:/opt/gopath/src/github.com/hyperledger/fabric/peer#
```

如果不想针对默认对等`peer0.org1.example.com`运行`CLI`命令，请在四个环境变量中替换`peer0`或`org1`的值并运行命令：

```sh
# Environment variables for PEER0

export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

接下来，将作为创建通道请求的一部分，将在“[创建通道配置事务](https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html#createchanneltx)”部分（我们称之为`channel.tx`）中创建的生成的通道配置事务文件传递给订购者。

使用`-c`标志**指定通道名称**，使用`-f`标志**指定通道交易配置**。在这种情况下，它是`channel.tx`，但是可以使用其他名称装入自己的交易配置。再次在`CLI`容器中设置`CHANNEL_NAME`环境变量，以便不必显式传递此参数。**通道名称必须全部小写，长度小于250个字符，并且与正则表达式`[a-z][a-z0-9.-]*`匹配**。

```sh
export CHANNEL_NAME=mychannel

# the channel.tx file is mounted in the channel-artifacts directory within your CLI container
# as a result, we pass the full path for the file
# we also pass the path for the orderer ca-cert in order to verify the TLS handshake
# be sure to export or replace the $CHANNEL_NAME variable appropriately

$ peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

> **注意**：注意作为此命令的一部分传递的`--cafile`。它是`orderer`的根证书的**本地路径**，允许**验证`TLS`握手**。

此命令返回一个创世块：`<channel-ID.block>` ，将用它来加入频道。它包含`channel.tx`中**指定的配置信息**如果没有对默认通道名称进行任何修改，那么该命令将返回一个名为`mychannel.block`的原型二进制配置文件。

> **注意**：对于其余的这些手动命令，将保留在`CLI`容器中。在定位`peer0.org1.example.com`以外的对等方时，还必须记住**在所有命令前加上相应的环境变量**。

现在将`peer0.org1.example.com`加入频道

```sh
# By default, this joins ``peer0.org1.example.com`` only
# the <channel-ID.block> was returned by the previous command
# if you have not modified the channel name, you will join with mychannel.block
# if you have created a different channel name, then pass in the appropriately named block

$ peer channel join -b mychannel.block
```

可以根据需要通过对在上面的[环境变量](https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html#peerenvvars)部分中使用的四个环境变量进行适当更改来使其他对等方加入通道。 

将加入`peer0.org2.example.com`，而不是加入每个对等体，**以便可以正确更新通道中的锚点对等体**。由于将覆盖`CLI`容器中的默认环境变量，因此完整命令如下：

```sh
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt 
$ peer channel join -b mychannel.block
```

或者，可以选择单独设置这些环境变量，而不是传入整个字符串。一旦设置完毕，只需再次发出`peer channel join`命令，`CLI`容器将代表`peer0.org2.example.com`。

## 更新锚点对等节点

以下命令是通道更新，它们将传播到通道的定义。实质上，**在通道的创世块之上添加了额外的配置信息**。请注意，不是修改`genesis`块，而是**简单地将增量添加**到将定义锚点对等的链中。

更新通道定义以将`Org1`的锚点对等体定义为`peer0.org1.example.com`：

```sh
$ peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

现在更新通道定义以将`Org2`的锚点对等体定义为`peer0.org2.example.com`。与`Org2`对等体的对等通道连接命令相同，需要在此调用前加上适当的环境变量。

```sh
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org2MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

## 安装和实例化`Chaincode`

> **注意**：将使用一个简单的现有链码。要了解如何编写自己的链代码，请参阅[Chaincode for Developers](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html)教程。

应用程序与区块链分类帐交互`chaincode`。因此，需要在**每个将执行和支持交易的对等体上安装**链码，然后在通道上实例化链码。

首先，将示例`Go，Node.js`或`Java`链代码安装到**四个对等节点之一**上。这些命令将指定的源代码放在我们的对等文件系统上。

> **注意**：**只能为每个链代码名称和版本安装一个版本的源代码**。源代码存在于对等体的**文件系统**中，在链代码名称和版本的上下文中，它与语言无关。类似地，实例化的链代码容器将反映对等体上安装的任何语言。

**GoLang**

```sh
# this installs the Go chaincode
$ peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
```

**Node.js**

```sh
# this installs the Node.js chaincode
# make note of the -l flag; we use this to specify the language
$ peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node/
```

**Java**

```sh
$ peer chaincode install -n mycc -v 1.0 -l java -p /opt/gopath/src/github.com/chaincode/chaincode_example02/java/
```

接下来，**在通道上实例化链码**。这将**初始化**通道上的链代码，**设置**链代码的**认可策略**，并为目标对等方**启动链代码容器**。记下`-P`参数。这是背书认可策略，在此策略中**指定针对要验证的此链码的交易所需的认可级别**。

在下面的命令中，会注意到将策略指定为`-P "AND ('Org1MSP.peer','Org2MSP.peer')"`。这意味着**需要来自属于`Org1 AND Org2`的对等方的“认可”（即两个认可）**。如果将语法更改为**OR**，那么**只需要一个认可**。

**GoLang**

```sh
# be sure to replace the $CHANNEL_NAME environment variable if you have not exported it
# if you did not install your chaincode with a name of mycc, then modify that argument as well

$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
```

**Node.js**

> **注意**：`Node.js`链代码的实例化大约需要一分钟。命令没有挂起，而是在**编译图像时安装`fabric-shim`镜像图层**。国内的可能需要下载数据，最好能访问国外的资源。

```sh
# be sure to replace the $CHANNEL_NAME environment variable if you have not exported it
# if you did not install your chaincode with a name of mycc, then modify that argument as well
# notice that we must pass the -l flag after the chaincode name to identify the language

$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -l node -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
```

**Java**

> **注意**：请注意，`Java`链代码实例化可能需要一些时间，因为它编译链代码并使用`java`环境下载`docker`容器。

```sh
$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -l java -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
```

有关策略实施的更多详细信息，请参阅[认可政策](http://hyperledger-fabric.readthedocs.io/en/latest/endorsement-policies.html)文档。 

如果希望其他对等方与分类帐**进行交互**，则需要将它们连接到通道，并**将链码源的相同名称，版本和语言安装到相应的对等文件系统**上。一旦他们尝试与特定的链代码进行交互，就会为**每个对等体启动一个链代码容器**。再次，要认识到`Node.js`镜像的编译速度会慢一些。

一旦在通道上实例化了链代码，就可以放弃`-l`标志。只需**传递通道标识符和链码的名称**。

## 查询

让我们查询`a`的值，以确保链代码被正确实例化并填充状态`DB`。查询语法如下：

```sh
# be sure to set the -C and -n flags appropriately

$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
```

## 调用

现在让我们从`a`到`b`移动`10`。此事务将创建新块并更新状态`DB`。调用的语法如下：

```sh
# be sure to set the -C and -n flags appropriately

$ peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'
```

## 查询

**确认之前的调用是否正确执行**。使用值`100`初始化了键值`a`，并在之前的调用中删除了`10`。因此，对`a`的查询应该显示`90`查询的语法如下。

```sh
# be sure to set the -C and -n flags appropriately

$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
```

应该看到以下内容：

```sh
Query Result: 90
```

## 应用内部发生了什么？

> **注意**：这些步骤描述了`script.sh`由`./byfn.sh up`运行的场景。使用`./byfn.sh down`清理网络并确保此命令处于活动状态。然后使用相同的`docker-compose`提示再次启动网络。

+ 脚本 `script.sh`在`CLI`容器中拷贝。该脚本根据提供的**通道名称**驱动`createChannel`命令，并使用`channel.tx`文件**进行通道配置**。
+ `createChannel`的**输出是一个创世块** `<your_channel_name> .block`它**存储在对等体的文件系统中***，并包含从`channel.tx`指定的通道配置。
+ 对**所有四个对等体执行`joinChannel`命令**，其将**先前生成的生成块**作为**输入**。此命令指示对等方加入`<your_channel_name>`并创建以`<your_channel_name>.block`开头的链。
+ 现在我们有一个**由四个同行和两个组织组成的频道**。这是`TwoOrgsChannel`的信息资料。
+ `peer0.org1.example.com`和`peer1.org1.example.com`属于`Org1`。`peer0.org2.example.com`和`peer1.org2.example.com`属于`Org2`。
+ 这些关系是通过`crypto-config.yaml`定义的，`MSP`路径是在`docker compose`中指定的。
+ 然后更新 `Org1MSP（peer0.org1.example.com）`和`Org2MSP（peer0.org2.example.com）`的**锚点对等体**。通过将`Org1MSPanchors.tx`和`Org2MSPanchors.tx`工件与**通道名称**一起传递给**orderer 服务**来完成此操作。
+ 链码 `chaincode_example02` 安装在 `peer0.org1.example.com` 和 `peer0.org2.example.com` 上。
+ 然后在`peer0.org2.example.com`上**实例化**链代码。**实例化将链代码添加到通道**，启动目标对等方的容器，并**初始化与链代码关联的键值对**。该示例的初始值是`[“a”,”100” “b”,”200”]`，实例化会启动一个名为`dev-peer0.org2.example.com-mycc-1.0`的容器。
+ 实例化也传递了**背书策略的论据**。该策略定义为`-P "AND ('Org1MSP.peer','Org2MSP.peer')"`，表示任何事务必须由与`Org1`和`Org2`相关联的对等方签署。
+ 向`peer0.org1.example.com`发出针对`a`值的查询。该链代码以前安装在`peer0.org1.example.com`上，因此这将以`dev-peer0.org1.example.com-mycc-1.0`的名称为`Org1 peer0`启动一个容器。还将返回查询结果。**没有发生写入操作**，因此对`a`的查询仍将返回值`100`。
+ 将调用发送到`peer0.org1.example.com`，并`a`向`b`转账10
+ 然后将链代码安装在`peer1.org2.example.com`上
+ 查询将发送到`peer1.org2.example.com`以获取`a`的值。这将启动名为`dev-peer1.org2.example.com-mycc-1.0`的第三个链代码容器。返回值`90`，正确反映上一个事务，在此期间，键`a`的值被修改为10。

## 这说明了什么？

**`Chaincode`必须安装在对等体上**，以便它能够成功地对分类帐执行**读/写**操作。此外，在针对该链代码执行`init`或传统事务（**读/写**）之前，**不会为对等体启动链代码容器**（例如，查询“a”的值）。该事务导致容器启动。此外，通道中的所有对等体都**保持分类帐的精确副本**，其包括用于**以块的形式**存储**不可变的有序**记录的区块链，以及用于维护当前**状态的快照的状态数据库**。这包括那些没有安装链代码的对等体（如上例中的`peer1.org1.example.com`）。最后，链代码在安装后可以访问（如上例中的`peer1.org2.example.com`），因为它已经被实例化了。

## 如何查看这些交易？

检查`CLI Docker`容器的日志：

```sh
$ docker logs -f cli
```

应该看到以下输出：

```sh
2017-05-16 17:08:01.366 UTC [msp] GetLocalMSP -> DEBU 004 Returning existing local MSP
2017-05-16 17:08:01.366 UTC [msp] GetDefaultSigningIdentity -> DEBU 005 Obtaining default signing identity
2017-05-16 17:08:01.366 UTC [msp/identity] Sign -> DEBU 006 Sign: plaintext: 0AB1070A6708031A0C08F1E3ECC80510...6D7963631A0A0A0571756572790A0161
2017-05-16 17:08:01.367 UTC [msp/identity] Sign -> DEBU 007 Sign: digest: E61DB37F4E8B0D32C9FE10E3936BA9B8CD278FAA1F3320B08712164248285C54
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

可以滚动浏览这些日志以查看各种交易。

## 如何查看链码日志？

检查各个链代码容器，以查看针对每个容器执行的单独事务。以下是每个容器的组合输出：

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

# 了解`Docker Compose`服务编排

`BYFN`示例提供了两种`Docker Compose`文件，这两种文件都是从`docker-compose-base.yaml`（位于基本文件夹中）扩展而来的。第一个版本`docker-compose-cli.yaml`为提供了一个`CLI`容器，以及一个订购者，四个同行。将此文件用于此页面上的所有说明。

> **注意**：本节的其余部分介绍了为`SDK`设计的`docker-compose`文件。 有关运行这些测试的详细信息，请参阅[Node SDK](https://github.com/hyperledger/fabric-sdk-node) repo。

第二种风格`docker-compose-e2e.yaml`构建为使用`Node.js SDK`运行端到端测试。除了使用`SDK`之外，它的主要区别在于`Fabric-ca`服务器还有容器。因此，**可以向组织`CA`发送`REST`调用以进行用户注册和认证**。

如果想在没有先运行`byfn.sh`脚本的情况下使用`docker-compose-e2e.yaml`，那么需要进行四次略微修改。需要修改指向组织CA的私钥。可以在`crypto-config`文件夹中找到这些值。例如，要找到`Org1`的私钥，将遵循此路径 `crypto-config/peerOrganizations/org1.example.com/ca/`。私钥是一个长哈希值，后跟`_sk`。`Org2`的路径是 `crypto-config/peerOrganizations/org2.example.com/ca/`。

在`docker-compose-e2e.yaml`中更新`ca0`和`ca1`的`FABRIC_CA_SERVER_TLS_KEYFILE`变量。还需要编辑命令中提供的路径以启动`ca`服务器。为每个`CA`容器提供两次相同的私钥。

# 使用 `CouchDB`

状态数据库可以从默认（`goleveldb`）切换到`CouchDB`。`CouchDB`提供了相同的链代码功能，但是，根据链代码数据被建模为`JSON`，还可以根据状态数据库数据内容执行**丰富而复杂的查询**。

要使用`CouchDB`而不是默认数据库（`goleveldb`），请按照前面概述的相同步骤生成工件，除非在启动网络时也通过`docker-compose-couch.yaml`：

```sh
$ docker-compose -f docker-compose-cli.yaml -f docker-compose-couch.yaml up -d
```

`chaincode_example02` 现在应该使用下面的`CouchDB`。

> **注意**：如果选择将`fabric-couchdb`容器端口映射到主机端口，请确保了解安全隐患。在开发环境中映射端口使`CouchDB REST API`可用，并允许通过`CouchDB Web`界面（`Fauxton`）可视化数据库。**生产环境可能会避免实施端口映射，以限制对`CouchDB`容器的外部访问**。

可以使用上面列出的步骤对`CouchDB`状态数据库使用`chaincode_example02`链代码，但是为了运用`CouchDB`查询功能，需要使用具有建模为`JSON`的数据的链代码（例如`marbles02`）。您可以在`fabric/examples/chaincode/go`目录中找到`marbles02`链代码。

将按照上面相同的流程创建和加入频道，如上面的[创建和加入频道](https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html#createandjoin)部分所述。将对等方加入频道后，请使用以下步骤与**marbles02**链代码进行交互：

+ 安装并实例化链代码`peer0.org1.example.com`：

  ```sh
  # be sure to modify the $CHANNEL_NAME variable accordingly for the instantiate command
  
  peer chaincode install -n marbles -v 1.0 -p github.com/chaincode/marbles02/go
  peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org0MSP.peer','Org1MSP.peer')"
  ```

+ 制作一些弹珠并移动它们：

  ```sh
  # be sure to modify the $CHANNEL_NAME variable accordingly
  
  peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble1","blue","35","tom"]}'
  peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble2","red","50","tom"]}'
  peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble3","blue","70","tom"]}'
  peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarble","marble2","jerry"]}'
  peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarblesBasedOnColor","blue","jerry"]}'
  peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["delete","marble1"]}'
  ```

+ 如果选择在`docker-compose`中映射`CouchDB`端口，现在可以通过打开浏览器并导航到以下`URL`，通过`CouchDB Web`界面（`Fauxton`）查看状态数据库：

  ```sh
  http://localhost:5984/_utils
  ```

  应该看到名为`mychannel`（或唯一通道名称）的数据库及其中的文档。

+ **注意**：对于以下命令，请确保适当更新`$CHANNEL_NAME`变量。

  可以从`CLI`运行常规查询：

  ```sh
  peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["readMarble","marble2"]}'
  ```

  输出应显示`marble2`的详细信息：

  ```sh
  Query Result: {"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}
  ```

+ 可以检索特定弹珠的历史记录，例如`marble1`：

  ```sh
  peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["getHistoryForMarble","marble1"]}'
  ```

  输出应显示`marble1`上的交易：

  ```sh
  Query Result: [{"TxId":"1c3d3caf124c89f91a4c0f353723ac736c58155325f02890adebaa15e16e6464", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"tom"}},{"TxId":"755d55c281889eaeebf405586f9e25d71d36eb3d35420af833a20a2f53a3eefd", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"jerry"}},{"TxId":"819451032d813dde6247f85e56a89262555e04f14788ee33e28b232eef36d98f", "Value":}]
  ```

+ 还可以对数据内容执行丰富的查询，例如按所有者`jerry`查询大弹珠字段：

  ```sh
  peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarblesByOwner","jerry"]}'
  ```

  输出结果：

  ```sh
  Query Result: [{"Key":"marble2", "Record":{"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}},{"Key":"marble3", "Record":{"color":"blue","docType":"marble","name":"marble3","owner":"jerry","size":70}}]
  ```

# 为何选择`CouchDB`

`CouchDB`是一种`NoSQL`解决方案。它是一个面向文档的数据库，其中文档字段存储为**键值映射**。字段可以是简单的**键值对，列表或映射**。除了`LevelDB`支持的键控/复合键/键范围查询外，`CouchDB`还支持完全**数据丰富的查询**功能，例如针对整个区块链数据的**非键查询**，因为其数据内容以`JSON`格式存储，完全可查询。因此，`CouchDB`可以满足`LevelDB`不支持的许多用例的链代码，审计和报告要求。

`CouchDB`还可以**增强区块链中的合规性和数据保护的安全性**。因为它能够通过**过滤和屏蔽**事务中的各个属性来实现字段级**安全性**，并且只在需要时**授权只读权限**。

此外，`CouchDB`属于`CAP`定理的`AP`类型（**可用性和分区容差**）。它使用具有最终一致性的主：**主复制模型**。可以在`CouchDB`文档的`Eventual Consistency`页面上找到更多信息。但是，在每个结构对等体下，没有数据库副本，对数据库的**写入保证一致且持久**（不是最终一致性）。

`CouchDB`是`Fabric`的第一个外部**可插拔状态数据库**，可能也应该有其他外部数据库选项。例如，`IBM`为其区块链启用了关系数据库。并且`CP`类型（一致性和分区容差）数据库也可能需要，以便在没有应用程序级别保证的情况下实现数据一致性。

# 关于数据持久性

如果在对等容器或·CouchDB·容器上需要**数据持久性**，则可以选择将`docker-host`中的目录安装到容器中的相关目录中。例如，可以在`docker-compose-base.yaml`文件的`peer`容器中添加以下两行：

```yaml
volumes:
 - /var/hyperledger/peer0:/var/hyperledger/production
```

对于`CouchDB`容器，可以在`CouchDB`容器规范中添加以下两行：

```sh
volumes:
 - /var/hyperledger/couchdb0:/opt/couchdb/data
```