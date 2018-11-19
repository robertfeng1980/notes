# `hyperledger fabric` 访问控制列表`ACL`

> **注意**：本文介绍**通道管理级别的访问控制和策略**。要了解链代码中的访问控制，请查看[开发链代码教程](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#Chaincode_API)。

`Fabric`使用访问控制列表（`ACL`）来**管理对资源的访问**，方法是将**策略（在给定一组身份的情况下指定评估为`true`或`false`的规则）与资源相关联**。`Fabric`包含许多默认`ACL`。在本文档中，将讨论如何格式化它们以及如何覆盖默认值。

# 资源

用户通过**定位[用户链代码](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html)，[系统链代码](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4noah.html)或[事件流源](https://hyperledger-fabric.readthedocs.io/en/latest/peer_event_services.html)**与`Fabric`进行交互。因此，这些端点被视为应在其上执行访问控制的**资源**。

应用程序开发人员需要了解这些资源以及与之**关联的默认策略**。这些资源的**完整列表可在`configtx.yaml`中找到**。可以[在此处查看示例`configtx.yaml`文件](http://github.com/hyperledger/fabric/blob/release-1.2/sampleconfig/configtx.yaml)。

`configtx.yaml`中指定的资源**是`Fabric`当前定义的所有内部资源的详尽列表**。那里采用的松散约定是`<component>/<resource>`。因此`cscc/GetConfigBlock`是`CSCC`组件中`GetConfigBlock`调用的资源。

# 策略

策略是`Fabric`工作方式的基础，因为它们允许根据与完成请求所需**资源相关联的策略**来检查与请求关联的身份（或身份集）。**认可政策用于确定交易是否得到适当认可**。通道配置中定义的策略被引用为**修改策略**以及**访问控制**，并在通道配置本身中定义。

策略可以采用以下两种方式之一进行组织：作为**签名策略**或作为`ImplicitMeta`策略。

## 签名策略

这些策略标识必须**签署的特定用户**才能满足策略。例如：

```yml
Policies:
  MyPolicy:
    Type: Signature
    Rule: “Org1.Peer OR Org2.Peer”
```

此策略构造可以解释为：名为`MyPolicy`的策略，只能通过具有“**来自`Org1`的对等体**”或“**来自`Org2`的对等体**”角色的身份的**签名来满足**。

签名策略支持`AND，OR和NOutOf`的任意组合，允许构建极其强大的规则，例如：“组织管理员和其他两个管理员，或20个组织管理员中的11个”。

## `ImplicitMeta` 策略

`ImplicitMeta`策略**聚合配置层次结构中更深层次的策略结果**，这些策略最终由`Signature` 签名策略定义。它们支持**默认规则**，例如“大多数组织管理员”。与`Signature` 签名策略相比，这些策略使用不同但仍然非常简单的语法：`<ALL|ANY|MAJORITY> <sub_policy>`

例如：任何读者`ANY` `Readers`或`MAJORITY`管理员`MAJORITY` `Admins`。

请注意，在默认策略配置中，`Admins`具有操作角色。指定**只有管理员**或**管理员的某个子集**，可以访问资源的策略将倾向于用于**网络的敏感或操作方面**（例如在通道上实例化链代码）。`Writers` 倾向于能够**提出分类帐更新**，例如交易，但通常**不具有管理权限**。`Readers` 有被动的角色。**可以访问信息**，但**无权提出分类帐更新**，也**无法执行管理任务**。可以**添加、编辑或补充**这些默认策略，例如，通过新的对等和客户端角色（如果具有`NodeOU`支持）。

以下是`ImplicitMeta`策略结构的示例：

```yml
Policies:
  AnotherPolicy:
    Type: ImplicitMeta
    Rule: "MAJORITY Admins"
```

在这里，策略`AnotherPolicy`可以由管理员的`MAJORITY`满足，其中管理员最终由**较低级别的签名策略指定**。