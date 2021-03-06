# `hyperledger fabric` 通道

`Hyperledger Fabric`通道是**两个或多个特定网络成员之间通信的私有“子网”**，用于进行**私人和机密交易**。通道由**成员（组织）**，每个成员的**锚点对等体**，共享**分类帐**，**链代码应用**程序和**订购服务**节点定义。**网络上的每个事务都在一个通道上执行**，每个通道都**必须经过身份验证**并**授权**在该通道上进行交易。**加入信道**的每个对等体具有由**成员服务提供者（`MSP`）给出的其自己的身份**，其向每个对等体**验证**其信道对等体和服务。

要创建新通道，客户端`SDK`会**调用配置系统链代码和引用属性**，例如**锚点对等点和成员（组织）**。此请求为通道分类帐**创建一个创建块**，该块**包含有关通道策略**，**成员和锚点对等方**的配置信息。将新成员添加到现有通道时，将与**新成员共享**此**创建块**或更新的**重新配置块**（如果适用）。

> **注意**：有关配置事务的属性和原型结构的更多详细信息，请参阅[通道配置（`configtx`）部分](https://hyperledger-fabric.readthedocs.io/en/latest/configtx.html)。

为通道上的每个成员**选择主要对等体确定哪个对等体代表该成员与订购服务通信**。如果**没有识别出领导者**，则可以使用**算法来识别领导者**。**共识服务对事务进行排序**，并在一个**块**中将它们**传递给每个主要对等体**，然后使用**`gossip`协议**将**块分发给其成员对等体和整个信道**。

虽然**任何一个锚点**对等体都**可以属于多个通道**，因此**可以维护多个分类账**，但是**没有分类账数据**可以从**一个通道传递到另一个通道**。通过信道，**分类账的分离**由配置链码，身份成员服务和`gossip`数据**传播协议**来定义和实现。数据的传播（包括有关交易，分类账状态和渠道成员资格的信息）仅限于在渠道上具有**可验证成员资格**的同行。通过**渠道隔离同行和分类帐数据**，允许需要**私人和机密**交易的网络成员在同一区块链网络上与**业务竞争者和其他受限制成员共存**。





