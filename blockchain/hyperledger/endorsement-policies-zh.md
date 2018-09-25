# 背书策略

背书策略用于指示同行节点如何确定交易是否得到适当背书认可。 当对等节点接收到交易时，它调用与事务的`Chaincode` 相关联的 `VSCC`（验证系统链接码）作为事务验证流程的一部分，以确定事务的有效性。回想一下，一个交易包含来自同一批人的一个或多个认可。`VSCC` 的任务是做出以下决定：

 - 所有背书认可都是有效的（即它们是来自预期消息的有效证书的有效签名）
 - 有适当数量的背书认可
 - 背书来自预期的来源

背书策略是指定第二和第三点的一种方式。

## 背书策略设计

背书策略有两个主要组成部分：
 - a principal 主要的
 - a threshold gate 阈值

委托人`P`识别预期签名的实体。

阈值`T`采用两个输入：整数`t`（阈值）和`n`个主要或阀门的列表，这个门基本上捕获了那些要求满足这些`n`个校长或门的期望。

example:
 - `T(2, 'A', 'B', 'C')` 要求`A`，`B`或`C`之外的任何`2` `principals `签名
 - `T(1, 'A', T(2, 'B', 'C'))` 请求来自 `principal`  `A`的一个签名或来自`B`和`C`的1个签名
## 背书策略语法客户端

在`CLI`中，使用简单语言来表示基于主体的布尔表达式的策略。

根据`MSP`描述委托人，该委员会的任务是验证签名者的身份以及签名者在该`MSP`中的角色。 目前，支持两个角色： **member** and **admin**。 `principal` 被描述为`MSP`。`ROLE`，其中`MSP`是所需的`MSP ID`，`ROLE`是两个字符串`member`和`admin`之一。有效主体的示例是`'Org0.admin'`（`Org0 MSP`的任何管理员）或`'Org1.member'`（`Org1 MSP`的任何成员）。 

该语言的语法是：

`EXPR(E[, E...])`

其中`EXPR`是`AND`或`OR`，表示两个布尔表达式，E是主体（使用上述语法）或另一个嵌套调用EXPR。

示例：
 - `AND('Org1.member', 'Org2.member', 'Org3.member')`  请求三个`principals`中的每一个签名
 - `OR('Org1.member', 'Org2.member')`  从两个`principals`中的任何一个请求`1`个签名
 - `OR('Org1.member', AND('Org2.member', 'Org3.member'))` 请求来自`Org1 MSP`成员的一个签名或来自`Org2 MSP`成员的1个签名和来自`Org3 MSP`成员的1个签名。
## 为链码指定背书策略

Using this language, a chaincode deployer can request that the endorsements for a chaincode be
validated against the specified policy. NOTE - the default policy requires one signature 
from a member of the `DEFAULT` MSP). This is used if a policy is not specified in the CLI.

The policy can be specified at deploy time using the `-P` switch, followed by the policy.

For example:

```
peer chaincode deploy -C testchainid -n mycc -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -c '{"Args":["init","a","100","b","200"]}' -P "AND('Org1.member', 'Org2.member')"
```

This command deploys chaincode `mycc` on chain `testchainid` with the policy `AND('Org1.member', 'Org2.member')`.

## Future enhancements

In this section we list future enhancements for endorsement policies:
 - alongside the existing way of identifying principals by their relationship with an MSP, we plan 
   to identify principals in terms of the _Organization Unit (OU)_ expected in their certificates; 
   this is useful to express policies where we request signatures from any identity displaying a 
   valid certificate with an OU matching the one requested in the definition of the principal.
 - instead of the syntax `AND(., .)` we plan to move to a more intuitive syntax `. AND .`
 - we plan to expose generalized threshold gates in the language as well alongside `AND` (which is 
   the special `n`-out-of-`n` gate) and `OR` (which is the special `1`-out-of-`n` gate)