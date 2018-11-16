# `hyperledger fabric configtx` 通道配置

`Hyperledger Fabric`区块链网络的**共享配置存储在集合配置事务**中，每个通道一个。每个配置事务通常由较短的名称`configtx`引用。

**通道配置具有以下重要属性**：

+ **`Versioned` 版本化**：配置的所有元素都有一个**关联的版本**，每次都会提前修改。此外，每个提交的配置都接收序列号。
+ **`Permissioned` 有权限**：配置的每个元素都有一个**关联的策略**，该策略控制**是否允许修改该元素**。拥有先前`configtx`副本（并且没有其他信息）的**任何人都可以根据这些策略验证新配置的有效性**。
+ **`Hierarchical` 分级**：根配置组包含**子组、层次结构**的每个组都具有关联的值和策略。这些策略可以利用层次结构**从较低级别的策略中获取一个级别的策略**。

# 解剖配置

**配置作为`HeaderType_CONFIG`类型的事务存储在没有其他事务的块中**。这些块称为**配置块**，其中第一个称为**创世块**。

配置的`proto`结构存储在`fabric/protos/common/configtx.proto`中。`HeaderType_CONFIG`类型的`Envelope`将`ConfigEnvelope`消息编码为`Payload`数据`data`字段。`ConfigEnvelope`的原型定义如下：

```go
message ConfigEnvelope {
    Config config = 1;
    Envelope last_update = 2;
}
```

`last_update`字段在下面的**配置更新部分中定义**，但**仅在验证配置**而不是读取配置时才需要。相反，**当前已提交的配置**存储在配置 `config` 字段中，其中包含`Config`消息。

```go
message Config {
    uint64 sequence = 1;
    ConfigGroup channel_group = 2;
}
```

对于每个提交的配置，**序列号`sequence` 递增`1`**。`channel_group`字段是包含配置的**根组**。**递归**定义`ConfigGroup`结构，并**构建一个组树**，每个组包含**值和策略**。它的定义如下：

```go
message ConfigGroup {
    uint64 version = 1;
    map<string,ConfigGroup> groups = 2;
    map<string,ConfigValue> values = 3;
    map<string,ConfigPolicy> policies = 4;
    string mod_policy = 5;
}
```

因为`ConfigGroup`是**递归结构**，所以它具有**分层**排列。以下示例表示为`golang`表示法的清晰度。

```go
// Assume the following groups are defined
var root, child1, child2, grandChild1, grandChild2, grandChild3 *ConfigGroup

// Set the following values
root.Groups["child1"] = child1
root.Groups["child2"] = child2
child1.Groups["grandChild1"] = grandChild1
child2.Groups["grandChild2"] = grandChild2
child2.Groups["grandChild3"] = grandChild3

// The resulting config structure of groups looks like:
// root:
//     child1:
//         grandChild1
//     child2:
//         grandChild2
//         grandChild3
```

每个组在**配置层次结构**中定义一个级别，每个组都有一组**关联**的值（由**字符串键索引**）和**策略**（也由字符串键索引）。

值定义如下：

```go
message ConfigValue {
    uint64 version = 1;
    bytes value = 2;
    string mod_policy = 3;
}
```

策略定义如下：

```go
message ConfigPolicy {
    uint64 version = 1;
    Policy policy = 2;
    string mod_policy = 3;
}
```

请注意，**值、策略和组**都具有`version` 和`mod_policy`。每次**修改元素时，元素的版本都会递增**。`mod_policy`用于**管理所需的签名以修改该元素**。**对于组**，修改是在**值、策略或组映射中添加或删除元素**（或更改`mod_policy`）。**对于值和策略**，修改分别更改“**值**”和“**策略**”字段（或更改`mod_policy`）。每个元素的`mod_policy`都在`config`的当前级别的**上下文**中进行评估。请考虑在`Channel.Groups["Application"]`中定义的以下示例`mod`策略（这里，使用`golang`映射引用语法，因此`Channel.Groups["Application"].Policies["policy1"]`引用基本`Channel`组的`Application group`的`Policies`映射的`policy1`策略）

