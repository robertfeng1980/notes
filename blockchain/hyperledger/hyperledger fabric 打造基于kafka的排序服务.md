# `fabric` 打造基于`kafka`的排序服务

假设读者通常知道如何设置`Kafka`集群和`ZooKeeper`集合。本指南的目的是确定需要采取的步骤，以便让一组`Hyperledger Fabric`订购服务节点（`OSN`）使用`Kafka`集群并为区块链网络提供订购服务。

# 示意图

每个**通道映射到`Kafka`中的单独的单分区**主题。当`OSN`通过**广播`RPC`接收事务**时，它检查以**确保广播客户端具有在信道上写入的权限**，然后将这些事务中继（即产生）到`Kafka`中的适当分区。`OSN`也会使用此分区，它将**接收到的事务分组到本地块**中，将它们保存在**本地分类帐**中，并**通过`Deliver RPC`为它们提供服务**。有关低级细节，请参阅描述[如何实现此设计的文档](https://docs.google.com/document/d/19JihmW-8blTzN99lAubOfseLUZqdrB6sBR0HsRgCAnY/edit) - 图8是上述过程的示意图。

# 步骤

下面使用`K`和`Z`分别为`Kafka`集群和`ZooKeeper`集合中的节点数：

1. **Kafka**：至少应将`K`设置为`4`（正如将在下面的步骤`4`中解释的那样，这是为了展示**崩溃容错**所需的**最小节点数**，即有`4`个代理，**可以让`1`个代理停止运行**，所有通道将继续**可写和可读**，并且可以**创建新通道**）

2. **Zookeeper**：`Z`将是`3,5`或`7`。它**必须是一个奇数**以避免裂脑情况，并且**大于1以避免单点故障**。**超过7台`ZooKeeper`服务器**的任何东西都被视为**过度杀伤**。

3. **Orderers**：在网络的**创世块中编码`Kafka`相关信息**。如果正在使用`configtxgen`，请编辑`configtx.yaml` 或选择系统通道的创建块的预设配置文件，以便：
   + `Orderer.OrdererType` 被设置为 `kafka`
   + `Orderer.Kafka.Brokers` 包含**集群中至少两个`Kafka`代理**的`IP`地址：`IP:por`。该清单不需要详尽无遗（这些是自助经纪人）

4. **Orderers**：**设置最大块大小**。每个块**最多只有`Orderer.AbsoluteMaxBytes`个字节**（不包括标题），这个值可以在`configtx.yaml`中设置。在此处选择的值为`A`并记下它，它将影响在步骤`6`中配置`Kafka`代理的方式。

5. **Orderers**：**创建创世块**，使用`configtxgen`。在上面的步骤`3`和`4`中选择的设置是**系统范围的设置**，即它们适用于所有`OSN`的网络，记下创世块的位置。

6. **Kafka cluster**：适当配置`Kafka`集群经纪人。确保每个`Kafka`代理都配置了以下键值：

   + `unclean.leader.election.enable = false` **数据一致性是区块链环境中的关键**。不能在**同步副本集之外选择频道领导者**，否则冒着**覆盖前一个领导者产生的偏移的风险**，并且**结果重写**了订货人产生的**区块**链。

   + `min.insync.replicas = M` **副本**选择值`M`，**使得`1 <M <N`**（参见下面的`default.replication.factor`）。当数据被**写入至少`M`个副本**（然后被认为是同步并且属于同步副本集或`ISR`）时，**认为数据被提交**。在任何其他情况下，**写操作都会返回错误**。

     + 如果通道数据被**写入的`N`的`N-M`个副本多达不可用**，则操作**正常进行**。
     + 如果**更多副本不可用**，`Kafka`**无法维护**`M`的`ISR`集，因此它会**停止接受写入**。`Reader`工作没有问题。**当`M`个副本同步时，该通道再次可写**。

   + `default.replication.factor = N` 复制因子**选择值`N`使得`N < K`**。复制因子为`N`，意味着**每个通道的数据**都将**复制到`N`个代理**。这些是渠道的`ISR`集合的**候选者**。正如在上面的`min.insync.replicas`部分中所提到的，**并非所有这些代理都必须始终可用**。`N`应该严格设置为**小于`K`**，因为如果**少于`N`个经纪人，则渠道创建不能继续进行**。因此，如果设置`N = K`，则**单个代理`down`**意味着**不能**在区块链网络上**创建新**的通道，**订购服务的崩溃容错不存在**。

     基于上面所描述的**`M`和`N`的最小允许值分别为`2`和`3`**。此配置允许继续**创建新的通道**，并**允许所有通道继续可写**。

   + `message.max.bytes` 和 `replica.fetch.max.bytes` 应**设置为大于`A`的值**，即在上面的步骤`4`中的`Orderer.AbsoluteMaxBytes`中选择的值。添加一些缓冲区来存储`header`  `1 MiB`绰绰有余。以下条件适用：

     ```sh
     Orderer.AbsoluteMaxBytes < replica.fetch.max.bytes <= message.max.bytes
     ```

     为了完整起见，**注意到`message.max.bytes`应该严格小于`socket.request.max.bytes`，默认设置为`100 MiB`**。如果希望块大于`100 MiB`，则需要在`fabric/orderer/kafka/config.go`中的`brokerConfig.Producer.MaxMessageBytes`中**编辑硬编码值**，并**从源码重建二进制文件**。这是不可取的。

   + `log.retention.ms = -1` 在订购服务添加对修剪`Kafka`日志的支持之前，应该**禁用基于时间的保留并防止段过期**。在撰写本文时，`Kafka`默认禁用基于大小的保留`log.retention.bytes`，因此**无需明确设置它**。

7. **Orderers**：将**每个`OSN`指向创世块**。在`orderer.yaml`中编辑`General.GenesisFile`，使其指向上面步骤`5`中创建的创世块（在此期间，确保正确设置该`YAML`文件中的所有其他键）

8. **Orderers**：调整**轮询间隔和超时** （可选步骤）

   + `orderer.yaml`文件中的`Kafka.Retry`部分允许调整**元数据/生产者/消费者**请求的**频率**以及套接字**超时**（这些是希望在`Kafka`制作人或消费者中看到的所有设置）

   + 此外，在**创建**新通道时，或者在**重新加载**现有通道时（重新启动的订购者），**订购者通过以下方式与`Kafka`群集交互**：

     + 它为与频道对应的`Kafka`分区创建了一个`Kafka`**生成器（`writer`）**。
     + 它使用该生产者向该分区发布**无操作`CONNECT`消息**。
     + 它为该分区创建了一个`Kafka`**使用者（`reader`）**。

     如果这些步骤中的**任何一个失败**，可以**调整**它们**重复的频率**。具体来说，将在每个`Kafka.Retry.ShortInterval`中重新尝试总共`Kafka.Retry.ShortTotal`，然后每个`Kafka.Retry.LongInterval`重新尝试总计`Kafka.Retry.LongTotal`直到成功。请注意，在成功完成上述所有步骤之前，**订购者将无法写入或读取通道**。

9. 设置`OSN`和`Kafka`集群，以便它们**通过`SSL`进行通信**。（可选步骤，但强烈建议）请参阅方程式`Kafka`群集端的[`Confluent`指南](https://docs.confluent.io/2.0.0/kafka/ssl.html)，并相应地在**每个`OSN`上的`orderer.yaml`中的`Kafka.TLS`下设置键值**。

10. 按以下顺序打开节点：`ZooKeeper`集合，`Kafka`集群，`Orderer`服务节点。

# 其他考虑因素

+ **首选消息大小**：在上面的步骤`4`中（请参阅步骤部分），还可以通过设置`Orderer.Batchsize.PreferredMaxBytes`键来**设置块的首选大小**。`Kafka`在处理相对较小的消息时**提供更高的吞吐量**，目标是**不超过`1 MiB`的值**。
+ **使用环境变量覆盖设置**：使用`Fabric`提供的示例`Kafka`和`Zookeeper Docker`镜像时（请分别参见`images/kafka`和`images/zookeeper`），可以使用**环境变量覆盖`Kafka`代理或`ZooKeeper`服务器的设置**。用**下划线替换配置键的点**，例如`KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE = false`将允许覆盖`unclean.leader.election.enable`的默认值。这同样适用于`OSN`的本地配置，即可以在`orderer.yaml`中设置的内容。例如，`ORDERER_KAFKA_RETRY_SHORTINTERVAL = 1s`允许覆盖`Orderer.Kafka.Retry.ShortInterval`的默认值。

