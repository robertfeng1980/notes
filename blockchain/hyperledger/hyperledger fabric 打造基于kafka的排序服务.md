# `fabric` 打造基于`kafka`的排序服务

假设读者通常知道如何设置`Kafka`集群和`ZooKeeper`集合。本指南的目的是确定需要采取的步骤，以便让一组`Hyperledger Fabric`订购服务节点（`OSN`）使用`Kafka`集群并为区块链网络提供订购服务。

# 示意图

每个**通道映射到`Kafka`中的单独的单分区**主题。当`OSN`通过**广播`RPC`接收事务**时，它检查以**确保广播客户端具有在信道上写入的权限**，然后将这些事务中继（即产生）到`Kafka`中的适当分区。`OSN`也会使用此分区，它将**接收到的事务分组到本地块**中，将它们保存在**本地分类帐**中，并**通过`Deliver RPC`为它们提供服务**。有关低级细节，请参阅描述[如何实现此设计的文档](https://docs.google.com/document/d/19JihmW-8blTzN99lAubOfseLUZqdrB6sBR0HsRgCAnY/edit) - 图8是上述过程的示意图。