- `policy1` 映射到 `Channel.Groups["Application"].Policies["policy1"]`
- `Org1/policy2` 映射到 `Channel.Groups["Application"].Groups["Org1"].Policies["policy2"]`
- `/Channel/policy3` 映射到 `Channel.Policies["policy3"]`

> **注意**：**如果`mod_policy`引用了不存在的策略，则无法修改该项**。

# 配置更新

配置更新作为`HeaderType_CONFIG_UPDATE`类型的`Envelope`消息提交。`Payload`数据`data` 交易数据是一个封装的`ConfigUpdateEnvelope`。`ConfigUpdateEnvelope`定义如下：

```go
message ConfigUpdateEnvelope {
    bytes config_update = 1;
    repeated ConfigSignature signatures = 2;
}
```

签名`signatures` 字段包含**授权配置更新的签名集**。它的消息定义是：

```go
message ConfigSignature {
    bytes signature_header = 1;
    bytes signature = 2;
}
```

`signature_header`是为标准事务定义的，而签名是通过`ConfigUpdateEnvelope`消息中的`signature_header`字节和`config_update`字节的串联。

`ConfigUpdateEnvelope` `config_update`字节是封送的`ConfigUpdate`消息，其定义如下：

```go
message ConfigUpdate {
    string channel_id = 1;
    ConfigGroup read_set = 2;
    ConfigGroup write_set = 3;
}
```

`channel_id`是更新绑定的**通道`ID`**，这对于支持此**重新配置的签名范围**是必要的。

## 读写集

`read_set`指定**现有配置的子集**，稀疏地指定，其中**仅设置`version` 字段**，并且**不必填充其他字段**。永远不应在`read_set`中设置特定的`ConfigValue` `value`值或`ConfigPolicy` `policy`策略字段。`ConfigGroup`可以填充其**映射字段的子集**，以便**引用配置树中更深的元素**。例如，要在`read_set`中包含`Application`组，其**父级（`Channel`组）也必须包含在读取集中**，但`Channel`组**不需要填充所有键**，例如`Orderer`组键，或任何值或政策密钥。

`write_set`指定要**修改的配置片段**。由于配置的**分层特性**，对层次结构**深处的元素**的写入**也必须在其`write_set`中包含更高级别的元素**。但是，对于`write_set`中也在同一版本的`read_set`中**指定的任何元素**，**应该稀疏地指定该元素，就像在`read_set`中一样**。

例如，给定配置：

```sh
Channel: (version 0)
    Orderer (version 0)
    Application (version 3)
       Org1 (version 2)
```

要提交修改`Org1`的配置更新，`read_set`将为：

```sh
Channel: (version 0)
    Application: (version 3)
```

而`write_set`将是：

```sh
Channel: (version 0)
    Application: (version 3)
        Org1 (version 3)
```

## 新配置

收到`CONFIG_UPDATE`后，订货人通过执行以下操作来**计算生成的`CONFIG`**：

1. **验证**`channel_id`和`read_set`。`read_set`中的所有元素**必须存在于给定版本**中。
2. 通过**收集**`write_set`中**未出现**在`read_set`中**相同版本**的**所有元素**来计算更新集。
3. **验证**更新集中的每个元素是否将元素**更新的版本号精确增加`1`**。
4. **验证**附加到`ConfigUpdateEnvelope`的签名集是否**满足更新集中每个元素的`mod_policy`**。
5. 通过将更新集**应用于当前配置**来**计算**配置的**新完整版本**。
6. 将**新配置**写入`ConfigEnvelope`，其中包括`CONFIG_UPDATE`作为`last_update`字段和`config` 字段中编码的**新配置**，以及**递增的序列值`sequence` **。
7. 将新的`ConfigEnvelope`写入`CONFIG`类型的`Envelope`，并最终将其作为**新配置块中的唯一事务写入**。

