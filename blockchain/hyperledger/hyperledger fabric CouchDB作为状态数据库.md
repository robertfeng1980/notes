# `CouchDB` 作为状态数据库

状态数据库选项包括`LevelDB`和`CouchDB`。`LevelDB`是**嵌入在对等进程中**的**默认键值状态数据库**。`CouchDB`是**可选的替代外部状态数据库**。与`LevelDB`键值存储一样，`CouchDB`可以存储以**链代码建模的任何二进制数据**（`CouchDB`附件功能在内部用于**非`JSON`二进制数据**）。但作为`JSON`**文档存储**，当链代码值（例如资产）被建模为`JSON`数据时，`CouchDB`还能够**对链代码数据进行丰富的查询**。

`LevelDB`和`CouchDB`都**支持核心链代码操作**，例如**获取和设置**键（资产）以及基于键查询。可以**按范围查询**键值，并且可以对**复合键值建模**以启用针对多**个参数的等效查询**。例如，**所有者的复合键**，`asset_id`可用于查询某个实体拥有的所有资产。这些基于键的查询可用于针对**分类帐的只读查询**，以及更新分类帐的事务。

如果将资产**建模为`JSON`并使用`CouchDB`**，还可以使用链代码中的`CouchDB` `JSON`查询语言对链代码数据值执行**复杂的富查询**。这些类型的查询非常适合理解分类帐上的内容。针对这些类型的查询的提议响应通常对客户端应用程序有用，但通常不作为事务提交给订购服务。实际上，**无法保证**结果集在链代码执行和富查询的提交时间之间是**稳定**的，因此**丰富的查询不适合在更新事务中使用**，除非应用程序**可以保证结果集在链代码执行时和提交时间**，或者**可以处理后续事务中的潜在更改**。例如，如果对`Alice`拥有的所有资产执行富查询并将其传输给`Bob`，则可能会在链代码执行时间和提交时间之间通过另一个事务将新资产分配给`Alice`，**将错过此“幻像”项**。

`CouchDB`作为一个**独立的数据库进程与对等体一起运行**，因此在设置，管理和操作方面还有其他考虑因素。可以考虑从默认的嵌入式`LevelDB`开始，如果需要其他**复杂的富查询**，请转到`CouchDB`。将链码资产数据**建模为`JSON`是一种很好的做法**，因此可以选择**在将来需要时执行复杂的富查询**。

> **注意**：`CouchDB JSON`文档的关键不能以下划线（“_”）开头。此外，`JSON`文档不能在顶级使用以下字段名称。这些**保留供内部使用**。
>
> - `任何以下划线开头的字段“_”`
> - `~version`

# 使用`Chaincode`的`CouchDB`

## `Chaincode`查询

