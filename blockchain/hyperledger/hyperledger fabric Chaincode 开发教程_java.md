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