当对等方（或任何其他接收方）收到此配置块时，它应通过将`last_update`消息应用于**当前配置**并**验证`orderer-computed`配置字段**包含**正确的新配置**来验证配置是否已正确验证。

# 允许的配置组和值

任何有效配置**都是以下配置的子集**。这里我们使用符号`peer.<MSG>`来定义`ConfigValue`，其`value` 字段是在`fabric/protos/peer/configuration.proto`中定义的名为`<MSG>`的封送原型消息。常见的符号`common.<MSG>`, `msp.<MSG>`, `orderer.<MSG>`对应类似，但他们的消息分别定义在`fabric/protos/common/configuration.proto`, `fabric/protos/msp/mspconfig.proto`, `fabric/protos/orderer/configuration.proto`。

请注意，键`{{org_name}}`和`{{consortium_name}}`表示任意名称，并指示可以使用不同名称重复的元素。

```go
&ConfigGroup{
    Groups: map<string, *ConfigGroup> {
        "Application":&ConfigGroup{
            Groups:map<String, *ConfigGroup> {
                {{org_name}}:&ConfigGroup{
                    Values:map<string, *ConfigValue>{
                        "MSP":msp.MSPConfig,
                        "AnchorPeers":peer.AnchorPeers,
                    },
                },
            },
        },
        "Orderer":&ConfigGroup{
            Groups:map<String, *ConfigGroup> {
                {{org_name}}:&ConfigGroup{
                    Values:map<string, *ConfigValue>{
                        "MSP":msp.MSPConfig,
                    },
                },
            },

            Values:map<string, *ConfigValue> {
                "ConsensusType":orderer.ConsensusType,
                "BatchSize":orderer.BatchSize,
                "BatchTimeout":orderer.BatchTimeout,
                "KafkaBrokers":orderer.KafkaBrokers,
            },
        },
        "Consortiums":&ConfigGroup{
            Groups:map<String, *ConfigGroup> {
                {{consortium_name}}:&ConfigGroup{
                    Groups:map<string, *ConfigGroup> {
                        {{org_name}}:&ConfigGroup{
                            Values:map<string, *ConfigValue>{
                                "MSP":msp.MSPConfig,
                            },
                        },
                    },
                    Values:map<string, *ConfigValue> {
                        "ChannelCreationPolicy":common.Policy,
                    }
                },
            },
        },
    },

    Values: map<string, *ConfigValue> {
        "HashingAlgorithm":common.HashingAlgorithm,
        "BlockHashingDataStructure":common.BlockDataHashingStructure,
        "Consortium":common.Consortium,
        "OrdererAddresses":common.OrdererAddresses,
    },
}
```

# `Orderer`系统通道配置

订购系统渠道需要定义**订购参数**，以及用于创建渠道的**联合体**。订购服务**必须只有一个订购系统渠道**，**它是第一个要创建的渠道**（或更准确地引导）。建议**永远不要在订购系统通道创建配置中定义<u>应用程序</u>部分**，但可以进行测试。请注意，对订购系统渠道具有**读访问权限的任何成员都可以看到所有渠道创建**，因此应**限制此渠道的访问权限**。

排序参数定义为以下配置子集：

```go
&ConfigGroup{
    Groups: map<string, *ConfigGroup> {
        "Orderer":&ConfigGroup{
            Groups:map<String, *ConfigGroup> {
                {{org_name}}:&ConfigGroup{
                    Values:map<string, *ConfigValue>{
                        "MSP":msp.MSPConfig,
                    },
                },
            },

            Values:map<string, *ConfigValue> {
                "ConsensusType":orderer.ConsensusType,
                "BatchSize":orderer.BatchSize,
                "BatchTimeout":orderer.BatchTimeout,
                "KafkaBrokers":orderer.KafkaBrokers,
            },
        },
    },
```

