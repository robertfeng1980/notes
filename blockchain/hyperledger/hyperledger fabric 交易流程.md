# `hyperledger fabric` 交易流程

本文档概述了**标准资产交换过程中发生的交易机制**。该方案包括两个客户A和B，他们正在买卖萝卜。他们每个人都有一个网络上的同伴，**通过他们发送交易并与分类账进行交互**。

![_images/step0.png](https://hyperledger-fabric.readthedocs.io/en/latest/_images/step0.png)

## 假设

此流程假定**已建立并运行通道**。应用程序用户**已注册并认证了组织的证书颁发机构（CA）**，并收到了**必要的加密材料**，该材料用于向**网络进行身份验证**。

链代码（包含表示萝卜市场**初始状态的一组键值对**）安装在**对等体上并在通道上实例化**。链码包含定义**一组交易指令**的**逻辑**和萝卜的商定**价格**。还为此链代码设置了一个**认可政策**，声明`peerA`和`peerB`都必须**支持任何交易**。

![_images/step1.png](https://hyperledger-fabric.readthedocs.io/en/latest/_images/step1.png)



