# `hyperledger fabric` 第一个示例

首先，如果还不熟悉`Fabric`网络的基本架构，则可能需要在继续之前访问 [简介](https://hyperledger-fabric.readthedocs.io/en/latest/blockchain.html) 和 [构建您的第一个网络](https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html) 文档。

其次，在文将介绍一些示例程序，**以了解`Fabric`应用程序的工作原理**。这些应用程序（以及他们使用的智能合约）统称为 `fabcar`提供了`Fabric`功能的广泛演示。值得注意的是，我们将展示**与证书颁发机构进行交互并生成注册证书**的过程，之后我们将**利用这些身份来查询和更新分类帐**。

将通过三个主要步骤，完成示例演示：

**1、建立开发环境。** 应用程序**需要一个网络**进行交互，因此需要下载一个**仅限于注册/认证，查询和更新所需的组件**：

![_images / AppConceptsOverview.png](https://hyperledger-fabric.readthedocs.io/en/latest/_images/AppConceptsOverview.png)

**2、学习应用程序将使用的示例中智能合约的参数。**智能合约包含各种功能，能够以不同的方式**与分类帐**进行交互。将**进入并检查该智能合约**，以了解应用程序将使用的功能。

**3、开发应用程序以便能够在分类帐上查询和更新资产。** 将进入应用程序代码本身（应用程序已用`Javascript`编写）并**手动操作变量**以运行不同类型的**查询和更新**。

完成本教程后，应该基本了解**应用程序如何与智能合约一起编程**，以便与`Fabric`网络上的分类帐（即对等方）进行交互。

# 建立开发环境

如果已经完成了[构建第一个网络](hyperledger%20fabric%20建立你的第一个网络.md)，那么应该设置开发环境并下载`fabric-samples`以及附带的配置文件。要运行本教程，现在需要做的就是**拆除（停止）现有的任何网络**，可以通过发出以下命令来执行此操作：

```sh
$ ./byfn.sh down
```

如果没有开发环境以及网络和应用程序的附带配置工件，请访问[环境条件](hyperledger%20fabric%20入门.md)页面，并确保在计算机上安装了必要的依赖项。

接下来，如果尚未执行此操作，请访问[安装示例，二进制文件和Docker镜像](hyperledger%20fabric%20入门.md)页面，然后按照提供的说明进行操作。克隆`fabric-samples`存储库后，返回本教程 ，并**下载最新的稳定`Fabric`映像和可用应用程序**。

所有准备完成后，**导航到`fabric-samples`存储库中的`fabcar`子目录**，并查看内部的内容：

```sh
$ cd fabric-samples/fabcar  && ls

enrollAdmin.js  invoke.js  package.json  query.js  registerUser.js  startFabric.sh
```

在开始之前，我们还需要做一些**清理工作**。运行以下命令以**终止任何陈旧或活动容器**：

```sh
$ docker rm -f $(docker ps -aq)
```

**清除所有缓存的网络**：

```sh
# Press 'y' when prompted by the command
$ docker network prune
```

最后，如果已经完成了本教程，那么还需要**删除`fabcar`智能合约的基础链码图像**。如果您是第一次浏览此内容的用户，那么您的系统上将不会显示此链代码图像：

```sh
$ docker rmi dev-peer0.org1.example.com-fabcar-1.0-5c906e402ed29f20260ae42283216aa75549c571e2e380f3615826365d8269ba

# or
$ docker rmi $(docker images dev-* -aq)
```

## 安装客户端并启动网络

