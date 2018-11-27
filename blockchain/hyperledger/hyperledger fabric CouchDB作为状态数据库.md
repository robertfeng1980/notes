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


