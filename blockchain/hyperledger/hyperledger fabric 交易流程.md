# `hyperledger fabric` 交易流程

本文档概述了**标准资产交换过程中发生的交易机制**。该方案包括两个客户A和B，他们正在买卖萝卜。他们每个人都有一个网络上的同伴，**通过他们发送交易并与分类账进行交互**。

![_images/step0.png](https://hyperledger-fabric.readthedocs.io/en/latest/_images/step0.png)

## 假设

此流程假定**已建立并运行通道**。应用程序用户**已注册并认证了组织的证书颁发机构（CA）**，并收到了**必要的加密材料**，该材料用于向**网络进行身份验证**。

链代码（包含表示萝卜市场**初始状态的一组键值对**）安装在**对等体上并在通道上实例化**。链码包含定义**一组交易指令**的**逻辑**和萝卜的商定**价格**。还为此链代码设置了一个**认可政策**，声明`peerA`和`peerB`都必须**支持任何交易**。

![_images/step1.png](https://hyperledger-fabric.readthedocs.io/en/latest/_images/step1.png)

## 1、客户A启动交易

发生了什么？ **客户`A`正在发送购买萝卜的请求**。该请求的**目标是`peerA`和`peerB`**，它们分别代表**客户端A和客户端B**。认可策略**规定两个对等端**必须**支持任何事务**，因此**请求将转发给`peerA`和`peerB`**。

接下来，**构建交易提议**。利用受支持的`SDK`（`Node，Java，Python`）的**应用**程序，利用一个**可用的`API`**来生成**交易提议**。该提议是**调用链代码功能的请求**，以便可以将**数据读取或写入分类帐**（即为资产写入新的键值对）。`SDK`用作`shim`程序，将**事务提议打包为正确的架构格式**（`gRPC`上的协议缓冲区），并**使用用户的加密凭据**为此事务提议**生成唯一签名**。

![_images/step2.png](https://hyperledger-fabric.readthedocs.io/en/latest/_images/step2.png)



