# `Chaincode` 开发教程

# 什么是`Chaincode`？

`Chaincode`是一个用[`Go`](https://golang.org/)，[`node.js`](https://nodejs.org/)或[`Java`](https://java.com/en/)编写的程序，它**实现了一个指定的接口**。`Chaincode`运行在与支持对等节点的进程隔离的安全`Docker`容器中。`Chaincode`通过应用程序提交的事务初始化和管理分类帐状态。

链码通常处理网络成员同意的业务逻辑，因此可以将其视为**智能合约**。由链代码创建的状态**仅限于该链代码**，并且**不能由另一个链代码直接访问**。但是，在同一网络中，给定适当的权限，**链代码可以调用另一个链代码来访问其状态**。请注意，如果被调用的链代码与调用链代码**位于不同的通道**上，则**只允许读取查询**。也就是说，不同通道上的被调用链代码只是 `Query`，在后续提交阶段**不参与状态验证检查**。

# 两个参与人员角色

对链码有两种不同的开发参与角色：一，从应用程序开发人员的角度出发，开发一个名为[Chaincode for Developers的区块](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html)链应用程序/解决方案；[另一个是](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4noah.html)面向负责管理[区块](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4noah.html)链网络的[区块](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4noah.html)链网络运营商的[Chaincode for Operators](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4noah.html)，以及谁将利用`Hyperledger Fabric API`安装、实例化和升级链代码，但可能不会涉及链代码应用程序的开发。

**因此链码包含交易和运营两个重要的角色，运营角色负责安装、实例化和升级链码，而交易角色是指开发人员进行编写完成的代码，其内容是逻辑相关的业务的增删改查等操作**。



在以下部分中，将通过应用程序开发人员的眼睛探索链代码。将介绍一个简单的链代码示例应用程序，并介绍`Chaincode Shim API`中每个方法的用途。

# `Chaincode API`

> **注意**：还有另一组链代码`API`，允许客户端（提交者）身份用于**访问控制**决策，无论是基于**客户端身份**本身，还是**组织身份**，还是**客户端身份属性**。例如，表示为键/值的资产可以包括客户端的标识，并且**只有该客户端**可以被**授权对密钥/值进行更新**。客户端身份库具有`API`，链代码可以使用这些`API`来**检索**此提交者信息以做出此类**访问控制决策**。
>
> 本文中不会介绍它，但是[这里](https://github.com/hyperledger/fabric/blob/master/core/chaincode/lib/cid/README.md)记录了它。

每个`chaincode`程序都必须实现`Chaincode`接口，下面是`Java`的`Chaincode`接口：

```java
interface Chaincode {
	Chaincode.Response init(ChaincodeStub stub);
    
	Chaincode.Response invoke(ChaincodeStub stub);
}
```

接口文档参考：

- [`Go`](https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#Chaincode)
- [`node.js`](https://fabric-shim.github.io/ChaincodeInterface.html)
- [`Java`](https://fabric-chaincode-java.github.io/org/hyperledger/fabric/shim/Chaincode.html)

调用其方法以响应收到的交易。特别是当链码接收实例化或升级事务时调用Init方法，以便链代码可以执行任何必要的初始化，包括应用程序状态的初始化。调用Invoke方法以响应接收调用事务以处理事务提议。