# `hyperledger fabric` 中使用私有数据

本文为组织的授权同行节点在区块链网络上提供私有数据的存储和检索。 本文中的信息假定你了解私有数据存储及其用例。有关更多信息，请查看[私人数据](hyperledger%20fabric%20关键概念.md#私有数据)。

本文将指导完成以下步骤，以练习使用`Fabric`定义，配置和使用私有数据：

+ 构建集合定义`JSON`文件
+ 使用链代码`API`读取和写入私有数据
+ 使用集合安装和实例化链代码
+ 存储私人数据
+ 将私有数据作为授权对等体查询
+ 将私有数据查询为未授权的对等体
+ 清除私人数据
+ 使用带有索引的私有数据
+ 其他资源

本文将使用在 **构建第一个网络（`BYFN`）**教程网络上运行的[弹珠私有数据示例](https://github.com/hyperledger/fabric-samples/tree/master/chaincode/marbles02_private)来演示如何创建、部署和使用私有数据集合。弹珠私有数据样本将部署到[构建第一个网络](https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html)（`BYFN`）教程网络。应该已经完成了[安装样本，二进制文件和Docker镜像](https://hyperledger-fabric.readthedocs.io/en/latest/install.html)的任务；但是，运行`BYFN`教程不是本文的必要条件。相反，本文中提供了必要的命令来使用网络。将描述每个步骤中发生的事情，使得在不实际运行示例的情况下理解教程成为可能。

# 构建集合定义`JSON`文件

将**通道上的数据私有化的第一步是构建一个集合定义**，用于**定义对私有数据的访问**。

集合定义描述了**谁可以持久保存数据**，数据**分配到多少对等节点，传播私有数据需要多少对等节点，以及私有数据在私有数据库中保留多长时间**。稍后，将演示如何使用链码`API PutPrivateData`和`GetPrivateData`将集合映射到受保护的私有数据。

## 集合定义由五个属性组成

- `name`：集合的名称。
- `policy`：定义允许持久**保存**集合数据的组织**对等方**。
- `requiredPeerCount`：传播私有数据所需的**对等节点数量**，作为**认可链码的条件**
- `maxPeerCount`：出于数据冗余目的，当前认可对等方将尝试将数据分发到的其他对等方的数量。如果支持**对等体发生故障**，那么如果有请求提取私有数据，则**这些其他对等体在提交时可用**。
- `blockToLive`：对于**非常敏感**的信息，例如定价或个人信息，此值表示**数据应以块的形式存在于私有数据库中的时间**。数据将在私有数据库上为此**指定数量的块生效，之后将被清除**，从而使这些数据从**网络中过时**。要**无限期地保留私有数据**，即永远不会清除私有数据，请**将`blockToLive`属性设置为`0`**。

## 私有数据集合定义

为了说明私有数据的使用，弹珠私有数据示例包含两个私有数据集合定义：`collectionMarbles`和`collectionMarblePrivateDetails`。`collectionMarbles`定义中的`policy`属性允许通道的所有成员（`Org1`和`Org2`）在**私有数据库中拥有私有数据**。`collectionMarblesPrivateDetails`集合仅允许`Org1`的成员在其私有数据库中拥有私有数据。有关构建策略定义的更多信息，请参阅[认可策略](https://hyperledger-fabric.readthedocs.io/en/latest/endorsement-policies.html)主题。

```json
// collections_config.json

[
  {
       "name": "collectionMarbles",
       "policy": "OR('Org1MSP.member', 'Org2MSP.member')",
       "requiredPeerCount": 0,
       "maxPeerCount": 3,
       "blockToLive":1000000
  },

  {
       "name": "collectionMarblePrivateDetails",
       "policy": "OR('Org1MSP.member')",
       "requiredPeerCount": 0,
       "maxPeerCount": 3,
       "blockToLive":3
  }
]
```

这些策略要**保护的数据以链代码映射**，稍后将在本教程中显示。

当**使用`peer chaincode instantiate`命令在通道上[实例化其关联的链代码](http://hyperledger-fabric.readthedocs.io/en/latest/commands/peerchaincode.html#peer-chaincode-instantiate)时，此集合定义文件部署在通道上**。有关此过程的更多详细信息，请参见下面的第3节。

# 使用链代码`API`读取和写入私有数据

理解**如何在通道上私有化数据**的下一步是**在链代码中构建数据定义**。弹珠私有数据样本根据数据的访问方式将私有数据划分为**两个单独的数据定义**。

```go
// Peers in Org1 and Org2 will have this private data in a side database
type marble struct {
  ObjectType string `json:"docType"`
  Name       string `json:"name"`
  Color      string `json:"color"`
  Size       int    `json:"size"`
  Owner      string `json:"owner"`
}

// Only peers in Org1 will have this private data in a side database
type marblePrivateDetails struct {
  ObjectType string `json:"docType"`
  Name       string `json:"name"`
  Price      int    `json:"price"`
}
```

具体**访问私有数据将受到如下限制**：

+ `name, color, size, and owner` 将对渠道的**所有成员可见**（`Org1`和`Org2`）
+ `price`仅对`Org1`的成员可见

因此，在弹珠私人数据示例中定义了两组不同的私人数据。此数据到**限制其访问的集合策略**的映射由链代码`API`控制。具体来说，**使用集合定义读取和写入私有数据是通过调用`GetPrivateData()`和`PutPrivateData()`来执行**的，这可以在这里找到。

下图说明了弹珠私有数据示例使用的私有数据模型：

![_images/SideDB-org1.png](https://hyperledger-fabric.readthedocs.io/en/latest/_images/SideDB-org1.png)

![_images/SideDB-org2.png](https://hyperledger-fabric.readthedocs.io/en/latest/_images/SideDB-org2.png)

## 读取集合数据

使用链代码`API`的**方法 `GetPrivateData()`来查询数据库中的私有数据**。`GetPrivateData()`接受两个参数，**集合名称**和**数据键**。回想一下集合`collectionMarbles`允许`Org1`和`Org2`的成员在数据库中拥有私有数据，集合`collectionMarblePrivateDetails`只允许`Org1`的成员在数据库中拥有私有数据。有关实现细节，请参阅以下两个[弹珠数私有数据函数](https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02_private/go/marbles_chaincode_private.go)：

+ **`readMarble`** ：用于查询`name, color, size and owner`属性的值
+ **`readMarblePrivateDetails`** ：用于查询`price`属性的值

在本教程后面使用`peer`命令发出数据库查询时，将调用这两个函数。

## 写入私有数据

使用`chaincode API`的**方法 `PutPrivateData()`将私有数据存储到私有数据库**中。`API`还需要**集合的名称**。由于弹珠私有数据样本包含两个不同的集合，因此在链代码中调用两次：

+ 使用名为`collectionMarbles`的集合编写私有数据`name, color, size and owner`
+ 使用名为`collectionMarblePrivateDetails`的集合编写私有数据`price`

例如，在`initMarble`函数的以下片段中，`PutPrivateData()`被调用两次，每个私有数据集一次。

```go
// ==== Create marble object and marshal to JSON ====
objectType := "marble"
marble := &marble{objectType, marbleName, color, size, owner}
marbleJSONasBytes, err := json.Marshal(marble)
if err != nil {
    return shim.Error(err.Error())
}
//Alternatively, build the marble json string manually if you don't want to use struct marshalling
//marbleJSONasString := `{"docType":"Marble",  "name": "` + marbleName + `", "color": "` + color + `", "size": ` + strconv.Itoa(size) + `, "owner": "` + owner + `"}`
//marbleJSONasBytes := []byte(str)

// === Save marble to state ===
err = stub.PutPrivateData("collectionMarbles", marbleName, marbleJSONasBytes)
if err != nil {
    return shim.Error(err.Error())
}

// ==== Save marble private details ====
objectType = "marblePrivateDetails"
marblePrivateDetails := &marblePrivateDetails{objectType, marbleName, price}
marblePrivateDetailsBytes, err := json.Marshal(marblePrivateDetails)
if err != nil {
    return shim.Error(err.Error())
}
err = stub.PutPrivateData("collectionMarblePrivateDetails", marbleName, marblePrivateDetailsBytes)
if err != nil {
    return shim.Error(err.Error())
}
```

总而言之，上面针对`collection.json`的策略定义允许`Org1`和`Org2`中的**所有对等体**都可以在其私有数据库中**存储和交易（认可，提交，查询）**弹珠私有数据`name, color, size, owner`。但只有`Org1`中的同行可以在另外的私有数据库中存储和`price`私有数据。

作为额外的**数据隐私优势**，由于正在使用集合，因此只有**私有数据哈希值**通过`orderer`，而不是私有数据本身，从而**使私有数据对`orderer`保密**。

# 启动网络

现在准备逐步完成一些演示使用私有数据的命令。

在下面安装和实例化弹珠私有数据链代码之前，我们需要启动`BYFN`网络。以下命令将**终止所有活动或过时的`docker`容器并删除以前生成的配置工件**。因此，运行以下命令来清理以前的所有环境：

```sh
$ cd fabric-samples/first-network
$ ./byfn.sh down
```

如果之前已经完成了本教程，那么还需要**删除大理石私有数据链代码的底层`docker`容器**。运行以下命令来清理以前的环境：

```sh
$ docker rm -f $(docker ps -a | awk '($2 ~ /dev-peer.*.marblesp.*/) {print $1}')
$ docker rmi -f $(docker images | awk '($1 ~ /dev-peer.*.marblesp.*/) {print $3}')
```

通过运行以下命令，使用`CouchDB`启动`BYFN`网络：

```sh
$ ./byfn.sh up -c mychannel -s couchdb
```

这将创建一个简单的`Fabric`网络，其中包含一个名为`mychannel`的通道，其中包含两个组织（每个组织维护两个对等节点）和一个订购服务，同时使用`CouchDB`作为状态数据库。`LevelDB`或`CouchDB`可以与数据集合一起使用。选择`CouchDB`来演示如何将索引与私有数据一起使用。

> **注意**：要使集合起作用，必须正确配置跨组织的`gossip`。请参阅关于[`Gossip`数据传播协议](https://hyperledger-fabric.readthedocs.io/en/latest/gossip.html)的文档，特别注意“**对等锚点**”部分。本文没有关注`gossip`，因为它已经在`BYFN`示例中进行了配置，但在配置频道时，`gossip`锚定对等节点对于配置集合以使其正常工作至关重要。

# 安装和实例化使用集合的链码

客户端应用程序通过链码与区块链分类帐交互。因此，需要在每个将执行和支持交易的对等体上安装和实例化链码。`Chaincode`安装在对等体上，然后使用[`Peer Commands`](https://hyperledger-fabric.readthedocs.io/en/latest/peer-commands.html)实例化到通道上。

## 在所有对等体上安装链码

如上所述，`BYFN`网络包括两个组织`Org1`和`Org2`，每个组织有两个对等节点组。因此，链码必须安装在四个对等体上：

- `peer0.org1.example.com`
- `peer1.org1.example.com`
- `peer0.org2.example.com`
- `peer1.org2.example.com`

[`peer chaincode install`](http://hyperledger-fabric.readthedocs.io/en/master/commands/peerchaincode.html?%20chaincode%20instantiate#peer-chaincode-install)命令用来在每个对等体上安装`Marbles`链代码。

### 进入`CLI`容器

假设已启动`BYFN`网络，请输入`CLI`容器：

```sh
$ docker exec -it cli bash
```

命令输入后将看到如下效果：

```sh
root@81eac8493633:/opt/gopath/src/github.com/hyperledger/fabric/peer#
```

### 安装链码

+ 使用以下命令将`Marts`链代码从`git`存储库安装到`BYFN`网络中的`peer`节点 `peer0.org1.example.com`上。（默认情况下，启动`BYFN`网络后，设置操作的激活对等体为：`CORE_PEER_ADDRESS = peer0.org1.example.com:7051`）：

  ```sh
  $ CORE_PEER_ADDRESS=peer0.org1.example.com:7051
  $ peer chaincode install -n marblesp -v 1.0 -p github.com/chaincode/marbles02_private/go/
  ```

  完成后你应该看到类似的东西：

  ```sh
  install -> INFO 003 Installed remotely response:<status:200 payload:"OK" >
  ```

+ 使用`CLI`将活动对等方切换到`Org1`中的第二个对等方并安装链代码。将以下整个命令块复制并粘贴到`CLI`容器中并运行它们：

  ```sh
  export CORE_PEER_ADDRESS=peer1.org1.example.com:7051
  peer chaincode install -n marblesp -v 1.0 -p github.com/chaincode/marbles02_private/go/
  ```

+ 使用`CLI`切换到`Org2`。将以下命令块作为一个组复制并粘贴到对等容器中，并立即运行它们：

  ```sh
  export CORE_PEER_LOCALMSPID=Org2MSP
  export PEER0_ORG2_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
  export CORE_PEER_TLS_ROOTCERT_FILE=$PEER0_ORG2_CA
  export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
  ```

+ 将活动对等体切换到`Org2`中的第一个对等体并安装链码：

  ```sh
  export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
  peer chaincode install -n marblesp -v 1.0 -p github.com/chaincode/marbles02_private/go/
  ```

+ 将活动对等体切换到`org2`中的第二个对等体并安装链码：

  ```sh
  export CORE_PEER_ADDRESS=peer1.org2.example.com:7051
  peer chaincode install -n marblesp -v 1.0 -p github.com/chaincode/marbles02_private/go/
  ```

## 实例化通道上的链码

使用[`peer chaincode instantiate`](http://hyperledger-fabric.readthedocs.io/en/master/commands/peerchaincode.html?%20chaincode%20instantiate#peer-chaincode-instantiate)命令在通道上实例化大理石链代码。要在通道上配置链码集合，请在示例中指定标志`--collections-config`以及集合`JSON`文件的名称`collections_config.json`。

运行以下命令以实例化`BYFN`通道`mychannel`上的弹珠私有数据链代码：

```sh
$ export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C mychannel -n marblesp -v 1.0 -c '{"Args":["init"]}' -P "OR('Org1MSP.member','Org2MSP.member')" --collections-config  $GOPATH/src/github.com/chaincode/marbles02_private/collections_config.json
```

> **注意**：指定`--collections-config`标志的值时，需要指定`collections_config.json`文件的完全限定路径。例如：`--collections-config $GOPATH/src/github.com/chaincode/xx/collections_config.json`

实例化成功完成后，应该看到类似于：

```sh
[chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
[chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
```

# 存储私有数据

作为`Org1`的成员，**有权**与大理石私有数据示例中的所有私有数据进行交易，切换回`Org1`对等体并提交添加大理石的请求。

将以下命令集复制并粘贴到`CLI`命令行：

```sh
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID=Org1MSP
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export PEER0_ORG1_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

调用大理石链码`initMarble`函数，该函数创建一个**带有私有数据**的大理石 名称`marble1`由`tom`拥有，颜色为`blue`，大小为`35`，价格为`99`。在之前，私有数据`price`将与公共数据`name, owner, color, size`分开存储。出于这个原因`initMarble`函数调用`PutPrivateData()` 接口`API`两次以保留私有数据。

```sh
$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["initMarble","marble1","blue","35","tom","99"]}'
```

你应该看到类似的结果：

```sh
[chaincodeCmd] chaincodeInvokeOrQuery->INFO 001 Chaincode invoke successful. result: status:200
```

# 用`授权`对等体查询私有数据

集合定义允许`Org1`和`Org2`的**所有成员**在其数据库中具有`name, color, size, owner`私有数据，但只有`Org1`中的对等体可以在其数据库中具有`price`私有数据。作为`Org1`中的授权对等体，下面将查询两组私有数据。

## 代码分析

第一个**查询**命令调用`readMarble`函数，该函数将`collectionMarbles`作为参数传递。

```go
// ===============================================
// readMarble - read a marble from chaincode state
// ===============================================

func (t *SimpleChaincode) readMarble(stub shim.ChaincodeStubInterface, args []string) pb.Response {
     var name, jsonResp string
     var err error
     if len(args) != 1 {
             return shim.Error("Incorrect number of arguments. Expecting name of the marble to query")
     }

     name = args[0]
     valAsbytes, err := stub.GetPrivateData("collectionMarbles", name) //get the marble from chaincode state

     if err != nil {
             jsonResp = "{\"Error\":\"Failed to get state for " + name + "\"}"
             return shim.Error(jsonResp)
     } else if valAsbytes == nil {
             jsonResp = "{\"Error\":\"Marble does not exist: " + name + "\"}"
             return shim.Error(jsonResp)
     }

     return shim.Success(valAsbytes)
}
```

第二个**查询**命令调用`readMarblePrivateDetails`函数，该函数将`collectionMarblePrivateDetails`作为参数传递。

```go
// ===============================================
// readMarblePrivateDetails - read a marble private details from chaincode state
// ===============================================

func (t *SimpleChaincode) readMarblePrivateDetails(stub shim.ChaincodeStubInterface, args []string) pb.Response {
     var name, jsonResp string
     var err error

     if len(args) != 1 {
             return shim.Error("Incorrect number of arguments. Expecting name of the marble to query")
     }

     name = args[0]
     valAsbytes, err := stub.GetPrivateData("collectionMarblePrivateDetails", name) //get the marble private details from chaincode state

     if err != nil {
             jsonResp = "{\"Error\":\"Failed to get private details for " + name + ": " + err.Error() + "\"}"
             return shim.Error(jsonResp)
     } else if valAsbytes == nil {
             jsonResp = "{\"Error\":\"Marble private details does not exist: " + name + "\"}"
             return shim.Error(jsonResp)
     }
     return shim.Success(valAsbytes)
}
```

## 执行查询

+ 查询`marble1`作为`Org1`成员的`name, color, size and owner`私有数据：

  ```sh
  $ peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarble","marble1"]}'
  ```

  应该看到以下结果：

  ```sh
  {"color":"blue","docType":"marble","name":"marble1","owner":"tom","size":35}
  ```

+ 查询`marble1`的`price`私有数据作为`Org1`的成员：

  ```sh
  $ peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'
  ```

  应该看到以下结果：

  ```sh
  {"docType":"marblePrivateDetails","name":"marble1","price":99}
  ```

# 用`未授权`的对等体查询私有数据

现在将切换到`Org2`的对等体成员，该成员在其边数据库中具有大理石私有数据`name, color, size, owner`，但在其数据库中没有大理石`price`私有数据，将查询这两组私有数据。

## 切换到`Org2`中的对等体

从`docker`容器内部，运行以下命令切换到**未经授权**访问大理石`price`私有数据的对等方：

```sh
export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
export CORE_PEER_LOCALMSPID=Org2MSP
export PEER0_ORG2_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_TLS_ROOTCERT_FILE=$PEER0_ORG2_CA
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
```

## 查询`Org2`被授权的私有数据

`Org2`中的对等体应该在侧数据库中拥有第一组大理石私有数据（`name, color, size and owner`），并且可以使用`readMarble()`函数访问它，该函数使用`collectionMarbles`参数调用。

```sh
$ peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarble","marble1"]}'
```

应该看到类似于以下结果的东西：

```sh
{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"tom"}
```

## 查询`Org2`未被授权的私有数据

`Org2`中的对等方在其边数据库中没有大理石`price`私有数据。当尝试查询此数据时，**会返回与公共状态匹配的密钥的哈希值，但不会拥有私有状态**。

```sh
$ peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'
```

应该看到类似于的结果：

```sh
{"Error":"Failed to get private details for marble1: GET_STATE failed:
transaction ID: b04adebbf165ddc90b4ab897171e1daa7d360079ac18e65fa15d84ddfebfae90:
Private data matching public hash version is not available. Public hash
version = &version.Height{BlockNum:0x6, TxNum:0x0}, Private data version =
(*version.Height)(nil)"}
```

**`Org2`的成员只能看到私有数据的公共哈希值**。

# 清除私有数据

对于**私有数据只需要在分类账上直到可以复制到离线数据库中**的用例，可以**在一定数量的块之后“清除”数据，只留下数据的哈希值**。作为交易的**不可改变的证据**。

可能存在私人数据，包括个人或机密信息，例如示例中的定价数据，交易方不希望在渠道上向其他组织披露。因此，它**具有有限的寿命**，并且可以在区块链上使用**集合定义中的`blockToLive`属性**在指定数量的块上保持不变之后**进行清除**。

我们的`collectionMarblePrivateDetails`定义的`blockToLive`属性值为`3`，这意味着这些数据将存放在侧数据库中**三个块**，然后它**将被清除**。将所有部分绑定在一起，回想一下这个集合定义`collectionMarblePrivateDetails`在调用`PutPrivateData()` 方法并将`collectionMarblePrivateDetails`作为参数传递时，与`initMarble()`函数中的私有数据相关联。

逐步向链中添加块，然后通过发出四个新的交易（创建一个新的大理石，然后是三个大理石转移）来观察`price`信息被清除，这将为链添加四个新块。在第四次交易（第三次大理石转移）之后，将验证`price`私人数据是否被清除。

+ 使用以下命令切换回`Org1`中的`peer0`。复制并粘贴以下代码块并在对等容器中运行它：

  ```sh
  export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
  export CORE_PEER_LOCALMSPID=Org1MSP
  export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
  export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
  export PEER0_ORG1_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
  ```

+ 打开一个新的终端窗口，并通过运行以下命令查看该对等方的私有数据日志：

  ```sh
  $ docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
  ```

+ 应该看到类似于以下的结果。请注意列表中的**最高块编号**。在下面的示例中，最高块高度为 **4**。

  ```verilog
  [pvtdatastorage] func1 -> INFO 023 Purger started: Purging expired private data till block number [0]
  [pvtdatastorage] func1 -> INFO 024 Purger finished
  [kvledger] CommitWithPvtData -> INFO 022 Channel [mychannel]: Committed block [0] with 1 transaction(s)
  [kvledger] CommitWithPvtData -> INFO 02e Channel [mychannel]: Committed block [1] with 1 transaction(s)
  [kvledger] CommitWithPvtData -> INFO 030 Channel [mychannel]: Committed block [2] with 1 transaction(s)
  [kvledger] CommitWithPvtData -> INFO 036 Channel [mychannel]: Committed block [3] with 1 transaction(s)
  [kvledger] CommitWithPvtData -> INFO 03e Channel [mychannel]: Committed block [4] with 1 transaction(s)
  ```

+ 返回对等容器，通过运行以下命令查询`marble1`价格数据。（**查询不会在分类帐上创建新事务，因为没有数据处理**）。

  ```sh
  $ peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'
  ```

  应该看到类似的结果：

  ```sh
  {"docType":"marblePrivateDetails","name":"marble1","price":99}
  ```

  会发现**价格数据仍在私人数据分类帐中**。

## 添加新区块

+ 通过执行以下命令创建一个新的`marble2`，此事务在链上创建一个新块。

  ```sh
  $ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["initMarble","marble2","blue","35","tom","99"]}'
  ```

+ 切换回终端窗口，再次查看该对等体的私有数据日志。应该看到**块高度增加`1`**。

  ```sh
  $ docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
  ```

+ 返回对等容器，通过运行以下命令再次查询`marble1`价格数据：

  ```sh
  $ peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'
  ```

+ 私有数据尚未清除，因此结果与先前的查询相同：

  ```sh
  {"docType":"marblePrivateDetails","name":"marble1","price":99}
  ```

## 添加第二个区块

+ 通过运行以下命令将`marble2`传输到“`joe`”。此事务将在链上添加第二个新块。

  ```sh
  $ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["transferMarble","marble2","joe"]}'
  ```

+ 切换回终端窗口，再次查看该对等体的私有数据日志。您应该看到**块高度增加`1`**。

  ```sh
  $ docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
  ```

+ 返回对等容器，通过运行以下命令查询`marble1`价格数据：

  ```sh
  $ peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'
  ```

  仍然可以看到价格私人数据：

  ```sh
  {"docType":"marblePrivateDetails","name":"marble1","price":99}
  ```

## 添加第三个区块

+ 通过运行以下命令将`marble2`传输到“`tom`”。此事务将在链上创建第三个新块。

  ```sh
  peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["transferMarble","marble2","tom"]}'
  ```

+ 切换回终端窗口，再次查看该对等体的私有数据日志。应该看到块高度增加1。

  ```sh
  docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
  ```

+ 返回对等容器，通过运行以下命令查询`marble1`价格数据：

  ```sh
  $ peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'
  ```

+ 仍然可以看到价格数据。

  ```sh
  {"docType":"marblePrivateDetails","name":"marble1","price":99}
  ```

## 区块被清除

+ 最后，通过运行以下命令将`marble2`转移到“`jerry`”。此事务将在链上创建第四个新块。此交易后应清除私人数据的价格。

  ```sh
  $ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["transferMarble","marble2","jerry"]}'
  ```

+ 切换回终端窗口，再次查看该对等体的私有数据日志。您应该看到块高度增加`1`。

  ```sh
  $ docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
  ```

+ 返回对等容器，通过运行以下命令查询`marble1`价格数据：

  ```sh
  $ peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'
  ```

+ 由于**价格数据已被清除**，您应该再也看不到它了。你应该看到类似的东西：

  ```sh
  Error: endorsement failure during query. response: status:500
  message:"{\"Error\":\"Marble private details does not exist: marble1\"}"
  ```

# 使用带索引的私有数据

通过在`META-INF/statedb/couchdb/collections/<collection_name>/indexes`目录中打包索引以及链代码，索引也可以应用于私有数据集合。这里有一个[示例索引](https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02_private/go/META-INF/statedb/couchdb/collections/collectionMarbles/indexes/indexOwner.json)。

为了将链码部署到生产环境，建议在链码旁边定义任何索引，以便链码和支持索引作为一个单元自动部署，一旦链代码安装在对等端并在通道上实例化。**当指定`--collections-config`标志指向集合`JSON`文件的位置时，关联索引将在通道上的链代码实例化时自动部署**。

