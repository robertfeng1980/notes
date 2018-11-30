# `Chaincode` 开发模式

通常，链路由**对等体启动和维护**。然而，在**开发模式**中，链代码由用户构建和启动。在链代码开发阶段，此模式非常有用，可用于**快速代码/构建/运行/调试周期周转**。

通过为**示例开发网络**利用**预先生成的订购者和渠道工件**来启动“**开发模式**”。这样，用户可以**立即进入编译链码和驱动回调**的过程。

## 安装`Fabric`示例代码

如果还没有这样做，请[安装示例代码和相关工具](hyperledger%20fabric%20建立你的第一个网络.md)。

切换到`fabric-samples clone`代码下的`chaincode-docker-devmode`目录：

```sh
$ cd fabric-samples/chaincode-docker-devmode
```

## 下载`docker`镜像

需要四个`docker`镜像，以便**开发模式**提供的`docker compose`脚本运行。如果安装了`fabric-samples repo clone`并按照说明安装了**[示例、二进制文件和`docker`镜像](hyperledger%20fabric%20入门.md)**，那么在本地应该安装必要的`Docker`镜像。

> **注意**：如果选择**手动拉取**镜像，则必须将其重新标记为**最新版本**。

执行`docker images`命令，显示本地的`Docker Registry`。应该看到类似于以下内容的内容：

```sh
$ docker iamges

REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
hyperledger/fabric-javaenv     amd64-1.3.0         2476cefaf833        7 weeks ago         1.7GB
hyperledger/fabric-javaenv     latest              2476cefaf833        7 weeks ago         1.7GB
hyperledger/fabric-tools       amd64-1.3.0         c056cd9890e7        7 weeks ago         1.5GB
hyperledger/fabric-tools       latest              c056cd9890e7        7 weeks ago         1.5GB
hyperledger/fabric-ccenv       amd64-1.3.0         953124d80237        7 weeks ago         1.38GB
hyperledger/fabric-ccenv       latest              953124d80237        7 weeks ago         1.38GB
hyperledger/fabric-orderer     amd64-1.3.0         f430f581b46b        7 weeks ago         145MB
hyperledger/fabric-orderer     latest              f430f581b46b        7 weeks ago         145MB
hyperledger/fabric-peer        amd64-1.3.0         f3ea63abddaa        7 weeks ago         151MB
hyperledger/fabric-peer        latest              f3ea63abddaa        7 weeks ago         151MB
```

# 资产转账示例

以下将演示`Go`、`Java`的链码运行示例方式。

## `Go`版本链码

### 终端1 - 启动网络

```sh
$ docker-compose -f docker-compose-simple.yaml up
```

以上内容使用`SingleSampleMSPSolo`订购者配置文件启动网络，并以“**开发模式**”启动对等体。它还**启动了两个额外的容器**：一个用于**链代码环境 `chaincode`**，另一个**用于与链代码交互 `CLI`**。**创建和加入通道**的命令嵌入在`CLI`容器中，可以查看`script.sh`脚本的代码，因此可以立即跳转到链代码调用。

### 终端2 - 构建并启动链码

此步骤仅限于`Go` 语言的链码，`Java`链码可以在其他开发环境进行打包构建。

```sh
$ docker exec -it chaincode bash
```

应该看到以下内容：

```sh
root@d2629980e76b:/opt/gopath/src/chaincode#
```

现在，编译链码：

```sh
$ cd chaincode_example02/go
$ go build -o chaincode_example02
```

现在运行链码：

```sh
$ CORE_PEER_ADDRESS=peer:7052 CORE_CHAINCODE_ID_NAME=mycc:0 ./chaincode_example02

2018-11-30 03:32:22.215 UTC [shim] setupChaincodeLogging -> INFO 001 Chaincode log level not provided; defaulting to: INFO
2018-11-30 03:32:22.248 UTC [shim] setupChaincodeLogging -> INFO 002 Chaincode (build level: ) starting up ...
```

链代码以**对等和链代码日志**开始，表示**向对等方成功注册**。请注意，在此阶段，链码**不与任何通道相关联**。这是使用`instantiate`命令在后续步骤中完成的。

### 终端3 - 使用链码

即使处于`--peer-chaincodedev`模式，仍然**必须安装链代码**，以便**生命周期**系统链代码可以**正常进行检查**。在`--pere-chaincodedev`模式下，将来可能会**删除**此要求。

进入到`CLI`容器，将利用`CLI`容器来调用链码进行交互。

```sh
$ docker exec -it cli bash
```

安装和实例化链码

```sh
$ peer chaincode install -p chaincodedev/chaincode/chaincode_example02/go -n mycc -v 0
$ peer chaincode instantiate -n mycc -v 0 -c '{"Args":["init","a","100","b","200"]}' -C myc
```

现在发出一个调用，将`10`从`a`移到`b`。

```sh
$ peer chaincode invoke -n mycc -c '{"Args":["invoke","a","b","10"]}' -C myc
```

最后，查询一个。应该看到`90`的值。

```sh
$ peer chaincode query -n mycc -c '{"Args":["query","a"]}' -C myc
```

### 测试新的链码

默认情况下，只挂载`chaincode_example02`。但是，可以通过**将不同的链代码添加到`fabric-samples\chaincode`子目录**并**重新启动网络**来**轻松地测试**它们。此时，可以在**链代码容器中访问**它们。

## `Java`版本链码

同上`Go`版本运行示例步骤。
### 终端1 - 重启网络

```sh
$ docker-compose -f docker-compose-simple.yaml down
$ docker-compose -f docker-compose-simple.yaml up
```

