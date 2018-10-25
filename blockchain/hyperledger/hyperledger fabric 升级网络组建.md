# 升级 `Hyperledger Fabric` 网络组建

以 `BYFN `（`first-sample`）作为示例，进行升级操作。将 `BYFN` 从版本 `1.1.0` 升级到版本 `1.2.0` 的网络。

# 概述

因为[构建第一个网络](https://hyperledger-fabric.readthedocs.io/en/release-1.2/build_network.html)（`BYFN`）教程默认采用**最新**二进制文件，如果您自`v1.2`发布以来已经运行它，您的计算机将安装`v1.2`二进制文件和工具，您将无法升级他们。

因此，本教程将提供基于`Hyperledger Fabric v1.1`二进制文件的网络以及您要升级到的`v1.2`二进制文件。此外，我们将展示如何**将通道配置更新为新的`v1.2`功能**，该功能将允许对等方正确处理[私有数据](https://hyperledger-fabric.readthedocs.io/en/release-1.2/private-data/private-data.html) 和[访问控制列表（ACL）](https://hyperledger-fabric.readthedocs.io/en/release-1.2/access_control.html)。

> **注意**：如果您的网络尚未在`Fabric v1.1`中，请按照将[网络升级到v1.1](http://hyperledger-fabric.readthedocs.io/en/release-1.1/upgrading_your_network_tutorial.html)的说明进行操作 。本文档中的说明仅涵盖从`v1.1`移至`v1.2`，而不是从任何其他版本移至`v1.2`。

## 不被升级的组件

由于`BYFN`不支持以下组件服务，因此我们用于升级`BYFN`的脚本不会涉及到它们：

- **`Fabric CA`**
- **`Kafka`**
- **`CouchDB`**
- **`SDK`**

升级这些组件的过程（如有必要）将在本教程后面的部分中介绍。

## 升级流程

我们的升级教程将执行以下步骤：

1. 备份分类帐本和`MSP`：将包含 `orderer`、`peer` 容器中的账本数据和 `MSP` 证书配置。
2. 将`orderer`二进制文件升级到`Fabric v1.2`。
3. 将`peer`二进制文件升级到`Fabric v1.2`。
4. 启用新的`v1.2`功能。

> **注意**：在生产环境中，`orderer`和`peer`可以**同时进行滚动升级**。换句话说，您可以**按任何顺序升级二进制文件，而无需关闭网络**。因为`BYFN`使用**SOLO** 排序共识服务，我们的脚本会使整个网络崩溃。但这在生产环境中不是必需的。
>
> 但是，确保启用功能不会对当前正在运行的`orderer`和`peer`版本产生问题非常重要。对于`v1.2`，新功能位于应用程序组中，该应用程序组管理与对等方节点相关的功能，因此不会与排序服务冲突。

本教程将演示如何使用`CLI`命令单独执行每个步骤。 

## 运行环境

如果还没有这样做，请确保计算机上具有所有依赖项，如[环境条件中所述](https://hyperledger-fabric.readthedocs.io/en/release-1.2/prereqs.html)。

# 启动`v1.1`网络

首先，我们将提供运行`Fabric v1.1` 镜像的基本网络。该网络将由**两个组织**组成，**每个组织维护两个对等节点，以及一个`独立`排序服务**。

将在本地克隆的`fabric-samples`子目录中运行`first-network`。立即切换到该目录。还需要打开一些额外的终端以方便使用。

```sh
$ cd opt/gopath/src/github.com/hyperledger/fabric-samples
$ git clone -b master https://github.com/hyperledger/fabric-samples.git
$ cd fabric-samples
$ git checkout v1.1.0
```

## 清理

保证干净清洁的环境是必要的，因此将使用`byfn.sh`脚本进行初步整理工作。此命令将终止(**停止并删除**)所有活动或过时的`docker`容器，并删除任何以前生成的文件。运行以下命令：

```sh
$ ./byfn.sh down
```

## 生成加密文件并启动网络

在干净的环境中，使用以下四个命令启动我们的`v1.1 BYFN`网络： 

```sh
# 生成加密文件，证书、公钥、私钥
$ ./byfn.sh generate
# 启动网络，设置延时时间和 镜像版本
$ ./byfn.sh up -t 3000 -i 1.1.0
```

> **注意**：如果本地构建的`v1.1`镜像，则示例将使用它们。如果您遇到错误，请考虑清理本地构建的`v1.1`映像并再次运行该示例。这将从`docker hub`下载`v1.1`映像。

如果`BYFN`正确启动，你会看到：

```
===================== All GOOD, BYFN execution completed =====================
```

现在准备将我们的网络升级到`Hyperledger Fabric v1.2`。

## 获取最新示例代码

> **注意**：以下说明适用于最新发布的`v1.2.x`版本。请将`1.2.x`替换为正在测试的已发布版本的版本标识符。换句话说，如果正在测试第一个候选版本，请将“`1.2.x`”替换为“`1.2.0`”。

在完成本教程的其余部分之前，获取样本的`v1.2.x`版本非常重要，可以通过以下方式执行此操作：

```sh
$ git fetch origin
$ git checkout v1.2.x
```

## 想立即升级吗？

这里有一个脚本可以升级`BYFN`中的所有组件以及启用功能。如果正在运行生产网络，或者是网络某个部分的管理员，则此脚本可用作执行自己升级的模板。

然后，通过完成脚本中的步骤，并描述每个代码在升级过程中所执行的操作。

要运行该脚本，请发出以下命令：

```sh
# Note, replace '1.2.x' with a specific version, for example '1.2.0'.
# Don't pass the image flag '-i 1.2.x' if you prefer to default to 'latest' images.
# 如果您希望默认为“最新”图像，请不要传递图像标记'-i 1.2.x'。

$ ./byfn.sh upgrade -i 1.2.x
```

如果升级成功，您应该看到以下内容：

```sh
======== All GOOD, End-2-End UPGRADE Scenario execution completed ================
```

如果要手动升级网络，只需再次运行`./byfn.sh down`并执行步骤 `./byfn.sh upgrade -i 1.2.x`。然后继续下一部分。

# 升级`orderer`容器

`Orderer`容器应以**滚动方式升级**（一次一个）。在较高级别，`orderer`升级过程如下：

1. 停止（`stop`）`orderer`。
2. 备份`orderer`的分类帐和`MSP`。
3. 使用最新镜像重新启动`orderer`。
4. 验证升级完成。

由于利用`BYFN`，我们有一个独立`orderer`设置，因此，我们只会执行一次此过程。但是，在`Kafka`设置中，必须为每个`orderer`执行此过程。

> **注意**：本教程使用`docker`部署。对于本机部署，请使用发布工件中的`orderer`文件替换该文件。备份`orderer.yaml`并将其替换为发布工件中的`orderer.yaml`文件。然后将备份的`orderer.yaml`中的任何修改变量移植到新的变量。使用像`diff`这样的实用程序可能会有所帮助。`v1.2`中没有新的`orderer.yaml`配置参数，但最佳做法是将更改作为升级过程的一部分移植到新配置文件中。

## 停止 `orderer` 服务

通过**停止`orderer`**开始升级过程：

```sh
# 停止 orderer 服务
$ docker stop orderer.example.com
# 设置备份目录
$ export LEDGERS_BACKUP=./ledgers-backup

# 注意，将'1.2.x'替换为特定版本，例如'1.2.0'。
# 如果您希望默认使用系统上标记为“最新”的图像，请将IMAGE_TAG设置为“最新”。

# 设置升级后的 docker 镜像版本
$ export IMAGE_TAG=$(go env GOARCH)-1.2.0-stable
```

我们为目录创建了一个变量，用于放入备份文件，并设置想要升级到的`docker`镜像`IMAGE_TAG`。

## 备份`orderer`的分类账本和`MSP`

`orderer`停掉后，需要**备份其分类帐和MSP**：

```sh
$ mkdir -p $LEDGERS_BACKUP

# 下面的目录是 docker-compose-cli.yaml 中挂载的卷路径
$ docker cp orderer.example.com:/var/hyperledger/production/orderer/ ./$LEDGERS_BACKUP/orderer.example.com
```

在生产网络中，将以滚动方式为每个基于`kafka`的`orderer`重复该过程。

## 使用最新镜像重新启动`orderer`

现在使用我们的新的镜像版本**下载并重新启动`orderer`**：

```sh
$ docker-compose -f docker-compose-cli.yaml up -d --no-deps orderer.example.com
```

由于我们的示例使用“独立” `orderer` 服务，因此重新启动的 `orderer` 必须同步的网络中没有其他 `orderer` 。但是，在利用`Kafka`的生产网络中，最好的做法是 `peer channel fetch <blocknumber>` 在重新启动 `orderer` 后验证是否已赶上其他订货人。

# 升级 `peer` 容器

接下来，如何将对等容器升级到`Fabric v1.2`。与`orderer`一样，`docker peer` 镜像应以**滚动方式升级**（一次一个）。正如`orderer`升级期间提到的那样，`orderers`和`peer`可以**并行升级**，但是为了本教程的目的，我们已经将这些进程分开了。将执行以下步骤：

1. 停止(`stop`)`peer`。
2. 备份`peer`的分类账本和`MSP`。
3. 删除链码容器和镜像。
4. 使用最新镜像重新启动`peer`容器。
5. 验证升级完成。

我们的网络中有四个同行。我们将为每个对等体执行一次此过程，总共进行四次升级。

> **注意**：同样，本教程使用了`docker`部署。对于**本机** 部署，请`peer`使用发布工件中的文件替换该文件。备份`core.yaml`并使用发布工件中的替换它。将所有已修改变量从备份移植`core.yaml`到新变量 。使用类似的实用程序`diff`可能会有所帮助。

## 停止`peer`服务

让我们用以下命令**关闭第一个对等**容器： 

```sh
$ export PEER=peer0.org1.example.com
$ docker stop $PEER
```

## 备份`peer`的分类账本和`MSP`

**备份对等的分类帐和MSP**：

```sh
$ mkdir -p $LEDGERS_BACKUP
$ docker cp $PEER:/var/hyperledger/production ./$LEDGERS_BACKUP/$PEER
```

## 删除链码容器和镜像

在对等体停止并备份分类帐后，**删除对等链代码容器**：

```sh
$ CC_CONTAINERS=$(docker ps | grep dev-$PEER | awk '{print $1}')
$ if [ -n "$CC_CONTAINERS" ] ; then docker rm -f $CC_CONTAINERS ; fi
```

**删除对等体链代码镜像**：

```sh
CC_IMAGES=$(docker images | grep dev-$PEER | awk '{print $1}')
if [ -n "$CC_IMAGES" ] ; then docker rmi -f $CC_IMAGES ; fi
```

## 使用最新镜像重新启动`peer`容器

现在我们将使用v1.2图像标记重新启动对等体：

```sh
$ docker-compose -f docker-compose-cli.yaml up -d --no-deps $PEER
```

> **注意**：尽管`BYFN`支持使用`CouchDB`，但我们在本教程中选择了更简单的实现。但是，如果您使用的是`CouchDB`，请发出此命令而不是上面的命令：

```sh
$ docker-compose -f docker-compose-cli.yaml -f docker-compose-couch.yaml up -d --no-deps $PEER
```

> **注意**：无需重新启动链代码容器，当对等方获得链代码请求（调用或查询）时，它首先检查它是否有运行该链代码的副本。如果是这样，它就会使用它。否则，在这种情况下，对等体启动链码（如果需要，重建图像）。

## 验证升级完成

现在已完成第一个`peer`的升级，但在继续之前，请检查以确保通过链代码调用正确完成升级。继续转账从`a`到`b`使用这些命令：

```sh
$ docker-compose -f docker-compose-cli.yaml up -d --no-deps cli

$ docker exec -it cli bash

$ peer chaincode invoke -o orderer.example.com:7050  --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C mychannel -n mycc -c '{"Args":["invoke","a","b","10"]}'
```

查询显示`a`有一个值`90`，我们刚刚调用了它们转账 `10`。因此，一个查询`a`应该显示`80`。让我们来看看：

```sh
$ peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
```

应该看到以下内容：

```
Query Result: 80
```

验证对等方是否已正确升级后，请确保`exit` 在继续升级对等方之前发出一个离开容器的方法。可以通过重复上述过程并导出不同的`peer`对等名称来完成此操作。

```sh
export PEER=peer1.org1.example.com
export PEER=peer0.org2.example.com
export PEER=peer1.org2.example.com
```

> **注意**：在启用`v1.2`功能之前，必须升级所有对等体。

# 启用新的`v1.2`功能

尽管`Fabric`二进制文件可以并且应该以滚动方式进行升级，但在**启用功能之前完成二进制文件的升级**非常重要。在启用新功能之前未升级到`v1.2`的任何对等方节点**可能会崩溃**，以指示可能导致状态为`forl`的错误配置。如果订购者未升级到`v1.2`，则不会崩溃，也不会创建状态分叉（与从`v1.0.x`升级到`v1.1`不同）。尽管如此，在启用新功能之前，**将所有`peer`和`orderer`二进制文件升级到`v1.2`仍然是最佳做法**。

启用功能后，**它将成为该通道的永久记录**的一部分。这意味着即使在**禁用该功能**之后，**旧的二进制文件也将无法参与该通道**，因为它们无法处理超出阻止功能的块进入禁用它的块。因此，**一旦启用了功能，就不建议或不支持禁用它**。

因此，**将升级通道功能视为不可逆，一旦升级就不能回退**。请在测试设置中尝试新功能，并在继续在生产中启用它们之前充满信心。

通过通道配置启用交易功能。有关更新通道配置的更多信息，请查看[向](https://hyperledger-fabric.readthedocs.io/en/release-1.2/channel_update_tutorial.html)通道[添加组织](https://hyperledger-fabric.readthedocs.io/en/release-1.2/channel_update_tutorial.html) 或[更新通道配置](https://hyperledger-fabric.readthedocs.io/en/release-1.2/config_update.html)的文档。

`v1.2`的新功能位于`Application`通道组中（这会影响**对等网络**行为，例如对等方处理交易的方式）。与任何通道配置更新一样，我们必须遵循以下流程：

1. 获取最新的频道配置
2. 创建修改后的频道配置
3. 创建配置更新交易

通过重新发布`docker exec -it cli bash`进入`cli`容器。

## Application 组

要更改应用程序组的配置，请将环境变量设置为`Org1`：

```sh
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
export CH_NAME="mychannel"
```

## 获取最新的频道配置

接下来，获取最新的频道配置：

```sh
$ peer channel fetch config config_block.pb -o orderer.example.com:7050 -c $CH_NAME --tls --cafile $ORDERER_CA

$ configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json

$ jq .data.data[0].payload.data.config config_block.json > config.json
```

## 创建修改后的频道配置

创建修改后的频道配置：

```sh
jq -s '.[0] * {"channel_group":{"groups":{"Application": {"values": {"Capabilities": .[1]}}}}}' config.json ./scripts/capabilities.json > modified_config.json
```

注意我们在这里要改变的内容：`Capabilities` 被添加为`value` 的 `Application` 组下`channel_group`（在`mychannel`中）。

## 创建配置更新交易

创建配置更新交易：

```sh
$ configtxlator proto_encode --input config.json --type common.Config --output config.pb

$ configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb

$ configtxlator compute_update --channel_id $CH_NAME --original config.pb --updated modified_config.pb --output config_update.pb
```

`Org1`签署交易：

```sh
$ peer channel signconfigtx -f config_update_in_envelope.pb
```

和上面同样的方式进行操作其他组织，将环境变量设置为`Org2`： 

```sh
export CORE_PEER_LOCALMSPID="Org2MSP"

export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt

export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp

export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
```

`Org2`使用其签名提交配置更新交易： 

```sh
$ peer channel update -f config_update_in_envelope.pb -c $CH_NAME -o orderer.example.com:7050 --tls true --cafile $ORDERER_CA
```

到此，现在已启用`v1.2`功能。 

## 重新验证升级完成

让我们确保网络仍然畅通，运行一下命令：

```sh
$ peer chaincode invoke -o orderer.example.com:7050  --tls --cafile $ORDERER_CA  -C $CH_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'
```

然后查询值`a`，该值应显示值`70`。让我们来看看：

```sh
$ peer chaincode query -C $CH_NAME -n mycc -c '{"Args":["query","a"]}'
```

我们应该看到以下内容：

```
Query Result: 70
```

> **注意**：虽然网络中的所有对等二进制文件都应该在此之前进行升级，但是在加入v1.1.x对等体的通道上启用功能要求将导致对等体崩溃。这种崩溃行为是故意的，因为它表明可能导致状态分叉的配置错误。