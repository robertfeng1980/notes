# `hyperledger fabric` 中使用`CouchDB`

本教程将介绍将`CouchDB`用作`Hyperledger Fabric`的**状态数据库**所需的步骤。到目前为止，应该熟悉`Fabric`概念并探索了一些示例和教程。

本教程将指导完成以下步骤：

+ 在`Hyperledger Fabric`中启用`CouchDB`
+ 创建索引
+ 将索引添加到您的`chaincode`文件夹
+ 安装并实例化`Chaincode`
+ 查询`CouchDB`状态数据库
+ 使用分页查询`CouchDB`状态数据库
+ 更新索引
+ 删除索引

要深入了解`CouchDB`，请将[`CouchDB`称为状态数据库](https://hyperledger-fabric.readthedocs.io/en/latest/couchdb_as_state_database.html)，有关`Fabric`分类帐的更多信息，请参阅[`Ledger`主题](https://hyperledger-fabric.readthedocs.io/en/latest/ledger/ledger.html)。请按照以下教程获取有关如何在区块链网络中利用`CouchDB`的详细信息。

在本教程中，我们将使用Marbles示例作为我们的用例来演示如何将CouchDB与Fabric配合使用，并将Marbles部署到构建您的第一个网络（BYFN）教程网络。您应该已完成安装样本，二进制文件和Docker镜像的任务。但是，运行BYFN教程不是本教程的先决条件，而是在本教程中提供必要的命令以使用网络。

# 为什么选择`CouchDB`

`Fabric`支持**两种类型**的对等数据库。`LevelDB`是**嵌入**在对等节点中的**默认状态数据库**，它将链码数据存储为**简单的键值对**，仅支持**键、键范围和复合键查询**。`CouchDB`是一个**可选的备用**状态数据库，当链码数据值**建模为`JSON`时**，它**支持富查询**。当要查询实际数据值内容而不是键时，富查询对**大型索引**数据存储更**灵活，更有效**。`CouchDB`是**一个`JSON`文档**数据存储区而**不是纯键值存储**区，因此可以**索引数据库中文档**的内容。

为了利用`CouchDB`的优势，即基于内容的`JSON`查询，数据必须以`JSON`格式建模。在设置网络之前，必须决定是使用`LevelDB`还是`CouchDB`。由于数据兼容性问题，不支持将对等体从使用`LevelDB`切换到`CouchDB`。网络上的**所有**对等方都**必须使用相同**的数据库类型。如果混合使用`JSON`和二进制数据值，仍可以使用`CouchDB`，但只能根据键、键范围和组合键查询查询二进制值。

# 在`Hyperledger Fabric`中启用`CouchDB`

`CouchDB`作为一个**独立**的数据库进程与对等体一起运行，因此在设置，管理和操作方面还有其他考虑因素。可以使用`CouchDB`的`docker`镜像，**建议它在与同级服务器相同的服务器上运行**。需要**为每个对等节点**设置一个`CouchDB`容器，并通过**更改`core.yaml`中的配置**来更新每个对等容器，以**指向`CouchDB`容器**。**文件`core.yaml`必须位于环境变量`FABRIC_CFG_PATH`指定的目录中**：

+ 对于`docker`部署，`core.yaml`**已预先配置并位于对等容器`FABRIC_CFG_PATH`文件夹**中。但是，在使用`docker`环境时，通常通过编辑`docker-compose-couch.yaml`来**覆盖**`core.yaml`来传递**环境变量**。
+ 对于本机二进制部署，`core.yaml`包含在发布配置工件分发中。

编辑`core.yaml`的`stateDatabase`部分。将`CouchDB`指定为`stateDatabase`并填写关联的`couchDBConfig`属性。有关配置`CouchDB`以使用结构的更多详细信息，请[参阅此处](http://hyperledger-fabric.readthedocs.io/en/master/couchdb_as_state_database.html#couchdb-configuration)。要查看为`CouchDB`配置的`core.yaml`文件的示例，请检查`HyperLedger/fabric-samples/first-network`目录中的**BYFN** `docker-compose-couch.yaml`。

# 创建索引

为什么索引很重要？

**索引允许查询数据库，而不必检查每个查询的每一行，使它们运行得更快，更有效**。通常，索引是针对**频繁出现**的查询条件构建的，允许更**有效地查询数据**。要利用`CouchDB`的主要优势**能够针对`JSON`数据执行丰富查询**，不需要索引，但**强烈建议使用它们来提高性能**。此外，如果查询中需要排序，`CouchDB`需要**排序字段的索引**。

> **注意**：没有索引的富查询将起作用，但可能会在`CouchDB`日志中发出**未找到索引的警告**。但是，如果富查询**包含排序规范，则需要该字段的索引；否则，查询将失败并将引发错误**。

为了演示构建索引，将使用[`Marbles`示例](https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02/go/marbles_chaincode.go)中的数据。在此示例中，`Marbles`数据结构定义为：

```go
type marble struct {
    ObjectType string `json:"docType"` //docType is used to distinguish the various types of objects in state database
    Name       string `json:"name"`    //the field tags are needed to keep case from bouncing around
    Color      string `json:"color"`
    Size       int    `json:"size"`
    Owner      string `json:"owner"`
}
```

在此结构中，属性（`docType`, `name`, `color`, `size`, `owner`）定义与资产关联的分类帐数据。属性`docType`是链代码中使用的模式，用于**区分可能需要单独查询的不同数据类型**。使用`CouchDB`时，建议包含此`docType`属性以**区分`chaincode`命名空间中的每种类型的文档**。（每个链码都表示为自己的`CouchDB`数据库，也就是说，每个链码都有自己的`key`命名空间）

对于`Marbles`数据结构，**属性`docType`用于标识此文档/资产是`Marbles`资产。可能在链码数据库中可能存在其他文档/资产**。数据库中的文档可以**针对所有这些属性值**进行搜索。

在定义用于链码查询的**索引**时，**每个索引必须在其自己的文本文件中定义**，扩展名为`*.json`，索引定义必须以`CouchDB`索引`JSON`格式进行格式化。

**要定义索引，需要三条信息**：

+ `fields`：这些是经常查询的字段
+ `name`：索引的名称
+ `type`：在这种情况下总是`json`

例如，名为`foo`的字段的名为`foo-index`的简单索引。

```json
{
    "index": {
        "fields": ["foo"]
    },
    "name" : "foo-index",
    "type" : "json"
}
```

可选地，可以在**索引定义上指定设计文档属性`ddoc`**。设计文档是`CouchDB`构造，旨在包含索引。**索引可以分组到[设计文档](http://guide.couchdb.org/draft/design.html)中以提高效率**，**但`CouchDB`建议每个设计文档使用一个索引**。

> **提示**：定义索引时，**最好包括`ddoc`属性和值以及索引名称**。包含此属性非常重要，以确保可以在以后需要时**更新索引**。此外，它还能够**显式指定要在查询上使用的索引**。

以下是`Marbles`示例中索引定义的另一个示例，索引名称`indexOwner`使用多个字段`docType`和`owner`，并包含`ddoc`属性：

```json
{
  "index":{
      "fields":["docType","owner"] // Names of the fields to be queried
  },
  "ddoc":"indexOwnerDoc", // (optional) Name of the design document in which the index will be created.
  "name":"indexOwner",
  "type":"json"
}
```

在上面的示例中，如果设计文档`indexOwnerDoc`尚不存在，则会在**部署索引时自动创建它**。可以使用字段列表中**指定的一个或多个属性**构造索引，并且可以指定**任何属性组合**。对于同一`docType`，属性可以存在于**多个**索引中。在以下示例中，`index1`仅包含属性`owner`，`index2`包括属性`owner and color`，`index3`包括属性`owner, color and size`。另外，请注意每个索引定义都有自己的`ddoc`值，遵循`CouchDB`建议的做法。

```json
{
  "index":{
      "fields":["owner"] // Names of the fields to be queried
  },
  "ddoc":"index1Doc", // (optional) Name of the design document in which the index will be created.
  "name":"index1",
  "type":"json"
}

{
  "index":{
      "fields":["owner", "color"] // Names of the fields to be queried
  },
  "ddoc":"index2Doc", // (optional) Name of the design document in which the index will be created.
  "name":"index2",
  "type":"json"
}

{
  "index":{
      "fields":["owner", "color", "size"] // Names of the fields to be queried
  },
  "ddoc":"index3Doc", // (optional) Name of the design document in which the index will be created.
  "name":"index3",
  "type":"json"
}
```

通常，应该为**索引字段建模以匹配将在查询过滤器和排序中使用的字段**。有关以`JSON`格式构建索引的更多详细信息，请[参阅`CouchDB`文档](http://docs.couchdb.org/en/latest/api/database/find.html#db-index)。

关于索引的最后一句话，`Fabric`负责使用称为`index warming`的模式索引数据库中的文档。在下一个查询之前，`CouchDB`通常**不会索引新文档或更新的文档**。`Fabric`通过在**提交**每个数据块之后**请求索引更新**来确保索引保持“**warm**”。这可以**确保查询很快**，因为它们**不必查询之前索引**文档。**每次将新记录添加到状态数据库时，此过程都会使索引保持最新并刷新**。

# 将索引添加到`chaincode`文件夹

**完成索引后，可以将其与链码打包在一起，以便将其放在相应的元数据文件夹中**。

如果链码安装和实例化使用`Hyperledger Fabric Node SDK`，则`JSON`索引文件可以位于任何文件夹中，只要它符合此目录结构即可。在使用`client.installChaincode() API`进行链码安装期间，请在安装请求中包含属性（`metadataPath`）。`metadataPath`的值是一个字符串，表示**包含`JSON`索引文件的目录结构的绝对路径**。

或者，如果**使用`peer-commands`来安装和实例化链代码，那么`JSON`索引文件必须位于`META-INF/statedb/couchdb/indexes`路径下，该路径位于链代码所在的目录内**。

下面的[`Marbles`示例](https://github.com/hyperledger/fabric-samples/tree/master/chaincode/marbles02/go)说明了索引如何与将使用`peer`命令安装的`chaincode`打包在一起。

![Marbles Chaincode Index Package](https://hyperledger-fabric.readthedocs.io/en/latest/_images/couchdb_tutorial_pkg_example.png)

# 启动网络

在安装和实例化弹珠链码之前，需要启动`BYFN`网络。为了本教程的目的，希望从已知的初始状态进行操作。以下命令将终止所有活动或过时的`docker`容器并删除以前生成的工件。因此，**运行以下命令来清理以前的所有环境**：

```sh
$ cd fabric-samples/first-network
$ ./byfn.sh down
```

现在通过运行以下命令启动与`CouchDB`的`BYFN`网络：

```sh
$ ./byfn.sh up -c mychannel -s couchdb
```

这将创建一个简单的`Fabric`网络，其中包含一个名为`mychannel`的通道，其中包含两个组织（每个组织维护两个对等节点）和一个订购服务，同时使用`CouchDB`作为状态数据库。

# 安装并实例化`Chaincode`

**客户端应用程序通过链码与区块链分类帐交互**。因此，需要在**每个将执行和支持事务的对等体上**安装链代码，并在通道上实例化链代码。在上一节中，演示了如何打包链代码，以便它们可以为部署做好准备。

`Chaincode`安装在对等体上，然后使用`peer-commands`实例化到通道上。

+ `peer chaincode install` 命令用来在对等体上安装`Marbles`链码。假设已启动`BYFN`网络，请使用以下命令进入到`CLI`容器：

  ```sh
  $ docker exec -it cli bash
  ```

  使用以下命令将`marbles`链代码从`git`存储库安装到`BYFN`网络中的对等方。`CLI`容器默认使用`org1`的`peer0`：

  ```sh
  $ peer chaincode install -n marbles -v 1.0 -p github.com/chaincode/marbles02/go
  ```

+ 执行`peer chaincode instantiate`命令以实例化通道上的链代码。要在`BYFN`通道`mychannel`上实例化`Marbles`示例，请运行以下命令：

  ```sh
  export CHANNEL_NAME=mychannel
  peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org0MSP.peer','Org1MSP.peer')"
  ```

# 验证索引已部署

一旦链码安装在对等体上并在通道上实例化，**索引将被部署到每个对等体的`CouchDB`状态数据库**。可以通过检查`Docker`容器中的**对等节点日志**来验证是**否已成功创建`CouchDB`索引**。

要查看对等`docker`容器中的日志，请打开一个新的终端窗口并运行以下命令以`grep`查找已创建索引的消息。

```sh
$ docker logs peer0.org1.example.com  2>&1 | grep "CouchDB index"
```

应该看到如下所示的结果：

```sh
[couchdb] CreateIndex -> INFO 0be Created CouchDB index [indexOwner] in state database [mychannel_marbles] using design document [_design/indexOwnerDoc]
```

> **注意**：如果`BYFN`示例的 `peer`节点 `peer0.org1.example.com`上未安装`Marbles`，则可能需要将其替换为安装了`Marbles`的其他对等方的名称。

# 查询`CouchDB`状态数据库

现在**索引已在`JSON`文件中定义并与链码一起部署**，链码函数可以对`CouchDB`状态数据库**执行`JSON`查询**，因此对等命令可以调用链码函数。

在**查询上指定索引名称是可选的。如果未指定，并且已查询的字段已存在索引，则将自动使用现有索引**。

> **提示**：使用`use_index`关键字在查询中**显式包含索引名称**是一种很好的做法。没有它，`CouchDB`可能会**选择一个不太理想的索引**。此外，`CouchDB`可能**根本不使用索引**，可能没有意识到它，在测试期间的低容量。只有在较高的卷上，可能会发现性能较慢，因为`CouchDB`没有使用索引而认为它是。

## 在链码中构建查询

可以使用链码中的`CouchDB JSON`查询语言对链码数据值执行复杂的富查询。正如上面所探讨的，[`marbles02`示例](https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02/go/marbles_chaincode.go)链代码包含一个索引，并且在函数中定义了丰富的查询 `queryMarbles`和`queryMarblesByOwner`：

+ `queryMarbles`富查询的示例 `ad hoc`。这是一个查询，其中（选择器）字符串可以传递给函数。此查询对于需要在运行时**动态构建自己的选择器**的客户端应用程序非常有用。有关选择器的更多信息，请参阅[`CouchDB`选择器语法](http://docs.couchdb.org/en/latest/api/database/find.html#find-selectors)。
+ `queryMarblesByOwner` 参数化查询的示例，其中查询逻辑被拷贝到链代码中。在这种情况下，函数接受单个参数，即弹珠所有者。然后，它使用`JSON`查询语法在状态数据库中查询与`marble`的`docType`和所有者`id`匹配的`JSON`文档。

## 使用`peer`命令运行查询

如果没有客户端应用程序来测试链码中定义的富查询，则可以使用对等命令。对等命令从`docker`容器内的命令行运行。将自定义`peer chaincode query`命令以使用`Marbles`索引`indexOwner`并使用`queryMarbles`函数查询`tom`拥有的所有弹珠。

在查询数据库之前，应该添加一些数据。在对等容器中运行以下命令以创建由`tom`拥有的弹珠：

```sh
$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble1","blue","35","tom"]}'
```

在**链代码实例化期间部署索引之后，链代码查询将自动使用它**。`CouchDB`可以根据要**查询的字段**确定要使用的索引。如果查询**条件存在索引**，则将使用该索引。但是，**建议的方法是在查询中指定`use_index`关键字**。下面的`peer`命令是一个如何通过包含`use_index`关键字在选择器语法中显式指定索引的示例：

```sh
// Rich Query with index name explicitly specified:
$ peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarbles", "{\"selector\":{\"docType\":\"marble\",\"owner\":\"tom\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}"]}'
```

深入研究上面的查询命令，有三个重要的参数：

+ `queryMarbles`

  `Marbles`链码中函数的名称。注意`shim.ChaincodeStubInterface`用于**访问和修改**分类帐。`getQueryResultForQueryString()`将`queryString`传递给`shim API getQueryResult()`。

  ```go
  func (t *SimpleChaincode) queryMarbles(stub shim.ChaincodeStubInterface, args []string) pb.Response {
  
          //   0
          // "queryString"
           if len(args) < 1 {
                   return shim.Error("Incorrect number of arguments. Expecting 1")
           }
  
           queryString := args[0]
  
           queryResults, err := getQueryResultForQueryString(stub, queryString)
           if err != nil {
                 return shim.Error(err.Error())
           }
           return shim.Success(queryResults)
  }
  ```

+ `{"selector":{"docType":"marble","owner":"tom"}`

  这是一个`ad hoc`选择器字符串的示例，它**查找所有类型为`marble`的文档**，其中`owner`属性的值为`tom`。

+ `"use_index":["_design/indexOwnerDoc", "indexOwner"]`

  指定设计文档名称`indexOwnerDoc`和索引名称`indexOwner`。在此示例中，**选择器查询显式包含使用`use_index`关键字指定的索引名称**。回顾上面的索引定义[创建一个索引](https://hyperledger-fabric.readthedocs.io/en/latest/couchdb_tutorial.html#cdb-create-index)，它包含一个设计文档`"ddoc":"indexOwnerDoc"`。使用`CouchDB`时，如果计划在查询中显式包含索引名称，则**索引定义必须包含`ddoc`值**，因此可以使用`use_index`关键字引用它。

查询成功运行，并利用以下结果利用索引：

```sh
Query Result: [{"Key":"marble1", "Record":{"color":"blue","docType":"marble","name":"marble1","owner":"tom","size":35}}]
```

# 使用分页查询`CouchDB`状态数据库



# 更新索引




# 删除索引