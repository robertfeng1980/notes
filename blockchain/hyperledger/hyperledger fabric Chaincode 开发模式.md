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
hyperledger/fabric-orderer     amd64-1.3.0         f430f581b46b        7 
