# `Chaincode` 运营操作

`Chaincode`是一个用[`Go`](https://golang.org/)，[`node.js`](https://nodejs.org/)或[`Java`](https://java.com/en/)编写的程序，它**实现了一个指定的接口**。`Chaincode`运行在与支持对等节点的进程隔离的安全`Docker`容器中。`Chaincode`通过应用程序提交的**事务初始化和管理分类帐状态**。

链码通常处理**网络成员同意认可的业务逻辑**，因此可以将其视为“**智能合约**”。由**链码创建的状态仅限于该链码，并且不能由另一个链码直接访问**。但是，在同一网络中，给定适当的权限，链码可以**调用另一个链码**来访问其状态。

在以下部分中，将通过区块链网络运营商`Noah`的眼睛探索链码。为了诺亚的利益，将专注于**链码生命周期**运营；根据区块链网络中链码的**运营生命周期、打包、安装、实例化和升级**链码的过程。

# `Chaincode` 生命周期

`Hyperledger Fabric API`支持与区块链网络中的**各个节点（对等体，订购者和`MSP`）进行交互**，并且还允许人们**在支持对等节点上打包，安装，实例化和升级**链代码。`Hyperledger Fabric`语言特定的`SDK`抽象出`Hyperledger Fabric API`的细节以促进应用程序开发，尽管它可用于管理链代码的生命周期。此外，可以通过`CLI`直接访问`Hyperledger Fabric API`，将在本文档中使用它。

系统提供了四个命令来管理链码的生命周期：**包、安装、实例化和升级**。在将来的版本中，正在考虑添加停止和启动事务来禁用和重新启用链代码而无需实际卸载它。**成功安装和实例化链码后，链码处于活动状态（正在运行），并可以通过调用事务处理事务，链码可能在安装后随时升级**。

# `Chaincode` 打包

**链码包由3部分组成**：

+ 链码由`ChaincodeDeploymentSpec`或`CDS`定义。`CDS`根据**代码和其他属性（如名称和版本）**定义链代码包
+ **可选**的实例化策略，可以通过用于**认可策略**的相同在语法上描述，并在[认可策略](https://hyperledger-fabric.readthedocs.io/en/latest/endorsement-policies.html)中描述
+ 由**拥有**链码的实体签署的一组签名

**签名用于以下目的**：

+ 建立链码的**所有权**
+ 允许**验证**包的内容
+ 允许**检测**包篡改

根据链码的实例化策略**验证**通道上链码的**实例化交易的创建者**。

## 创建包

包装链码有两种方法：一个用于**何时**要拥有链码的**多个所有者**，因此需要使用**多个身份签名**的链代码包。此工作流程要求最初**创建一个签名**的链代码包（`SignedCDS`），随后将其**串行**传递给**其他所有者以进行签名**。

更简单的工作流程适用于部署仅具有发起`install`交易的节点标识的签名的`SignedCDS`。首先解决更复杂的案例。但是，如果还不需要担心多个所有者，可以跳到下面的安装链代码部分。

要**创建签名的链码包**，请使用以下命令：

```sh
$ peer chaincode package -n mycc -p github.com/hyperledger/fabric/examples/chaincode/go/example02/cmd -v 0 -s -S -i "AND('OrgA.admin')" ccpack.out
```

`-s`选项**创建一个可由多个所有者签名的包**，而不是简单地创建原始`CDS`。指定`-s`时，**如果其他所有者需要签名，则还必须指定`-S`选项**。否则，该过程将创建一个`SignedCDS`，除了`CDS`之外，该策略仅包括实例化策略。

`-S`选项指示进程使用由`core.yaml`中`localMspid`属性的值标识的`MSP`对包进行`signpackage` 。

可选的`-i`选项允许为链码**指定实例化策略**。**实例化策略具有与认可策略相同的格式**，并指定哪些身份可以实例化链代码。在上面的示例中，只允许`OrgA`的管理员实例化链代码。如果**未提供策略，则使用默认策略，该策略仅允许对等方的`MSP`的管理员标识实例化链代码**。

## 包签名

在创建时签署的链码包可以**移交给其他所有者进行检查和签名**。该工作流程支持链码包的带外签名。

[`ChaincodeDeploymentSpec`](https://github.com/hyperledger/fabric/blob/master/protos/peer/chaincode.proto#L78)可以选择由**集体所有者签名**，以创建[`SignedChaincodeDeploymentSpec`](https://github.com/hyperledger/fabric/blob/master/protos/peer/signed_cc_dep_spec.proto#L26)（或`SignedCDS`）。`SignedCDS`包含3个元素：

+ `CDS`包含**源代码**，链码的**名称和版本**。
+ 链码的**实例化策略**，表示为背书认可策略。
+ 通过[背书](https://github.com/hyperledger/fabric/blob/master/protos/peer/proposal_response.proto#L111)认可策略定义的链码**所有者列表**。

> **注意**：此绑定策略是在带外确定的，以便在某些通道上实例化链代码时提供适当的`MSP`主体。如果未指定实例化策略，则**默认策略是该通道的任何`MSP`管理员**。

每个所有者通过将`ChaincodeDeploymentSpec`与该所有者的身份（例如证书）相结合并签署合并结果来认可`ChaincodeDeploymentSpec`。

链码所有者可以使用以下命令对先前创建的已签名包进行签名：

```sh
$ peer chaincode signpackage ccpack.out signedccpack.out
```

其中`ccpack.out`和`signedccpack.out`分别是**输入和输出包**。`signedccpack.out`包含使用本地`MSP`签名的程序包的**附加签名**。

## 安装链码

安装事务将链代码的源代码打包成称为`ChaincodeDeploymentSpec`（或`CDS`）的规定格式，并将其安装在将运行该链代码的**对等节点**上。

> **注意**：必须在将运行链码的通道的**每个支持对等节点**上安装链代码。

如果只为`ChaincodeDeploymentSpec`提供安装`API`，它将**默认实例化策略**并包含一个**空的所有者**列表。

> **注意**：`Chaincode`只应安装在**链码拥有成员的对等节点**上，以**保护链码逻辑**与网络上其他成员的**机密性**。那些**没有链码**的成员，**不能成为链码交易的参与者**。也就是说，他们**无法执行**链码。但是，他们仍然**可以验证事务并将其提交到分类帐**。

要安装链代码，请将[`SignedProposal`](https://github.com/hyperledger/fabric/blob/master/protos/peer/proposal.proto#L104)发送到[`System Chaincode`](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4noah.html#system-chaincode)部分中描述的生命周期系统链代码（`LSCC`）。例如，要使用`CLI`安装[`Simple Asset Chaincode`](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#simple-asset-chaincode)部分中描述的`sacc`示例链代码，命令将如下所示：

```sh
$ peer chaincode install -n asset_mgmt -v 1.0 -p sacc
```

`CLI`在内部为`sacc`创建`SignedChaincodeDeploymentSpec`并将其发送到**本地对等方**，后者在`LSCC`上调用`Install`方法。`-p`选项的参数**指定了链代码的路径**，该链代码必须位于用户`GOPATH`的源树中，例如，`$GOPATH/src/sacc`。有关命令选项的完整说明，请参阅`CLI`部分。

请注意，为了在对等方上安装，`SignedProposal`的签名**必须来自对等方的本地`MSP`管理员**。

