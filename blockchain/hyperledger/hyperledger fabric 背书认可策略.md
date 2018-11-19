# 背书认可策略

**每个链代码都有一个认可策略**，该策略指定**必须执行链代码并支持执行结果**的通道上的**对等节点集合**，**以便将事务视为有效**。这些认可政策定义了**组织（同行节点）必须“认可”（即批准）提案的执行**。

> **注意**：由键值对表示的**状态**与**区块链数据是分开的**。有关这方面的更多信息，请查看我们的[`Ledger`](https://hyperledger-fabric.readthedocs.io/en/latest/ledger/ledger.html)文档。

作为由**对等方执行的交易验证步骤的一部分**，每个验证对等方检查以确保该交易**包含适当数量的背书认可**，并且它们来自**预期来源**（这两者都在认可政策中指定）。还会**检查认可以确保它们有效**（即，它们是来自有效证书的**有效签名**）。

# 认可要求的两种方式

默认情况下，在**实例化或升级**时为通道的链码指定**认可策略**（即一个认可策略涵盖与链码关联的所有状态）。

但是，在某些情况下，特定状态（特定键值对）可能**需要具有不同的认可策略**。这种基于**状态的背书认可**允许默认的链代码级**认可策略被指定`key`的不同策略覆盖**。

为了说明可能使用这两种类型的认可政策的情况，请考虑交换汽车的渠道。汽车作为可以交易的资产的“创造”，也称为“发行”。换言之，将**代表它的键值对换成世界状态**，必须满足**链码级认可政策**。要了解如何设置链代码级别的认可政策，请查看以下部分。

如果汽车需要特定的认可政策，则可以在汽车创建时或之后进行定义。设置特定州的认可政策可能有必要或更可取的原因有很多。汽车可能具有历史重要性或价值，因此必须获得持牌评估师的认可。此外，汽车的所有者（如果他们是频道的成员）可能还希望确保他们的同伴在交易中签字。在这两种情况下，**特定资产都需要一个认可政策，该政策与该链代码相关的其他资产的默认认可政策不同**。

在后续部分中展示定义基于州的认可政策。但首先，看看如何设置链码级别的认可政策。

# 设置链码级别认可政策

可以在实例化时使用`SDK`（有关如何执行此操作的示例代码，[单击此处](https://github.com/hyperledger/fabric-sdk-node/blob/f8ffa90dc1b61a4a60a6fa25de760c647587b788/test/integration/e2e/e2eUtils.js#L178)）或使用`-P`开关后跟策略在对等`CLI`中指定`Chaincode`级别认可策略。

参考示例：

```sh
$ peer chaincode instantiate -C <channelid> -n mycc -P "AND('Org1.member', 'Org2.member')"
```

此命令使用**策略`AND（'Org1.member'，'Org2.member'）`部署链码`mycc（"my chaincode"）`，这将要求`Org1`和`Org2`的成员签署该事务**。

> **注意**：如果启用了身份分类（请参阅[成员服务提供商](https://hyperledger-fabric.readthedocs.io/en/latest/msp.html)），则可以使用`PEER`角色来限制**仅对等节点的认可**。

参考示例：

```sh
$ peer chaincode instantiate -C <channelid> -n mycc -P "AND('Org1.peer', 'Org2.peer')"
```

实例化后添加到通道的**新组织可以查询链码**（假设查询具有由通道策略定义的**适当授权**以及由链码强制执行的任何应用程序级别检查），但**无法执行或认可链代码**。需要**修改认可政策**，以**允许通过新组织的认可来进行交易**。

> **注意**：如果**未在实例化时指定认可策略**，则认可策略默认为“**通道中组织的任何成员**”。例如，具有“`Org1`”和“`Org2`”的频道将具有默认的**认可策略 `OR（'Org1.member'，'Org2.member'）`**。

## 认可策略语法

策略以主体表示（“`principals` 主体”是与角色匹配的身份）。`Principal`被描述为'`MSP.ROLE`'，其中`MSP`表示所需的`MSP ID`，`ROLE`表示四个可接受的角色之一：`member`，`admin`，`client`和`peer`。

**以下是有效主体的几个示例**：

- `'Org0.admin'`：`Org0 MSP`的任何管理员
- `'Org1.member'`：`Org1 MSP`的任何成员
- `'Org1.client'`：`Org1 MSP`的任何客户端
- `'Org1.peer'`：`Org1 MSP`的任何同行节点

**该语言的语法是**：

```sh
EXPR(E[, E...])
```

`EXPR`是`AND、OR、OutOf`，而`E`是主体（使用上述语法）或另一个嵌套调用`EXPR`。

例如：

+ `AND('Org1.member', 'Org2.member', 'Org3.member')` 请求三个主体成员中的每一个必须签名。
+ `OR('Org1.member', 'Org2.member')` 从两个成员中的任何一个请求一个签名。
+ `OR('Org1.member', AND('Org2.member', 'Org3.member'))` 请求来自`Org1 MSP`成员的一个签名或来自`Org2 MSP`成员的一个签名和来自`Org3 MSP`成员的一个签名。
+ `OutOf(1, 'Org1.member', 'Org2.member')` 这解决了同样的事情 `OR('Org1.member', 'Org2.member')`
+ `OutOf(2, 'Org1.member', 'Org2.member')` 等同于 `AND('Org1.member', 'Org2.member')`
+ `OutOf(2, 'Org1.member', 'Org2.member', 'Org3.member')` 等同于 `OR(AND('Org1.member', 'Org2.member'), AND('Org1.member', 'Org3.member'), AND('Org2.member', 'Org3.member'))`

# 设置键级别的认可政策

设置常规链码级认可策略与相应**链码的生命周期相关联**。只能在**实例化**或**升级通道**上的相应链码时**设置或修改**它们。

相反，可以从链码中以**更精细**的方式设置和**修改键级别认可策略**。修改是常规事务的**读写集**的一部分。填充程序`API`提供以下功能来**设置和检索**常规关键的认可策略。

> **注意**：下面的`ep`代表“**认可政策**”，其可以通过使用上述相同的语法或通过使用下面描述的便利功能来表达。这两种方法都会生成可由基本填充程序`API`使用的认可策略的二进制版本。

```go
SetStateValidationParameter(key string, ep []byte) error
GetStateValidationParameter(key string) ([]byte, error)
```

对于属于集合中[私有数据](https://hyperledger-fabric.readthedocs.io/en/latest/private-data/private-data.html)的键，以下函数适用：

```go
SetPrivateDataValidationParameter(collection, key string, ep []byte) error
GetPrivateDataValidationParameter(collection, key string) ([]byte, error)
```

为了帮助设置认可策略并将其编组为**验证参数字节数组**，`shim`提供了便利功能，允许链码开发人员根据组织的`MSP`标识符处理认可策略：

```go
type KeyEndorsementPolicy interface {
    // Policy 将签注策略作为字节返回
    Policy() ([]byte, error)

    // AddOrgs 将指定的组织添加到所需的组织列表中认可
    AddOrgs(roleType RoleType, organizations ...string) error

    // DelOrgs 从现有的键级认可中删除指定的渠道组织KVS密钥的策略。如果没有任何组织，将返回错误。
    DelOrgs([]string) error

    // DelAllOrgs 从此KVS密钥中删除任何键级别的认可策略。
    DelAllOrgs() error

    // ListOrgs 返回支持更改所需的通道组织数组
    ListOrgs() ([]string, error)
}
```

例如，要**为键设置认可策略**，其中**需要两个特定组织来认可键更改**，请将两个组织`MSPIDs`传递给`AddOrgs()`，然后调用`Policy()`以构造可以传递给的背书策略字节数组 `SetStateValidationParameter()`。

# 验证

在提交时，**设置键的值与设置键的认可策略没有区别**，既**更新键的状态**又根据相同的**规则进行验证**。

| 验证               | 没有验证参数集       | 验证参数集         |
| ------------------ | -------------------- | ------------------ |
| 修改值             | 检查链码背书认可策略 | 检查键级别认可策略 |
| 修改键级别认可策略 | 检查链码背书认可策略 | 检查键级别认可策略 |

如上所述，如果**修改键并且不存在键级认可策略**，则默认情况下应用**链代码级认可策略**。当**首次**为键设置键级认可策略时也是如此，必须首先**根据预先存在的链码级认可策略认可新的键级认可策略**。

如果**修改了键**并且**存在键级认可策略**，则密钥级认可策略将**覆盖**链代码级认可策略。实际上，这意味着密钥级认可策略可以比链代码级认可策略**更少限制或更严格**。由于**必须满足链码级认可策略才能首次设置密钥级认可策略，因此未违反任何信任假设**。

如果**删除密钥的认可策略**（设置为`nil`），则**链代码级认可策略将再次成为默认策略**。

如果事务使用不同的**关联密钥级认可策略修改多个密钥**，则需要**满足所有**这些策略才能使事务有效。

