# ` hyperledger fabric` 访问控制列表`ACL`

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

# 访问控制在哪里指定？

**访问控制默认值存在于`configtx.yaml`中**，`configtxgen`用于构建通道配置的文件。

可以通过编辑`configtx.yaml`本身（可以**将`ACL`更改传播到任何新通道**）或通过更新**特定通道**的通道**配置**中的访问控制**来更新访问控制**，方法有两种。

# 如何在`configtx.yaml`中格式化`ACL`

`ACL`被格式化为**键值对**，由**资源函数名称后跟字符串组成**。要查看它，请[参考此示例`configtx.yaml`文件](https://github.com/hyperledger/fabric/blob/release-1.2/sampleconfig/configtx.yaml)。

示例参考：

```yaml
# ACL policy for invoking chaincodes on peer
peer/Propose: /Channel/Application/Writers

# ACL policy for sending block events
event/Block: /Channel/Application/Readers
```

这些`ACL`定义对`peer/Propose`和`event/Block`**资源的访问限制**为分别满足在规范路径`/Channel/Application/Writers`和`/Channel/Application/Readers`中**定义的策略的身份**。

# 在`configtx.yaml`中更新`ACL`默认值

如果在引导网络时需要**覆盖`ACL`默认值**，或者在**引导通道之前更改`ACL`**，最佳做法是**更新`configtx.yaml`**。

假设你要修改 `ACL `默认`peer/Propose` 它指定了在**对等体上调用链代码的策略**， 从`/Channel/Application/Writers`到一个名为`MyPolicy`的策略。

这是通过添加一个名为`MyPolicy`的策略来完成的（它可以被称为任何东西，但是对于这个例子称之为`MyPolicy`）。该**策略在`configtx.yaml`内的`Application.Policies`部分中定义**，并指定要**检查以授予或拒绝用户访问权限的规则**。在本例中，将创建一个标识`SampleOrg.admin`的签名策略。

```yml
Policies: &ApplicationDefaultPolicies
    Readers:
        Type: ImplicitMeta
        Rule: "ANY Readers"
    Writers:
        Type: ImplicitMeta
        Rule: "ANY Writers"
    Admins:
        Type: ImplicitMeta
        Rule: "MAJORITY Admins"
    MyPolicy:
        Type: Signature
        Rule: "OR('SampleOrg.admin')"
```

然后，编辑`configtx.yaml`中的`Application`：`ACLs`部分以更改`peer/Propose`：

`peer/Propose: /Channel/Application/Writers` 改为： `peer/Propose: /Channel/Application/MyPolicy`

在`configtx.yaml`中更改了这些字段后，`configtxgen`工具将使用**创建通道创建事务时定义的策略和`ACL`**。当联盟成员的一个管理员适当地**签名和提交**时，将创建具有**已定义的`ACL`和策略的新通道**。一旦`MyPolicy`被引导到通道配置中，它也可以被引用以**覆盖**其他`ACL`默认值。例如：

```yml
SampleSingleMSPChannel:
    Consortium: SampleConsortium
    Application:
        <<: *ApplicationDefaults
        ACLs:
            <<: *ACLsDefault
            event/Block: /Channel/Application/MyPolicy
```

这会**限制**将块事件订阅到`SampleOrg.admin`的能力。如果已创建了要使用此`ACL`的通道，则必须使用以下流程一次更新一个通道配置

# 更新通道配置中的`ACL`默认值

如果已经创建了希望使用`MyPolicy`来**限制对等/建议访问的通道**，或者如果想要创建`ACL`，**不希望其他渠道知道**。将**不得不一次更新一个通道配置**通过配置更新事务。

注意：渠道配置事务是一个我们不会在此深入研究的过程。如果想了解更多关于它们的信息，请查看[关于频道配置更新的文档](https://hyperledger-fabric.readthedocs.io/en/latest/config_update.html)以及[向频道添加组织](https://hyperledger-fabric.readthedocs.io/en/latest/channel_update_tutorial.html)教程。

在**提取、转换和剥离**其元数据的配置块之后，可以**通过在`Application: policies`下添加`MyPolicy`来编辑配置**，其中`Admins`，`Writers`和`Readers`策略已经存在。

```json
"MyPolicy": {
  "mod_policy": "Admins",
  "policy": {
    "type": 1,
    "value": {
      "identities": [
        {
          "principal": {
            "msp_identifier": "SampleOrg",
            "role": "ADMIN"
          },
          "principal_classification": "ROLE"
        }
      ],
      "rule": {
        "n_out_of": {
          "n": 1,
          "rules": [
            {
              "signed_by": 0
            }
          ]
        }
      },
      "version": 0
    }
  },
  "version": "0"
},
```

请**特别注意`msp_identifer`和角色**。然后，在配置的`ACLs`部分中，更改`peer/Propose` 的`ACL`：

```json
"peer/Propose": {
  "policy_ref": "/Channel/Application/Writers"
```

更改为：

```json
"peer/Propose": {
  "policy_ref": "/Channel/Application/MyPolicy"
```

> **注意**：如果通道配置中未定义`ACL`，则必须添加整个`ACL`结构。

配置更新后，需要通过**通常的频道更新流程**提交。

# 访问多个资源的`ACL`

如果成员发出调用多个系统链代码的请求，则**必须满足这些系统链代码的所有`ACL`**。

例如，`peer/Propose`引用通道上的**任何提议请求**。如果特定提案需要访问需要满足`Writers`的身份的两个系统链代码和需要满足`MyPolicy`的身份的一个系统链代码，则提交提议的**成员必须具有对`Writers`和`MyPolicy`评估为`true`的身份**。

在默认配置中，`Writers`是一个**签名策略**，其规则`rule`是`SampleOrg.member`。换句话说，“我组织的**任何成员**”。上面列出的`MyPolicy`具有`SampleOrg.admin`规则或“我的组织的**任何管理员**”规则。要满足这些`ACL`，该成员必须是**管理员和`SampleOrg`的成员**。默认情况下，**所有管理员都是成员**（尽管**并非**所有管理员都是成员），但可以将这些策略覆盖到任何状态。因此，跟踪这些策略以确保同行提议的`ACL`不是不可能满足（除非是这样）是很重要的。

# 使用实验性`ACL`功能的客户的迁移注意事项

以前，访问控制列表的管理是在**通道创建事务的`isolated_data`部分中完成**的，并**通过`PEER_RESOURCE_UPDATE`事务**进行更新。最初，人们认为资源树将处理多个函数的更新，这些函数最终以其他方式处理，因此**维护单独的并行对等节点配置树被认为是不必要**的。

可以使用`v1.1`中的实验资源树进行客户迁移。由于官方`v1.2`版本**不支持旧的`ACL`方法**，因此网络运营商应**关闭所有对等方**。然后，他们应该将它们升级到`v1.2`，**提交一个通道重新配置事务**，该事务启用`v1.2`功能并设置所需的`ACL`，然后最终**重新启动升级的对等体**。重新启动的对等体**将立即使用新的通道配置并根据需要强制执行`ACL`**。