# `Java Chaincode` 开发教程

本教程将讲解如何编写基于`Java`的`Hyperledger Fabric`链代码。有关链码的一般说明，如何编写和操作，请访问[Chaincode教程](hyperledger%20fabric%20Chaincode%20开发教程)。

# 必要工具

+ `JDK8+`
+ `Gradle 4.8+`
+ `Hyperledger Fabric 1.3+`

# 环境配置

这里主要是配置 `Gradle` 的环境变量。

```sh
export path=path:/opt/gradle-4.8
```

# 简单的`Chaincode`开发示例

编写自己的链代码需要了解`Fabric`平台，`Java`和`Gradle`。应用程序是一个基本的示例链代码，用于在分类帐上创建资产（键值对）。

## 下载代码

```sh
$ git clone https://github.com/hyperledger/fabric-chaincode-java.git
```

在开发工具`eclipse`中导入工程代码。

**文件夹结构**：

`fabric-chaincode-protos` 文件夹包含`Java shim`用于与`Fabric`对等方通信的`protobuf`定义文件。

`fabric-chaincode-shim` 文件夹包含定义`Java`链代码`API`的`java shim`类以及与`Fabric`对等方通信的方式。

`fabric-chaincode-docker` 文件夹包含构建`docker`镜像的说明 `hyperledger/fabric-javaenv`。

`fabric-chaincode-example-gradle` 包含一个示例`java chaincode gradle`项目，其中包含示例链代码和基本`gradle`构建指令。

## 创建`Gradle`项目

可以使用`fabric-chaincode-example-gradle`作为起始点。确保项目构建创建一个可运行的`jar`，其中包含名为`chaincode.jar`的所有依赖项。

```groovy
plugins {
    id 'com.github.johnrengelman.shadow' version '2.0.3'
    id 'java'
}

group 'org.hyperledger.fabric-chaincode-java'
version '1.3.1-SNAPSHOT'

sourceCompatibility = 1.8

repositories {
    mavenLocal()
    mavenCentral()
}

dependencies {
    compile group: 'org.hyperledger.fabric-chaincode-java', name: 'fabric-chaincode-shim', version: '1.3.+'
    testCompile group: 'junit', name: 'junit', version: '4.12'
}

shadowJar {
    baseName = 'chaincode'
    version = null
    classifier = null

    manifest {
        attributes 'Main-Class': 'com.github.hooj0.chaincode.SimpleAssetChaincode'
    }
}
```

新建完成后，可以在项目上右键，选择`Gradle -> Refresh Gradle Project`加载项目依赖的`Jar`包。

## 创建链码类

使用`Java`版的[`Simple Asset Chaincode`](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#simple-asset-chaincode)作为示例。这个链代码是`Simple Asset Chaincode`的`Go to Java`翻译，将对此进行解释。

`ChaincodeBase`类是一个**抽象类**，它继承了`Chaincode`形式，它包含用于启动`chaincode`的`start`方法。因此，将通过扩展`ChaincodeBase`而不是实现`Chaincode`来创建我们的链代码。

首先，从一些基本的开始，创建一个`class`文件`SimpleAssetChaincode`。与每个链代码一样，它**继承了[`ChaincodeBase`抽象类](https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#Chaincode)，特别是实现了`init`和`invoke`函数**。

```java
package com.github.hooj0.chaincode;

import org.hyperledger.fabric.shim.ChaincodeBase;
import org.hyperledger.fabric.shim.ChaincodeStub;

/**
 * simple asset chaincode
 * @author hoojo
 * @createDate 2018年11月30日 下午4:13:27
 * @file SimpleAssetChaincode.java
 * @package com.github.hooj0.chaincode
 * @project fabric-chaincode-asset-gradle
 * @blog http://hoojo.cnblogs.com
 * @email hoojo_@126.com
 * @version 1.0
 */
public class SimpleAssetChaincode extends ChaincodeBase {

}
```

## 初始化`Chaincode`

接下来，将**实现`init`函数**。

```java
    /**
     * Init is called during chaincode instantiation to initialize any
     * data. Note that chaincode upgrade also calls this function to reset
     * or to migrate data.
     *
     * @param stub {@link ChaincodeStub} to operate proposal and ledger
     * @return response
     */
    @Override
    public Response init(ChaincodeStub stub) {       
    }
```

> **注意**：链码**升级也会调用此函数**。在编写将升级现有链代码的链代码时，请确保正确修改`Init`函数。特别是，如果**没有“迁移”或者在升级过程中没有任何内容要初始化，请提供一个空的`Init`方法**。

接下来，将**使用`ChaincodeStub -> stub.getStringArgs`函数检索`Init`调用的参数并检查其有效性**。在例子中，将使用一个键值对。`Chaincode`初始化在`Response init(ChaincodeStub stub)`方法内完成。首先，使用`ChaincodeStub.getStringArgs()`方法获取参数。

```go
    /**
     * Init is called during chaincode instantiation to initialize any
     * data. Note that chaincode upgrade also calls this function to reset
     * or to migrate data.
     *
     * @param stub {@link ChaincodeStub} to operate proposal and ledger
     * @return response
     */
    @Override
    public Response init(ChaincodeStub stub) {
        // Get the args from the transaction proposal
        List<String> args = stub.getStringArgs();
        if (args.size() != 2) {
            newErrorResponse("Incorrect arguments. Expecting a key and a value");
        }
        return newSuccessResponse();
    }
```

接下来，既然已经确定调用有效，将**把初始状态存储在分类帐中**。为此，将调用`stub.putStringState`作为参数传入的键和值。假设一切顺利，返回一个`Response`对象，表明初始化成功。

```java
    /**
     * Init is called during chaincode instantiation to initialize any
     * data. Note that chaincode upgrade also calls this function to reset
     * or to migrate data.
     *
     * @param stub {@link ChaincodeStub} to operate proposal and ledger
     * @return response
     */
    @Override
    public Response init(ChaincodeStub stub) {
        try {
            // Get the args from the transaction proposal
            List<String> args = stub.getStringArgs();
            if (args.size() != 2) {
                newErrorResponse("Incorrect arguments. Expecting a key and a value");
            }
            // Set up any variables or assets here by calling stub.putState()
            // We store the key and the value on the ledger
            stub.putStringState(args.get(0), args.get(1));
            return newSuccessResponse();
        } catch (Throwable e) {
            return newErrorResponse("Failed to create asset");
        }
    }
```

