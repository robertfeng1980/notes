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

