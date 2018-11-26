# `hyperledger fabric` 性能要求

由于`Fabric`是一个通常**涉及多个组织的分布式系统**（有时在不同的国家），因此**网络中可能存在**（并且通常）**许多不同版本**的`Fabric`代码。然而，网络以**相同的方式处理事务**以使每个人对**当前网络状态具有相同的视图**是至关重要的。

这意味着**每个网络**，以及该网络中的**每个通道**，必须定义一组称之为“**性能**”的**才能参与处理交易**。例如，`Fabric v1.1`引入了新的`MSP`角色类型`Peer`和`Client`。但是，如果`v1.0`对等方**不理解这些新角色类型**，则**无法正确评估**引用它们的认可策略。这意味着在可以使用新角色类型之前，网络**必须同意启用`v1.1`信道功能**要求，确保所有对等方做出相同的决策。

只有支持所需功能的**二进制文件**才能参与该通道，而较新的二进制版本在启用相应功能之前**不会启用新的验证逻辑**。通过这种方式，功能要求确保即使使用不同的构建和版本，网络也不可能遭受状态分叉。

# 定义性能要求

通道配置中的每个通道定义了**性能要求**（可在通道的**最新配置块**中找到）。通道配置包含**三个位置**，每个位置定义不同类型的功能。

| Capability Type | Canonical Path                    | JSON Path                                              |
| --------------- | --------------------------------- | ------------------------------------------------------ |
| Channel         | /Channel/Capabilities             | .channel_group.values.Capabilities                     |
| Orderer         | /Channel/Orderer/Capabilities     | .channel_group.groups.Orderer.values.Capabilities      |
| Application     | /Channel/Application/Capabilities | .channel_group.groups.Application.values. Capabilities |

- **Channel**：这些功能适用于**对等和订购者**，并且位于**根通道组**中。
- **Orderer**：仅适用于`orderers`，位于`Orderer`组。
- **Application**：仅适用于**对等**项，位于`Application`组中。

这些功能被分解为这些组，以便与现有的**管理结构保持一致**。更新`orderer`功能是**订购组织独立于应用程序组织管理的内容**。同样，更新应用程序功能只是**应用程序管理员**可以管理的内容。通过在`Orderer`和`Application`之间**拆分功能**，假设网络可以运行`v1.6`订购服务，**同时支持`v1.3`对等应用程序网络**。

但是，某些功能**跨越了`Application`和`Orderer`组**。如前所述，**添加新的`MSP`角色**类型是订购者和应用程序管理员同意并需要**识别的内容**。**订购者必须理解`MSP`角色的含义**，以便允许**事务通过排序**，而**对等方必须理解角色才能验证事务**。这些功能，**跨越应用程序和订购者组件**，在**顶级“通道”组中定义**。

> **注意**：通道功能可能定义为版本`v1.3`，而定购商和应用程序功能分别定义为版本`1.1`和`v1.4`。**在“通道”组级别启用功能并不意味着在更具体的“订购者”和“应用程序”组级别上可以使用相同的功能**。

# 设置功能

功能被设置为**通道配置的一部分**（作为**初始配置**的一部分或作为重新配置的一部分）。

> **注意**：我们有两个文档，介绍了通道重新配置的不同方面。首先，我们有一个教程，将指导您完成[向通道添加组织的过程](https://hyperledger-fabric.readthedocs.io/en/latest/channel_update_tutorial.html)。我们还有一个文档，通过[更新通道配置](https://hyperledger-fabric.readthedocs.io/en/latest/config_update.html)进行讨论，该配置概述了可能的各种更新以及更全面的签名过程。

由于新通道**默认复制`Orderer`系统通道的配置**，因此将**自动配置新通道**以使用`Orderer`系统通道的**订购者和通道功能**以及通道创建事务指定的应用程序功能。但是，**必须重新配置已存在的通道**。

`Capabilities`值的模式在`protobuf`中定义为：

```go
message Capabilities {
      map<string, Capability> capabilities = 1;
}

message Capability { }
```

作为示例，以`JSON`呈现：

```go
{
    "capabilities": {
        "V1_1": {}
    }
}
```