### 终端2 - 构建并启动链码

`Java`链码可以在`windows`下的`eclipse`开发环境进行打包构建，也可以在`chaincode`环境打包。
现在，编译链码（前提是安装好`maven`和`Gradle`工具）：

```sh
$ cd fabric-samples/chaincode/chaincode_example02/java

$ ll


$ gradle clean build shadowJar
```
编译完成后，会产生`jar`文件

```sh
$ ll build/libs/
total 16583
drwxrwxrwx 1 vagrant vagrant        0 Nov 30 03:40 ./
drwxrwxrwx 1 vagrant vagrant        0 Nov 30 03:40 ../
-rwxrwxrwx 1 vagrant vagrant 16976115 Nov 30 03:41 chaincode.jar*
-rwxrwxrwx 1 vagrant vagrant     3724 Nov 30 03:40 fabric-chaincode-example-gradle-1.0-SNAPSHOT.jar*
```

进入`chaincode`容器

```sh
$ docker exec -it chaincode bash
```

应该看到以下内容：
```sh
root@d2629980e76b:/opt/gopath/src/chaincode#
```

现在运行链码：

```sh
$ cd chaincode_example02/java

$ CORE_PEER_ADDRESS=peer:7052 CORE_CHAINCODE_ID_NAME=mycc:0 java -cp .；./build/libs/*.jar；./build/classes/java/main org.hyperledger.fabric.example.SimpleChaincode.class
```
这里运行的仅仅是`Java`链码中的`main`主函数。

### 终端3 - 使用链码

即使处于`--peer-chaincodedev`模式，仍然**必须安装链代码**，以便**生命周期**系统链代码可以**正常进行检查**。在`--pere-chaincodedev`模式下，将来可能会**删除**此要求。

进入到`CLI`容器，将利用`CLI`容器来调用链码进行交互。

```sh
$ docker exec -it cli bash

root@c9e6b820d754:/opt/gopath/src/chaincodedev#
```

安装和实例化链码

```sh
$ peer chaincode install -n mycc -v 0 -l java -p ./chaincode/chaincode_example02/java/

$ peer chaincode instantiate -n mycc -v 0 -c '{"Args":["init","a","100","b","200"]}' -C myc -l java
```

现在发出一个调用，将`10`从`a`移到`b`。

```sh
$ peer chaincode invoke -n mycc -c '{"Args":["invoke","a","b","10"]}' -C myc
```

最后，查询一个。应该看到`90`的值。

```sh
$ peer chaincode query -n mycc -c '{"Args":["query","a"]}' -C myc
```

## `JAR`版本链码
同上`Java`版本运行示例步骤。
### 终端1 - 重启网络

```sh
$ docker-compose -f docker-compose-simple.yaml down
$ docker-compose -f docker-compose-simple.yaml up
```

### 终端2 - 构建链码

`Java`链码可以在`windows`下的`eclipse`开发环境进行打包构建，也可以在`chaincode`环境打包。
现在，编译链码（前提是安装好`maven`和`Gradle`工具）：

```sh
$ cd fabric-samples/chaincode/chaincode_example02/java
$ gradle clean build shadowJar
```
编译完成后，会产生`jar`文件

```sh
 ll build/libs/
total 16583
drwxrwxrwx 1 vagrant vagrant        0 Nov 30 03:40 ./
drwxrwxrwx 1 vagrant vagrant        0 Nov 30 03:40 ../
-rwxrwxrwx 1 vagrant vagrant 16976115 Nov 30 03:41 chaincode.jar*
-rwxrwxrwx 1 vagrant vagrant     3724 Nov 30 03:40 fabric-chaincode-example-gradle-1.0-SNAPSHOT.jar*
```

将`JAR`文件和`gradle`配置文件一起放置到一个新的目录`jar`中

```sh
$ cp fabric-samples/chaincode/chaincode_example02/java/build/libs/chaincode.jar fabric-samples/chaincode/chaincode_example02/jar/

# 经过测试，发现打包好 JAR 的不需要 *.gradle 配置文件
$ cp fabric-samples/chaincode/chaincode_example02/java/*.gradle fabric-samples/chaincode/chaincode_example02/jar/

$ ll fabric-samples/chaincode/chaincode_example02/jar

total 16580
drwxrwxrwx 1 vagrant vagrant        0 Nov 30 02:17 ./
drwxrwxrwx 1 vagrant vagrant        0 Nov 30 02:46 ../
-rwxrwxrwx 1 vagrant vagrant      621 Nov  1 09:32 build.gradle*
-rwxrwxrwx 1 vagrant vagrant 16976113 Nov  6 09:43 chaincode.jar*
-rwxrwxrwx 1 vagrant vagrant       54 Nov  1 09:31 settings.gradle*
```

### 终端3 - 使用链码

进入到`CLI`容器，将利用`CLI`容器来调用链码进行交互。

```sh
$ docker exec -it cli bash

root@c9e6b820d754:/opt/gopath/src/chaincodedev#
```

安装和实例化链码

```sh
$ peer chaincode install -n mycc -v 0 -l java -p ./chaincode/chaincode_example02/jar/

$ peer chaincode instantiate -n mycc -v 0 -c '{"Args":["init","a","100","b","200"]}' -C myc -l java
```

现在发出一个调用，将`10`从`a`移到`b`。

```sh
$ peer chaincode invoke -n mycc -c '{"Args":["invoke","a","b","10"]}' -C myc
```

最后，查询一个。应该看到`90`的值。

```sh
$ peer chaincode query -n mycc -c '{"Args":["query","a"]}' -C myc
```

