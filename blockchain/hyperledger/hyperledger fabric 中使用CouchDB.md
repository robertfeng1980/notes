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

编辑`core.yaml`的`stateDatabase`部分。将`CouchDB`指定为`stateDatabase`并填写关联的`couchDBConfig`属性。有关配置`CouchDB`以使用结构的更多详细信息，请参阅此处。要查看为`CouchDB`配置的`core.yaml`文件的示例，请检查`HyperLedger/fabric-samples/first-network`目录中的**BYFN** `docker-compose-couch.yaml`。

# 创建索引



# 将索引添加到您的`chaincode`文件夹



# 安装并实例化`Chaincode`



# 查询`CouchDB`状态数据库



# 使用分页查询`CouchDB`状态数据库



# 更新索引




# 删除索引