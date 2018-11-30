# `Java Chaincode` 开发教程

本教程将讲解如何编写基于`Java`的`Hyperledger Fabric`链代码。有关链码的一般说明，如何编写和操作，请访问[Chaincode教程](hyperledger%20fabric%20Chaincode%20开发教程)。

# 必要工具

+ `JDK8+`
+ `Gradle 4.8+`
+ `Hyperledger Fabric 1.3+`

# 环境配置



# 简单的`Chaincode`开发示例

编写自己的链代码需要了解`Fabric`平台，`Java`和`Gradle`。应用程序是一个基本的示例链代码，用于在分类帐上创建资产（键值对）。



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

通过为示例开发网络，利用预先生成的定序者和通道工件来启动“开发模式(`dev mode`)”。这样，用户可以立即进入编译链码和操作调用的过程。

# 安装`Hyperledger Fabric`示例

如果还没有这样做，请[安装样本，二进制文件和Docker镜像](https://hyperledger-fabric.readthedocs.io/en/latest/install.html)。

进入到`fabric-samples`项目的`chaincode-docker-devmode`目录：

```sh
$ cd chaincode-docker-devmode
```

现在打开三个终端并导航到每个终端中的`chaincode-docker-devmode`目录。

# 终端1 - 启动网络

```sh
$ docker-compose -f docker-compose-simple.yaml up
```

以上内容**使用`SingleSampleMSPSolo`定序者配置文件**启动网络，并以“**开发模式**”启动对等体。它还启动了两个额外的容器，一个**用于链码环境**，另一个**用于与链代码交互**。创建和加入通道的命令嵌入在`CLI`容器中，因此可以立即跳转到链代码调用。

# 终端2  - 构建并启动链码

```sh
$ docker exec -it chaincode bash
```

应该看到以下内容：

```sh
root@d2629980e76b:/opt/gopath/src/chaincode#
```

现在，编译链码：

```sh
$ cd sacc
$ go build
```

现在运行链码：

```sh
$ CORE_PEER_ADDRESS=peer:7052 CORE_CHAINCODE_ID_NAME=mycc:0 ./sacc
```

**链代码以对等节点和链代码日志开始输出，表示向对等方成功注册**。请注意，在**此阶段，链码不与任何通道相关联**。这是使用`instantiate`命令在后续步骤中完成的。

# 终端3  - 使用链码

即使处于`--peer-chaincodedev`模式，仍然**必须安装链代码**，以便**生命周期系统链码可以正常进行检查**。在`--pere-chaincodedev`模式下，将来可能会删除此要求。

```sh
$ docker exec -it cli bash
```

在 `cli` 容器中运行：

```sh
$ peer chaincode install -p chaincodedev/chaincode/sacc -n mycc -v 0

$ peer chaincode instantiate -n mycc -v 0 -c '{"Args":["a","10"]}' -C myc
```

现在发出一个`invoke`调用，将`a`的值更改为“`20`”。

```sh
$ peer chaincode invoke -n mycc -c '{"Args":["set", "a", "20"]}' -C myc
```

最后，查询`a`。应该看到`20`的值。

```sh
$ peer chaincode query -n mycc -c '{"Args":["query","a"]}' -C myc
```

# 测试新的链码

默认情况下，只挂载`sacc`。但是，可以通过将不同的**链码添加到`chaincode`子目录**并**重新启动网络来轻松地测试它们**。此时，可以在`chaincode`容器中访问它们。

# `Chaincode`加密

在某些情况下，**完全或仅部分地加密与密钥相关联的值**可能是有用的。例如，如果某人的社会安全号码或地址被写入分类帐，那么可能不希望此数据以**明文形式出现**。通过利用**实体扩展**来实现`Chaincode`加密，该[实体扩展](https://github.com/hyperledger/fabric/tree/master/core/chaincode/shim/ext/entities)是具有**物品工厂和功能的`BCCSP`包装器**，以执行加密操作，例如加密和椭圆曲线数字签名。例如，为了加密，链代码的调用者**通过瞬态字段传递加密密钥**。然后可以将**相同的密钥用于后续查询操作**，从而允许**对加密状态值进行适当的解密**。

有关更多信息和示例，请参阅`fabric/examples`目录中的[`Encc`示例](https://github.com/hyperledger/fabric/tree/master/examples/chaincode/go/enccc_example)。特别注意`utils.go`助手程序。此实用程序**加载`chaincode shim API`和实体扩展**，并构建一个新类型的函数（例如，`encryptAndPutState`和`getStateAndDecrypt`），然后**加密示例链代码将利用这些函数**。因此，链码现在可以与`Get`和`Put`的基本`shim API`结合使用`Encrypt`和`Decrypt`的附加功能。

# 管理用`Go`编写的链码的外部依赖关系

如果链码需要`Go`标准库未提供的包，则需要将这些包与链码一起包含在内。有[许多工具](https://github.com/golang/go/wiki/PackageManagementTools)可用于管理这些依赖项。以下演示了如何使用`govendor`：

```sh
$ govendor init
$ govendor add +external  // Add all external package, or
$ govendor add github.com/external/pkg // Add specific external package
```

这会将**外部依赖项导入本地供应商目录**。然后，对等链码包和对等链码安装操作将包含与链代码包中的依赖关联的代码。