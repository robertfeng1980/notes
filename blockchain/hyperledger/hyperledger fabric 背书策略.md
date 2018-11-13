# 背书策略

背书策略用于指示同行节点如何确定交易是否得到适当背书认可。 当对等节点接收到交易时，它调用与事务的`Chaincode` 相关联的 `VSCC`（验证系统链接码）作为事务验证流程的一部分，以确定事务的有效性。回想一下，一个交易包含来自同一批人的一个或多个认可。`VSCC` 的任务是做出以下决定：

 - 所有背书认可都是有效的（即它们是来自预期消息的有效证书的有效签名）
 - 有适当数量的背书认可
 - 背书来自预期的来源

背书策略是指定第二和第三点的一种方式。

## 背书策略设计

背书策略有两个主要组成部分：
 - `a principal` 主要的
 - `a threshold gate` 阈值

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

使用此语言，链码部署者可以请求根据指定的策略验证链码的背书认可策略。 **注：默认策略需要来自DEFAULT MSP成员的一个签名** ，如果未在`CLI`中指定策略，则使用此选项。

可以在部署时使用`-P`开关指定策略，然后使用策略。示例如下：

```
peer chaincode deploy -C testchainid -n mycc -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -c '{"Args":["init","a","100","b","200"]}' -P "AND('Org1.member', 'Org2.member')"
```

此命令使用策略`AND（'Org1.member'，'Org2.member'）`在链`testchainid`上部署链码`mycc`。

## 未来的改进

在本节中，我们列出了背书策略的未来增强功能：
 - 除了通过与`MSP`的关系识别委托人的现有方式外，我们还计划根据其证书中预期的组织单位`（OU）`确定委托人，这对于表达我们从显示有效证书的任何标识请求签名的策略非常有用，该证书的`OU`与主体定义中请求的`OU`匹配。
 - 我们计划转向更直观的语法`. AND .`，而不是语法`AND（.，.）`
 - 我们计划在语言中暴露广义阈值门，以及`AND`（这是特殊的`n-out-of-n`）和`OR`（这是特殊的`1-out-of-n`）