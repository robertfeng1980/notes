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

本文将使用在 **构建第一个网络（`BYFN`）**教程网络上运行的[弹珠私有数据示例](https://github.com/hyperledger/fabric-samples/tree/master/chaincode/marbles02_private)来演示如何创建、部署和使用私有数据集合。大理石私有数据样本将部署到[构建您的第一个网络](https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html)（`BYFN`）教程网络。应该已经完成了[安装样本，二进制文件和Docker镜像](https://hyperledger-fabric.readthedocs.io/en/latest/install.html)的任务; 但是，运行BYFN教程不是本教程的先决条件。相反，本教程中提供了必要的命令来使用网络。我们将描述每个步骤中发生的事情，使得在不实际运行示例的情况下理解教程成为可能。