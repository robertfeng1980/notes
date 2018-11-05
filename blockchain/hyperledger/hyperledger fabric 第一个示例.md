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



## 卸载网络