大多数链码`shim`程序`API`可以与`LevelDB`或`CouchDB`状态数据库一起使用，例如，`GetState`，`PutState`，`GetStateByRange`，`GetStateByPartialCompositeKey`。此外，当您将`CouchDB`用作状态数据库并将模型资产用作链码中的`JSON`时，可以使用`GetQueryResult` `API`并传递`CouchDB`查询字符串，对状态数据库中的`JSON`执行丰富查询。查询字符串[遵循`CouchDB JSON`查询语法](http://docs.couchdb.org/en/2.1.1/api/database/find.html)。

`marbles02`结构样本演示了如何使用来自链代码的`CouchDB`查询。它包含一个`queryMarblesByOwner()`函数，它通过将所有者`ID`传递给`chaincode`来演示参数化查询。然后，它使用`JSON`查询语法查询状态数据以查找与`marble`的`docType`和所有者`id`匹配的`JSON`文档：

```json
{"selector":{"docType":"marble","owner":<OWNER_ID>}}
```

## `CouchDB`分页

`Fabric`支持对**丰富查询和基于范围的查询**的查询结果**进行分页**。支持分页的`API`允许使用**页面大小**和**书签**来用于范围和富查询。

如果使用分页查询`API`（`GetStateByRangeWithPagination()`，`GetStateByPartialCompositeKeyWithPagination()`和`GetQueryResultWithPagination()`）指定了`pagesize`，则将返回一组结果以及书签。**书签可以与后续查询一起使用以接收结果的下一个“页面”**。

所有链代码查询都由`core.yaml`的`totalQueryLimit`（默认值`100000`）绑定。这是`chaincode`将迭代并返回到客户端的**最大结果数**，以**避免意外或恶意长时间运行的查询**。

使用分页的示例包含在[`Using CouchDB`教程](https://hyperledger-fabric.readthedocs.io/en/latest/couchdb_tutorial.html)中。

> **注意**：无论链代码是否使用分页查询，对等方都将基于`core.yaml`中的`internalQueryLimit`（默认值`1000`）批量查询`CouchDB`。此行为可确保在**对等方和`CouchDB`之间传递合理大小的结果集**，并且对链代码是**透明的，无需其他配置**。

## `CouchDB`索引

`CouchDB`中的**索引是必需的**，以便**使`JSON`查询高效**，并且对于任何带有**排序的`JSON`查询都是必需**的。**索引可以与`/META-INF/statedb/couchdb/indexes`目录中的`chaincode`一起打包**。每个索引必须在其**自己的文本文件中定义，扩展名为`* .json`，索引定义采用`JSON`格式，[遵循`CouchDB`索引`JSON`语法](http://docs.couchdb.org/en/2.1.1/api/database/find.html#db-index)**。例如，要支持上述大理查询，请提供`docType`和`owner`字段的示例索引：

```json
{"index":{"fields":["docType","owner"]},"ddoc":"indexOwnerDoc", "name":"indexOwner","type":"json"}
```

示例索引可以在[这里找到](https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02/go/META-INF/statedb/couchdb/indexes/indexOwner.json)。

链代码的`META-INF/statedb/couchdb/indexes`目录中的**任何索引**都将与用于**部署的链代码打包**在一起。当链代码**安装**在对等体上并在对等体的一个通道上**实例化**时，**索引将自动部署**到对等体的**通道**和链代码特定的**状态数据库**（如果它已配置为使用`CouchDB`）。如果**首先安装链代码然后在通道上实例化链代码**，则索引将在**链代码实例化时进行部署**。如果**链接代码已在通道上实例化**，并且稍后在**对等方上安装链代码**，则**索引将在链代码安装时部署**。

部署后，**链代码查询将自动使用索引**。`CouchDB`可以根据查询中使用的**字段自动确定要使用的索引**。或者，在选择器查询中，可以**使用`use_index`关键字指定索引**。

安装的链代码的后续版本中**可能存在相同的索引**。要更改索引，请**使用相同的索引名称**，但更改索引定义。**在安装/实例化时，索引定义将重新部署到对等方的状态数据库**。

如果**已经拥有大量数据**，并且**稍后安装**了链代码，则**安装时创建索引**可能**需要一些时间**。同样，如果**已经拥有大量数据**并**实例化后续版本的链代码**，则**索引创建可能需要一些时间**。**避免在这些时间**调用**查询**状态数据库的链代码函数，因为在**索引初始化时链代码查询可能会超时**。在事务处理期间，**当块被提交到分类帐时，索引将自动刷新**。

# `CouchDB`配置

通过将状态数据库配置选项从`goleveldb`更改为`CouchDB`，可以**将`CouchDB`作为状态数据库启用**。此外，`couchDBAddress`需要**配置为指向对等方使用的`CouchDB`**。如果`CouchDB`配置了**用户名和密码**，则应使用管理员**用户名和密码**填充用户名和密码**属性**。`couchDBConfig`部分提供了其他选项，并记录在案。**对`core.yaml`的更改将在重新启动对等体后立即生效**。

还可以**传入`docker`环境变量来覆盖`core.yaml`值**，例如`CORE_LEDGER_STATE_STATEDATABASE`和`CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS`。

以下是`core.yaml`的`stateDatabase`部分：

```yaml
state:
  # stateDatabase - options are "goleveldb", "CouchDB"
  # goleveldb - default state database stored in goleveldb.
  # CouchDB - store state database in CouchDB
  stateDatabase: goleveldb
  # Limit on the number of records to return per query
  totalQueryLimit: 10000
  couchDBConfig:
     # It is recommended to run CouchDB on the same server as the peer, and
     # not map the CouchDB container port to a server port in docker-compose.
     # Otherwise proper security must be provided on the connection between
     # CouchDB client (on the peer) and server.
     couchDBAddress: couchdb:5984
     # This username must have read and write authority on CouchDB
     username:
     # The password is recommended to pass as an environment variable
     # during start up (e.g. LEDGER_COUCHDBCONFIG_PASSWORD).
     # If it is stored here, the file must be access control protected
     # to prevent unintended users from discovering the password.
     password:
     # CouchDB错误的重试次数
     maxRetries: 3
     # 对等启动期间CouchDB错误的重试次数
     maxRetriesOnStartup: 10
     # CouchDB请求超时（单位：持续时间，例如20秒）
     requestTimeout: 35s
     # 限制每个CouchDB查询的记录数
     # 请注意，链代码查询仅受totalQueryLimit的约束。
     # 内部链代码可以执行多个CouchDB查询，
     # 每个大小的internalQueryLimit。
     internalQueryLimit: 1000
     # 限制每个CouchDB批量更新批次的记录数
     maxBatchUpdateSize: 1000
     # 每N个块后的热指数。
     # 此选项可以加热已经存在的所有索引
     # 每N个块后部署到CouchDB。
     # 值为1将在每个块提交后加热索引，
     # 以确保快速选择器查询。
     # 增加该值可以提高peer和CouchDB的写入效率，
     # 但可能会降低查询响应时间。
     warmIndexesAfterNBlocks: 1
```

托管在`Hyperledger Fabric`提供的`docker`容器中的`CouchDB`能够使用`Docker Compose`脚本使用`COUCHDB_USER`和`COUCHDB_PASSWORD`环境变量传入环境变量来设置`CouchDB`用户名和密码。

对于`Fabric`提供的`docker`镜像之外的`CouchDB`安装，必须编辑该[安装的`local.ini`文件](http://docs.couchdb.org/en/2.1.1/config/intro.html#configuration-files)以**设置管理员用户名和密码**。

`Docker`撰写脚本**仅在创建容器时设置用户名和密码**。如果要在**创建容器后更改用户名或密码，则必须编辑`local.ini`文件**。

> **注意**：在每个对等启动时读取`CouchDB`对等选项。

