# `hyperledger fabric` 第一个示例

首先，如果还不熟悉`Fabric`网络的基本架构，则可能需要在继续之前访问 [简介](https://hyperledger-fabric.readthedocs.io/en/latest/blockchain.html) 和 [构建您的第一个网络](https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html) 文档。

其次，在文将介绍一些示例程序，**以了解`Fabric`应用程序的工作原理**。这些应用程序（以及他们使用的智能合约）统称为 `fabcar`提供了`Fabric`功能的广泛演示。值得注意的是，我们将展示**与证书颁发机构进行交互并生成注册证书**的过程，之后我们将**利用这些身份来查询和更新分类帐**。

将通过三个主要步骤，完成示例演示：

**1、建立开发环境。** 应用程序**需要一个网络**进行交互，因此需要下载一个**仅限于注册/认证，查询和更新所需的组件**：

![_images / AppConceptsOverview.png](https://hyperledger-fabric.readthedocs.io/en/latest/_images/AppConceptsOverview.png)

**2、学习应用程序将使用的示例中智能合约的参数。**智能合约包含各种功能，能够以不同的方式**与分类帐**进行交互。将**进入并检查该智能合约**，以了解应用程序将使用的功能。

**3、开发应用程序以便能够在分类帐上查询和更新资产。** 将进入应用程序代码本身（应用程序已用`Javascript`编写）并**手动操作变量**以运行不同类型的**查询和更新**。

完成本教程后，应该基本了解**应用程序如何与智能合约一起编程**，以便与`Fabric`网络上的分类帐（即对等方）进行交互。

