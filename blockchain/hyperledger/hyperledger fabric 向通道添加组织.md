# `hyperledger fabric` 向通道添加组织

在开始之前，请确保已按照安装**示例，二进制文件和Docker镜像**以及符合本文档版本的二进制文件。特别是，`fabric-samples`文件夹版本必须包含`eyfn.sh`（扩展你的第一个网络）脚本及其相关脚本。

**本教程用作构建你的第一个网络（`BYFN`）教程的扩展**，并将演示向`BYFN`自动生成的应用程序通道（`mychannel`）添加新组织`Org3`。它假设对`BYFN`有很强的理解，包括上述实用程序的用法和功能。

虽然仅关注此处新组织的扩展集成，但在执行其他**通道配置更新**（例如更新修改策略或更改批量大小）时可以采用相同的方法。要了解有关通道配置更新的过程和可能性的更多信息，请查看[更新通道配置](https://hyperledger-fabric.readthedocs.io/en/latest/config_update.html)。同样值得注意的是，像这里演示的频道配置更新通常是**组织管理员**（而不是链码或应用程序开发人员）的责任。

> **注意**：在继续之前，请确保自动`byfn.sh`脚本在计算机上运行时没有错误。如果已将**二进制文件和相关工具（`cryptogen`，`configtxgen`等）导出到`PATH`变量中**，则可以相应地修改命令而不传递完全限定的路径。

## 设置环境

将在本地`Fabric-samples`克隆中的第一个网络子目录的根目录下运行。立即切换到该目录。还需要打开一些额外的终端以方便使用。

```sh
$ cd fabric-samples/first-network
```

首先，**使用`byfn.sh`脚本进行容器或数据、镜像清理工作**。此命令将**终止所有活动或过时的`docker`容器并删除以前生成的配置工件**。为了执行通道配置更新任务，无需关闭`Fabric`网络。但是，为了本教程的目的，希望从已知的初始状态进行操作。因此，让我们运行以下命令来清理以前的所有环境：

```sh
$ ./byfn.sh down
```

现在生成默认的`BYFN`配置工件：

```sh
$ ./byfn.sh generate
```

并使用`CLI`容器中的脚本执行启动网络：

```sh
$ ./byfn.sh up
```

现在机器上运行了一个干净的`BYFN`版本，可以使用两种不同的路径。首先，提供一个完全注释的脚本，它将执行配置事务更新以将`Org3`引入网络。

此外，将显示相同过程的“**手动**”版本，显示每个步骤并解释它完成的内容（因为展示如何在此手动过程之前关闭网络，还可以运行脚本然后查看每个步骤）。

## 用脚本将`Org3`加入到通道

首先使用如下脚本，只需发出以下命令：

```sh
$ ./eyfn.sh up
```

这里的输出非常值得一读。可以看到**添加了`Org3`加密文件，创建并签署了配置更新**，然后安装了链码以允许`Org3`执行分类帐查询。

如果一切顺利，你会得到这样的信息：

```sh
========= All GOOD, EYFN test execution completed ===========
```

通过执行以下命令（而不是`./byfn.sh up`），`eyfn.sh`可以与`byfn.sh`使用相同的`Node.js`链码和数据库选项：

```sh
$ ./byfn.sh up -c testchannel -s couchdb -l node
```

然后：

```sh
$ ./eyfn.sh up -c testchannel -s couchdb -l node
```

对于那些想要仔细研究这个过程的人来说，文档的其余部分将向你展示进行频道更新的每个命令以及它的作用。

## 手动加入`Org3`到频道

下面列出的手动步骤`cli`和`Org3 cli`容器中的`CORE_LOGGING_LEVEL`设置为`DEBUG`。

对于`cli`容器，可以通过修改`first-network`目录中的`docker-compose-cli.yaml`文件来设置它。例如：

```yml
cli:
  container_name: cli
  image: hyperledger/fabric-tools:$IMAGE_TAG
  tty: true
  stdin_open: true
  environment:
    - GOPATH=/opt/gopath
    - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
    #- CORE_LOGGING_LEVEL=INFO
    - CORE_LOGGING_LEVEL=DEBUG
```

对于`Org3cli`容器，可以通过修改`first-network`目录中的`docker-compose-org3.yaml`文件来设置它。例如：

```sh
Org3cli:
  container_name: Org3cli
  image: hyperledger/fabric-tools:$IMAGE_TAG
  tty: true
  stdin_open: true
  environment:
    - GOPATH=/opt/gopath
    - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
    #- CORE_LOGGING_LEVEL=INFO
    - CORE_LOGGING_LEVEL=DEBUG
```

如果使用过`eyfn.sh`脚本，则需要关闭网络。这将关闭网络，删除所有容器并撤消我们为添加`Org3`所做的工作。可以通过执行命令：

```sh
$ ./eyfn.sh down
```

当网络关闭时，请重新启动它：

```sh
$ ./byfn.sh generate

$ ./byfn.sh up
```

这将使网络恢复到执行`eyfn.sh`脚本之前的状态。现在准备手动添加`Org3`了。作为第一步，需要生成`Org3`的加密文件。

## 生成`Org3`加密文件

在另一个终端中，从`first-network`切换到`org3-artifacts`子目录。

```sh
$ cd org3-artifacts
```

这里有两个`yaml`文件：`org3-crypto.yaml`和`configtx.yaml`。首先，为`Org3`生成加密文件：

```sh
$ ../../bin/cryptogen generate --config=./org3-crypto.yaml
```

此命令读入新加密`yaml`文件 `org3-crypto.yaml` 并利用`cryptogen`为`Org3 CA`以及绑定到此新组织的两个对等方**生成密钥和证书**。与`BYFN`实现一样，此加密文件将放入当前工作目录（在示例中为`org3-artifacts`）中新生成的`crypto-config`文件夹中。

现在使用`configtxgen`实用程序在`JSON`中打印出特定于`Org3`的配置材料。通过告诉工具在当前目录中查找它需要提取的`configtx.yaml`文件来执行命令。

```sh
$ export FABRIC_CFG_PATH=$PWD && ../../bin/configtxgen -printOrg Org3MSP > ../channel-artifacts/org3.json
```

上面的命令创建一个`JSON`文件 `org3.json`，并将其输出到`first-network`根目录下的`channel-artifacts`子目录中。此文件包含`Org3`的**背书策略**定义，以及以`base 64`格式显示的三个重要证书：**管理员用户证书（稍后将充当`Org3`的管理员），`CA`根证书和`TLS`根目录证书**。在接下来的步骤中，将此`JSON`文件添加到通道配置。

最后的工作是**将`Orderer Org`的`MSP`材料移植到`Org3` `crypto-config`目录**中。特别是，**关注的是`Orderer`的`TLS`根证书，它将允许`Org3`实体与网络订购节点之间的安全通信**。

```sh
$ cd ../ && cp -r crypto-config/ordererOrganizations org3-artifacts/crypto-config/
```

## 准备`CLI`环境

更新过程使用**配置转换器**工具 `configtxlator`。此工具提供**独立于`SDK`的无状态`REST API`**。此外，它还提供`CLI`，以简化`Fabric`网络中的配置任务。该工具允许在**不同的等效数据表示/格式之间轻松转换**（在本例中，在`protobufs`和`JSON`之间）。此外，该工具可以**根据两个通道配置之间的差异计算配置更新交易**。

首先，执行命令进入`CLI`容器。这个容器已经安装了`BYFN`加密配置库，可以访问两个原始对等组织和`Orderer`组织的`MSP`资料。引导标识是`Org1`管理员用户，这意味着想要充当`Org2`的任何步骤都需要导出特定于`MSP`的环境变量。

```sh
$ docker exec -it cli bash
```

导出`ORDERER_CA`和`CHANNEL_NAME`变量：

```sh
$ export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  && export CHANNEL_NAME=mychannel
```

检查以确保已正确设置变量：

```sh
$ echo $ORDERER_CA && echo $CHANNEL_NAME
```

> **注意**：由于任何原因需要重新启动`CLI`容器，则还需要**重新导出两个环境变量** `ORDERER_CA`和`CHANNEL_NAME`。

## 获取配置

现在有一个`CLI`容器，其中包含两个关键环境导出变量 `ORDERER_CA`和`CHANNEL_NAME`。让我们去获取频道的最新配置块 `mychannel`。

**必须提取配置的最新版本的原因是因为通道配置元素是版本化的**。版本控制很重要，原因有几个。它可以**防止重复**或回放配置更改（例如，恢复到使用旧`CRL`的通道配置将代表安全风险）。此外，它还有助于**确保并发性**（如果要从通道中删除组织，例如，在添加新组织后，版本控制将有助于防止删除两个组织，而不仅仅是要删除的组织）。

```sh
$ peer channel fetch config config_block.pb -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA
```

此命令**将二进制`protobuf`通道配置块保存到`config_block.pb`**。请注意，名称和文件扩展名的选择是任意的。但是，建议遵循一个约定标识**所表示的对象类型及其编码（`protobuf`或`JSON`）的约定**。

当发出`peer channel fetch`命令时，终端中有相当数量的日志输出。日志中的最后一行很有意义：

```sh
2017-11-07 17:17:57.383 UTC [channelCmd] readBlock -> DEBU 011 Received block: 2
```

这告诉我们`mychannel`的**最新配置块实际上是块2**，而**不是创世块**。默认情况下，`peer channel fetch config`命令返回目标通道的**最新配置块**，在这种情况下是第三个块。这是因为`BYFN`脚本在两个单独的通道更新事务中为我们的两个组织（`Org1`和`Org2`）定义了锚点对等体。

因此，有以下配置顺序：

+ **块0**：创世块
+ **块1**：`Org1`锚点对等更新
+ **块2**：`Org2`锚点对等更新

## 将配置转换为`JSON`

现在将**使用`configtxlator`工具将此通道配置块解码为`JSON`格式**，这样可以由人类读取和修改。还必须**删除与我们想要进行的更改无关的所有标头，元数据，创建者签名**等。通过`jq`工具实现这一目标：

```sh
$ configtxlator proto_decode --input config_block.pb --type common.Block | jq .data.data[0].payload.data.config > config.json
```

这将得到一个精简的`JSON`对象`config.json`，它位于`fabric-samples`文件夹中，它将作为配置更新的基线。

花一点时间在选择的文本编辑器（或浏览器）中打开此文件。即使在完成本教程之后，也值得研究它，因为它揭示了**底层配置结构和可以进行的其他类型的通道更新**。我们将在[更新频道配置](https://hyperledger-fabric.readthedocs.io/en/latest/config_update.html)中更详细地讨论它们。

## 添加`Org3`加密配置

> 无论尝试进行何种配置更新，采取的步骤几乎完全相同。选择在本教程中添加组织，因为它是可以尝试的最复杂的通道配置更新之一。

将再次使用`jq`工具将`Org3`配置定义 `org3.json` ，添加加到通道的应用程序组字段，并命名输出`modified_config.json`

```sh
jq -s '.[0] * {"channel_group":{"groups":{"Application":{"groups": {"Org3MSP":.[1]}}}}}' config.json ./channel-artifacts/org3.json > modified_config.json
```

现在，在`CLI`容器中，有两个`JSON`文件 `config.json`和`modified_config.json`。**初始文件仅包含`Org1`和`Org2`配置，而`modified`文件包含所有三个`Orgs`。此时，只需重新编码这两个`JSON`文件并计算增量即可**。

首先，将`config.json`转换回名为`config.pb`的`protobuf`：

```sh
$ configtxlator proto_encode --input config.json --type common.Config --output config.pb
```

接下来，将`modified_config.json`编码为`modified_config.pb`的`protobuf`：

```sh
$ configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb
```

现在使用`configtxlator`来计算这两个配置`protobufs`之间的**增量**。此命令将输出名为`org3_update.pb`的新`protobuf`二进制文件：

```sh
$ configtxlator compute_update --channel_id $CHANNEL_NAME --original config.pb --updated modified_config.pb --output org3_update.pb
```

这个新的`proto`   `org3_update.pb` 包含`Org3`和`Org1`和`Org2`材质的高级指针。能够丢弃`Org1`和`Org2`的广泛`MSP`材料和修改策略信息，因为这些数据已经存在于通道的创世块中。因此，只需要两种配置之间的增量。

在提交频道更新之前，需要执行一些最终步骤。首先，将此对象解码为可编辑的`JSON`格式并将其命名为`org3_update.json`：

```sh
$ configtxlator proto_decode --input org3_update.pb --type common.ConfigUpdate | jq . > org3_update.json
```

现在，有一个解码的更新文件 `org3_update.json` ，需要**包装一个信封消息**。这一步将返回之前剥离的标题字段。将此文件命名为`org3_update_in_envelope.json`：

```sh
$ echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":'$(cat org3_update.json)'}}}' | jq . > org3_update_in_envelope.json
```

使用最终形成的`JSON` 文件 `org3_update_in_envelope.json` ，将最后一次利用`configtxlator`工具并将其转换为`Fabric`所需的完全成熟的`protobuf`格式。将命名最终更新对象`org3_update_in_envelope.pb`：

```sh
$ configtxlator proto_encode --input org3_update_in_envelope.json --type common.Envelope --output org3_update_in_envelope.pb
```

## 签名并提交配置更新

现在在`CLI`容器中有一个`protobuf`二进制文件`org3_update_in_envelope.pb`。但是，在将配置写入分类帐之前，需要来自**必需管理员用户的签名**。通道应用程序组的修改策略（`mod_policy`）设置为默认值`MAJORITY`，这意味着需要多数**现有组织管理员对其进行签名**。因为只有两个组织`Org1`和`Org2`，而多数就是现有的两个组织，我们**需要它们两个的签名**。如果**没有这两个签名，订购服务将拒绝交易，因为未能履行该政策**。

首先，将此**更新二进制文件作为`Org1`管理员签名**。请记住，`CLI`容器当前是配置变量使用`Org1 MSP`加密文件引导的，因此只需要发出`peer channel signconfigtx`命令：

```sh
$ peer channel signconfigtx -f org3_update_in_envelope.pb
```

最后一步是切换`CLI`容器的身份 `Org2 Admin`用户。通过**导出特定于`Org2 MSP`的四个环境变量**来实现此目的。

> 在组织之间切换以签署配置交易（或执行任何其他操作）并不反映真实的`Fabric`操作。永远不会使用整个网络的加密材料安装单个容器。相反，配置更新需要安全地带外传递给`Org2`管理员进行检查和批准。

导出`Org2`环境变量：

```sh
# you can issue all of these commands at once

export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
```

最后，将发出`peer channel update`命令，**`Org2`管理员签名将附加到此调用，因此无需再次手动签署`protobuf`**。

> 即将进行的订购服务更新调用将进行一系列**系统签名和策略检查**。因此，可能会发现日志流和检查订购节点的日志很有用。从另一个`shell`发出`docker logs -f orderer.example.com`命令以显示它们。

发送更新调用：

```sh
$ peer channel update -f org3_update_in_envelope.pb -c $CHANNEL_NAME -o orderer.example.com:7050 --tls --cafile $ORDERER_CA
```

如果更新已成功提交，应该会看到类似于以下内容的消息摘要指示：

```sh
2018-02-24 18:56:33.499 UTC [msp/identity] Sign -> DEBU 00f Sign: digest: 3207B24E40DE2FAB87A2E42BC004FEAA1E6FDCA42977CB78C64F05A88E556ABA
```

还将看到配置事务的提交：

```sh
2018-02-24 18:56:33.499 UTC [channelCmd] update -> INFO 010 Successfully submitted channel update
```

成功的通道更新请求向通道上的所有对等体返回新块 `block 5`。如果你还记得，**块`0-2`是初始通道配置，而块`3-4`是`mycc`链码的实例化和调用**。因此，**块5用作最近的通道配置**，其中`Org3`现在在通道上定义。

检查`peer0.org1.example.com`的日志：

```sh
$ docker logs -f peer0.org1.example.com
```

如果要检查其内容，请按照演示过程获取和解码新配置块。

## 配置选举领导者