参与订购的每个组织**在`Orderer`组下都有一个组元素**。该组定义**单个参数`MSP`**，其中包含该组织的**加密身份信息**。`Orderer`组的值确定**排序节点的运行方式**。它们存在于**每个通道**中，因此`orderer.BatchTimeout`例如可以在一个通道上以不同方式指定。

在启动时，订货人面对的文件系统包含**许多渠道**的信息。订货人通过识别定义了联合体组的**渠道来识别系统渠道**。联盟具有以下结构。

```go
&ConfigGroup{
    Groups: map<string, *ConfigGroup> {
        "Consortiums":&ConfigGroup{
            Groups:map<String, *ConfigGroup> {
                {{consortium_name}}:&ConfigGroup{
                    Groups:map<string, *ConfigGroup> {
                        {{org_name}}:&ConfigGroup{
                            Values:map<string, *ConfigValue>{
                                "MSP":msp.MSPConfig,
                            },
                        },
                    },
                    Values:map<string, *ConfigValue> {
                        "ChannelCreationPolicy":common.Policy,
                    }
                },
            },
        },
    },
},
```

请注意，每个联盟都定义了**一组成员**，就像**订购组织的组织成员一样**。每个联盟还定义了`ChannelCreationPolicy`。这是一个用于**授权频道创建请求的策略**。通常，此值将设置为`ImplicitMetaPolicy`，**要求通道的新成员签名以授权创建通道**。

# 应用程序通道配置

**应用程序配置**适用于为**应用程序类型事务设计的通道**。它的定义如下：

```go
&ConfigGroup{
    Groups: map<string, *ConfigGroup> {
        "Application":&ConfigGroup{
            Groups:map<String, *ConfigGroup> {
                {{org_name}}:&ConfigGroup{
                    Values:map<string, *ConfigValue>{
                        "MSP":msp.MSPConfig,
                        "AnchorPeers":peer.AnchorPeers,
                    },
                },
            },
        },
    },
}
```

与`Orderer`部分一样，每个组织都被**编码为一个组**。然而，代替仅编码`MSP`身份信息，每个组织另外编码`AnchorPeers`列表。该列表允许**不同组织的对等方彼此联系以进行对等`gossip`网络**。

**应用程序通道对订货人组织和共识选项的副本进行编码**，以允许确定性地更新这些参数，因此包括来自订货人系统通道配置的相同`Orderer`部分。但是从应用程序的角度来看，这可能会被忽略。

# 频道创建

当订货人收到**不存在**的渠道的`CONFIG_UPDATE`时，订货人会认为这必须是**渠道创建请求**并执行以下操作。

1. 订货人**识别**要为其执行信道创建请求的**联盟**。它通过**查看顶级组的`Consortium`值**来实现。
2. 订购者**验证应用程序组**中包含的组织是相应**联盟中包含的组织的子集**，并且`ApplicationGroup`设置为`version 1`。
3. 订货人验证**如果联盟有成员，新渠道也有申请成员**（创建联盟和没有成员的渠道仅对测试有用）。
4. `orderer`通过从**订购系统通道获取`Orderer`组**，并使用**新指定的成员创建`Application`组**并将**其`mod_policy`指定为`consortium`配置中指定的`ChannelCreationPolicy`来创建模板配置**。请注意，**策略是在新配置的上下文中进行评估的**，因此需要**所有成员的策略**需要来自所有**新通道成员的签名**，而**不是联盟的所有成员**。
5. 然后，**订货人将`CONFIG_UPDATE`应用为此模板配置的更新**。**由于`CONFIG_UPDATE`对`Application` 组应用了修改**（其`version` 为`1`），因此配置代码会根据`ChannelCreationPolicy`验证**这些更新**。如果通道创建包含**任何其他修改**（例如，对于单个组织的锚点对等体），**则将调用该元素的相应`mod`策略**。
6. 具有**新通道配置的新`CONFIG`事务**被**包装**并**发送在订购系统通道上进行订购**。订购后，将创建该渠道。