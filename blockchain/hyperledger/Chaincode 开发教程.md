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

调用其方法以响应收到的交易。特别是当链码接收**实例化或升级交易**时调用`init`方法，以便链代码可以**执行任何必要的初始化**，包括应用程序状态的初始化。**调用`invoke`方法以响应接收调用交易以处理交易提议**。

链码**`shim` API**中的另一个接口是`ChaincodeStubInterface`：

- [`Go`](https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStubInterface)
- [`node.js`](https://fabric-shim.github.io/ChaincodeStub.html)
- [`Java`](https://fabric-chaincode-java.github.io/org/hyperledger/fabric/shim/ChaincodeStub.html)

用于**访问和修改分类帐**，以及**在链码之间**进行调用。

# 简单的`Chaincode`开发示例

应用程序是一个基本的示例链代码，用于在分类帐上创建资产（键值对）。

## 选择`Chaincode`的位置

如果还没有在`Go`中进行编程，可能需要确保安装了[`Go Programming Language`](https://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html#golang)并正确配置了系统环境。

现在，将要**为链码应用程序创建一个目录**，作为`$GOPATH/src/`的子目录。为了简单起见，使用以下命令：

```sh
$ mkdir -p $GOPATH/src/sacc && cd $GOPATH/src/sacc
```

现在，创建将用代码填写的源文件：

```sh
$ touch sacc.go
```

## 准备开始

首先，从一些基本的开始。与每个链代码一样，它**实现了[`Chaincode`接口](https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#Chaincode)，特别是`Init`和`Invoke`函数**。因此，为链码**添加必要依赖项的`Go import`语句**。将**导入`chaincode shim`包和[`peer protobuf`包](https://godoc.org/github.com/hyperledger/fabric/protos/peer)**。接下来，添加一个**结构`SimpleAsset`作为`Chaincode shim `函数的接收器**。

```go
package main

import (
    "fmt"

    "github.com/hyperledger/fabric/core/chaincode/shim"
    "github.com/hyperledger/fabric/protos/peer"
)

// SimpleAsset implements a simple chaincode to manage an asset
type SimpleAsset struct {
}
```

## 初始化`Chaincode`

接下来，将**实现`Init`函数**。

```go
// Init is called during chaincode instantiation to initialize any data.
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
}
```

> **注意**：链码**升级也会调用此函数**。在编写将升级现有链代码的链代码时，请确保正确修改`Init`函数。特别是，如果**没有“迁移”或者在升级过程中没有任何内容要初始化，请提供一个空的`Init`方法**。

接下来，将**使用`ChaincodeStubInterface.GetStringArgs`函数检索`Init`调用的参数并检查其有效性**。在例子中，将使用一个键值对。

```go
// Init is called during chaincode instantiation to initialize any
// data. Note that chaincode upgrade also calls this function to reset
// or to migrate data, so be careful to avoid a scenario where you
// inadvertently clobber your ledger's data!
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
  // Get the args from the transaction proposal
  args := stub.GetStringArgs()
  if len(args) != 2 {
    return shim.Error("Incorrect arguments. Expecting a key and a value")
  }
}
```

接下来，既然已经确定调用有效，将**把初始状态存储在分类帐中**。为此，将调用`ChaincodeStubInterface.PutState`作为参数传入的键和值。假设一切顺利，返回一个`peer.Response`对象，表明初始化成功。

```go
// Init is called during chaincode instantiation to initialize any
// data. Note that chaincode upgrade also calls this function to reset
// or to migrate data, so be careful to avoid a scenario where you
// inadvertently clobber your ledger's data!
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
  // Get the args from the transaction proposal
  args := stub.GetStringArgs()
  if len(args) != 2 {
    return shim.Error("Incorrect arguments. Expecting a key and a value")
  }

  // Set up any variables or assets here by calling stub.PutState()

  // We store the key and the value on the ledger
  err := stub.PutState(args[0], []byte(args[1]))
  if err != nil {
    return shim.Error(fmt.Sprintf("Failed to create asset: %s", args[0]))
  }
  return shim.Success(nil)
}
```

## 调用`Chaincode`

首先，**添加`Invoke`函数的签名**。

```go
// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The 'set'
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {

}
```

与上面的`Init`函数一样，需要**从`ChaincodeStubInterface`中提取参数**。**`Invoke`函数的参数将是要调用的链代码应用程序函数的名称**。在例子中，应用程序将只有两个函数：`set`和`get`，它们允许设置资产的值或检索其当前状态。首先**调用`ChaincodeStubInterface.GetFunctionAndParameters`来为该链代码应用程序函数提取函数名称和参数**。

```go
// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The Set
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
    // Extract the function and args from the transaction proposal
    fn, args := stub.GetFunctionAndParameters()
}
```

接下来，将函数名称验证为`set`或`get`，并调用这些链代码应用程序函数，通过`shim.Success`或`shim.Error`函数返回适当的响应，这些函数将**响应序列化为`gRPC protobuf`消息**。

```go
// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The Set
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
    // Extract the function and args from the transaction proposal
    fn, args := stub.GetFunctionAndParameters()

    var result string
    var err error
    if fn == "set" {
            result, err = set(stub, args)
    } else {
            result, err = get(stub, args)
    }
    if err != nil {
            return shim.Error(err.Error())
    }

    // Return the result as success payload
    return shim.Success([]byte(result))
}
```

## 实现`Chaincode`应用程序

如上所述，链码应用程序实现了两个可以通过`Invoke`函数调用的函数，现在实现这些功能。请注意，正如上面提到的，为了**访问分类帐的状态**，将利用`chaincode shim API`的[`ChaincodeStubInterface.PutState`](https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.PutState)和[`ChaincodeStubInterface.GetState`](https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.GetState)函数。

```go
// Set stores the asset (both key and value) on the ledger. If the key exists,
// it will override the value with the new one
func set(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    if len(args) != 2 {
            return "", fmt.Errorf("Incorrect arguments. Expecting a key and a value")
    }

    err := stub.PutState(args[0], []byte(args[1]))
    if err != nil {
            return "", fmt.Errorf("Failed to set asset: %s", args[0])
    }
    return args[1], nil
}

// Get returns the value of the specified asset key
func get(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    if len(args) != 1 {
            return "", fmt.Errorf("Incorrect arguments. Expecting a key")
    }

    value, err := stub.GetState(args[0])
    if err != nil {
            return "", fmt.Errorf("Failed to get asset: %s with error: %s", args[0], err)
    }
    if value == nil {
            return "", fmt.Errorf("Asset not found: %s", args[0])
    }
    return string(value), nil
}
```

## 示例完整代码

最后，需要添加`main`函数，它将调用`shim.Start`函数。这是整个链码程序的完整源代码文件。

```go
package main

import (
    "fmt"

    "github.com/hyperledger/fabric/core/chaincode/shim"
    "github.com/hyperledger/fabric/protos/peer"
)

// SimpleAsset implements a simple chaincode to manage an asset
type SimpleAsset struct {
}

// Init is called during chaincode instantiation to initialize any
// data. Note that chaincode upgrade also calls this function to reset
// or to migrate data.
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
    // Get the args from the transaction proposal
    args := stub.GetStringArgs()
    if len(args) != 2 {
            return shim.Error("Incorrect arguments. Expecting a key and a value")
    }

    // Set up any variables or assets here by calling stub.PutState()

    // We store the key and the value on the ledger
    err := stub.PutState(args[0], []byte(args[1]))
    if err != nil {
            return shim.Error(fmt.Sprintf("Failed to create asset: %s", args[0]))
    }
    return shim.Success(nil)
}

// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The Set
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
    // Extract the function and args from the transaction proposal
    fn, args := stub.GetFunctionAndParameters()

    var result string
    var err error
    if fn == "set" {
            result, err = set(stub, args)
    } else { // assume 'get' even if fn is nil
            result, err = get(stub, args)
    }
    if err != nil {
            return shim.Error(err.Error())
    }

    // Return the result as success payload
    return shim.Success([]byte(result))
}

// Set stores the asset (both key and value) on the ledger. If the key exists,
// it will override the value with the new one
func set(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    if len(args) != 2 {
            return "", fmt.Errorf("Incorrect arguments. Expecting a key and a value")
    }

    err := stub.PutState(args[0], []byte(args[1]))
    if err != nil {
            return "", fmt.Errorf("Failed to set asset: %s", args[0])
    }
    return args[1], nil
}

// Get returns the value of the specified asset key
func get(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    if len(args) != 1 {
            return "", fmt.Errorf("Incorrect arguments. Expecting a key")
    }

    value, err := stub.GetState(args[0])
    if err != nil {
            return "", fmt.Errorf("Failed to get asset: %s with error: %s", args[0], err)
    }
    if value == nil {
            return "", fmt.Errorf("Asset not found: %s", args[0])
    }
    return string(value), nil
}

// main function starts up the chaincode in the container during instantiate
func main() {
    if err := shim.Start(new(SimpleAsset)); err != nil {
            fmt.Printf("Error starting SimpleAsset chaincode: %s", err)
    }
}
```

## 构建`Chaincode`

现在编译构建链码。

```sh
$ go get -u github.com/hyperledger/fabric/core/chaincode/shim
$ go build
```

假设没有错误，现在可以继续下一步，测试链代码。

## 使用开发模式测试

通常，**链码由对等体启动和维护**。然而，在“**开发模式**”中，**链码由用户构建和启动**。在链码开发阶段，此模式非常有用，可用于**快速代码/构建/运行/调试**周期周转。

通过为示例开发网络，利用预先生成的订购者和渠道工件来启动“开发模式(`dev mode`)”。这样，用户可以立即进入编译链码和操作调用的过程。

# 安装`Hyperledger Fabric`示例

如果还没有这样做，请[安装样本，二进制文件和Docker镜像](https://hyperledger-fabric.readthedocs.io/en/latest/install.html)。