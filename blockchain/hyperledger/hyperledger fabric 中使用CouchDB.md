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



# 创建索引



# 将索引添加到您的`chaincode`文件夹



# 安装并实例化`Chaincode`



# 查询`CouchDB`状态数据库



# 使用分页查询`CouchDB`状态数据库



# 更新索引




# 删除索引