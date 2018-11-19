# `hyperledger fabric` 可插拔交易认可和验证

在提交时验证事务时，对等体在应用事务本身随附的**状态更改之前执行各种检查**：

+ **验证**签署事务的**身份**。
+ 在交易中**验证参与者的签名**。
+ 确保事务满足相应链代码的命名空间的**认可策略**。

有些用例要求自定义事务验证规则与默认的`Fabric`验证**规则不同**，例如：

+ **`UTXO`（未使用的交易输出）**：当验证考虑到交易**是否不会花费**其投入。
+ **`Anonymous transactions` 匿名交易**：当认可**不包含对等体的身份**时，但共享的签名和公钥不能链接到对等体的身份。

# 可插拔的认可和验证逻辑

`Fabric`允许将**定制认可和验证逻辑实现和部署到对等体中**，以可插拔的方式与链码处理相关联。这个逻辑可以**编译到对等体中**，**内置于可选逻辑中**，也可以**作为[`Golang`插件](https://golang.org/pkg/plugin/)与对等体一起编译和部署**。

在链代码**实例化**时，每个链代码都与其**自己的认可和验证逻辑相关联**。如果用户未选择一个，则**隐式选择默认的内置逻辑**。对等管理员可以通过在对等启动时**加载和应用的认可/验证逻辑**的定制来**改变**通过扩展对等体的**本地配置**而选择的认可/验证逻辑。

# 配置

每个对等体都有一个本地配置（`core.yaml`），它**声明了认可/验证逻辑名称和要运行的实现之间的映射**。

**默认逻辑**称为`ESCC`（`E`代表认可）和`VSCC`（`V`代表验证），它们可以在处理程序部分的**对等本地配置**中找到：

```yaml
handlers:
    endorsers:
      escc:
        name: DefaultEndorsement
    validators:
      vscc:
        name: DefaultValidation
```

当**认可或验证**实现被编译到**对等体**中时，`name`属性表示要**运行的初始化函数**，以便获得**创建认可/验证逻辑实例的工厂**。

该函数是`core/handlers/library/library.go`下的`HandlerLibrary`构造的实例方法，为了添加自定义认可或验证逻辑，需要使用任何其他方法**扩展此构造**。由于这很麻烦并且构成了部署挑战，因此可以通过在名为`library`的名称下**添加另一个属性**来将**自定义认可和验证部署为`Golang`插件**。

例如，如果有作为**插件实现的自定义认可和验证逻辑**，将在`core.yaml`的配置中包含以下条目：

```yaml
handlers:
    endorsers:
      escc:
        name: DefaultEndorsement
      custom:
        name: customEndorsement
        library: /etc/hyperledger/fabric/plugins/customEndorsement.so
    validators:
      vscc:
        name: DefaultValidation
      custom:
        name: customValidation
        library: /etc/hyperledger/fabric/plugins/customValidation.so
```

**必须将`.so`插件文件放在对等的本地文件系统中**。

> **注意**：此后，自定义认可或验证逻辑实现将被称为“插件”，即使它们被编译到对等体中。

# 认可插件实现

要实现认可插件，必须实现`core/handlers/endorsement/api/endorsement.go`中的`Plugin`接口：

```go
// 插件赞同提案回复
type Plugin interface {
    // Endorse签署给定的有效负载（ProposalResponsePayload字节），并可选择改变它。
    // Returns:
    // 认可：有效载荷上的签名，以及用于验证签名的标识
    // 作为输入提供的有效负载（可在此功能中修改）
    // 或者失败时出错
    Endorse(payload []byte, sp *peer.SignedProposal) (*peer.Endorsement, []byte, error)

    // Init将依赖项注入插件的实例
    Init(dependencies ...Dependency) error
}
```

通过让对等方在`PluginFactory`接口中**调用`New`方法**，为**每个通道**创建给定插件类型的**认可插件实例**（通过方法名称标识为`HandlerLibrary`的实例方法或插件`.so`文件路径）。由插件开发人员实现：

```go
// PluginFactory 创建插件的新实例
type PluginFactory interface {
    New() Plugin
}
```

期望`Init`方法接收在`core/handlers/endorsement/api/`下声明的**所有依赖项作为输入**，标识为嵌入`Dependency`接口。在创建`Plugin`实例之后，**在对等体上调用`Init`方法，并将依赖关系`dependencies` 作为参数传递**。

目前，`Fabric`为代言插件提供了以下依赖项：

+ `SigningIdentityFetcher`：根据**给定的签名提议**返回`SigningIdentity`的实例

  ```go
  // SigningIdentity 签署消息并将其公共标识序列化为字节
  type SigningIdentity interface {
      // Serialize返回此标识的字节表示形式，用于验证此SigningIdentity签名的消息
      Serialize() ([]byte, error)
  
      // Sign 签署给定的有效负载并返回签名
      Sign([]byte) ([]byte, error)
  }
  ```

+ `StateFetcher`：**获取与世界状态**交互的`State`对象

  ```go
  // State 定义与世界状态的互动
  type State interface {
      // GetPrivateDataMultipleKeys 在单个调用中获取多个私有数据项的值
      GetPrivateDataMultipleKeys(namespace, collection string, keys []string) ([][]byte, error)
  
      // GetStateMultipleKeys 在一次调用中获取多个键的值
      GetStateMultipleKeys(namespace string, keys []string) ([][]byte, error)
  
      // GetTransientByTXID 获取与给定txID关联的值私有数据
      GetTransientByTXID(txID string) ([]*rwset.TxPvtReadWriteSet, error)
  
      // Done 释放世界状态占用的资源
      Done()
   }
  ```

# 验证插件实现

要实现验证插件，必须实现`core/handlers/validation/api/validation.go`中的`Plugin`接口：

```go
// Plugin 验证交易
type Plugin interface {
    // Validate 如果在事务内给定位置的操作，则返回nil在给定块中的给定位置有效，否则出错。
    Validate(block *common.Block, namespace string, txPosition int, actionPosition int, contextData ...ContextDatum) error

    // Init 将依赖项注入插件的实例
    Init(dependencies ...Dependency) error
}
```

每个`ContextDatum`都是由**对等方传递给验证插件**的其他运行时**派生元数据**。目前，唯一传递的`ContextDatum`是代表链码代言政策的代码：

```go
 // SerializedPolicy 定义序列化策略
type SerializedPolicy interface {
      validation.ContextDatum

      // Bytes 返回SerializedPolicy的字节
      Bytes() []byte
 }
```

通过让对等方在`PluginFactory`接口中调用`New`方法，为**每个通道创建给定插件类型的验证插件实例**（通过方法名称识别为`HandlerLibrary`的实例方法或插件`.so`文件路径）。将由插件开发人员实现：

```go
// PluginFactory 创建插件的新实例
type PluginFactory interface {
    New() Plugin
}
```

`Init`方法应该接收在`core/handlers/validation/api/`下声明的**所有依赖项作为输入**，标识为嵌入`Dependency`接口。在**创建`Plugin`实例**之后，**对等体在其上调用`Init`方法**，并将依赖关系作为参数传递。

目前，`Fabric`为验证插件提供了以下依赖项：

+ `IdentityDeserializer`：将**身份的字节**表示**转换**为可用于验证由其签名的**签名的`Identity`对象**，根据其对应的`MSP`进行**验证**，并查看它们是否**满足给定的`MSP Principal`**。完整规范可以在`core/handlers/validation/api/identities/identities.go`找到。

+ `PolicyEvaluator`：**评估**是否满足给定策略

  ```go
  // PolicyEvaluator 评估政策
  type PolicyEvaluator interface {
      validation.Dependency
  
      // Evaluate 获取一组SignedData并评估这组签名是否满足具有给定字节的策略
      Evaluate(policyBytes []byte, signatureSet []*common.SignedData) error
  }
  ```

+ `StateFetcher`：获取**与世界状态交互**的`State`对象

  ```go
  // State 定义与世界状态的互动
  type State interface {
      // GetStateMultipleKeys 在一次调用中获取多个键的值
      GetStateMultipleKeys(namespace string, keys []string) ([][]byte, error)
  
      // GetStateRangeScanIterator 返回一个迭代器，它包含给定键范围之间的所有键值。
      // startKey包含在结果中，并且排除了endKey。空startKey是指第一个可用密钥
      // 并且一个空的endKey引用最后一个可用的键。用于扫描所有键，包括startKey和endKey
      // 可以作为空字符串提供。但是，出于性能原因，应谨慎使用完整扫描。
      // 返回的ResultsIterator包含类型* KV的结果，该结果在protos/ledger/queryresult中定义。
      GetStateRangeScanIterator(namespace string, startKey string, endKey string) (ResultsIterator, error)
  
      // GetStateMetadata 返回给定命名空间和键的元数据
      GetStateMetadata(namespace, key string) (map[string][]byte, error)
  
      // GetPrivateDataMetadata 获取由元组<namespace，collection，key>标识的私有数据项的元数据
      GetPrivateDataMetadata(namespace, collection, key string) (map[string][]byte, error)
  
      // Done 释放状态占用的资源
      Done()
  }
  ```

# 重要笔记

+ **验证同行之间的插件一致性**：在未来的版本中，`Fabric`通道基础设施将保证在任何给定的区块链高度，通道中的所有对等体对给定的**链代码使用相同的验证逻辑**，以**消除**可能导致状态差异的**错误配置的可能性**。**意外运行不同实现的同行**。但是，目前系统操作员和管理员有责任确保不会发生这种情况。
+ **验证插件错误处理**：每当验证插件**无法确定给定事务是否有效时**，由于某些**瞬态执行问题**（如无法访问数据库），它应返回在`core/handlers/validation/api/validation.go`中定义的`ExecutionFailureError`类型的错误`validation.go`。返回的**任何其他错误**都被**视为认可策略错误**，并将事务标记为**验证逻辑无效**。但是，如果返回`ExecutionFailureError`，则**链处理将暂停**，而**不是将事务标记为无效**。这是为了防止不同同伴之间的**状态分歧**。
+ **私有元数据检索的错误处理**：如果插件通过使用`StateFetcher`接口**检索私有数据的元数据**，则按如下方式处理错误非常重要：`CollConfigNotDefinedError`和`InvalidCollNameError`表示指定的**集合不存在**，应该作为**确定性错误**处理，不应该导致插件返回`ExecutionFailureError`。
+ **将`Fabric`代码导入插件**：作为插件的一部分导入属于`Fabric`以外的`Fabric`的代码是非常不鼓励的，并且可能在`Fabric`代码在发行版之间发生更改时导致问题，或者在运行混合对等版本时**可能导致不可操作性问题**。理想情况下，插件代码应该只**使用给定的依赖项**，并且应该导入除`protobufs`之外的最小值。

