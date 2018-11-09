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

作为额外的数据隐私优势，由于正在使用集合，因此只有**私有数据哈希值**通过`orderer`，而不是私有数据本身，从而**使私有数据对`orderer`保密**。

