# `Chaincode` 开发模式

通常，链路由**对等体启动和维护**。然而，在**开发模式**中，链代码由用户构建和启动。在链代码开发阶段，此模式非常有用，可用于**快速代码/构建/运行/调试周期周转**。

通过为**示例开发网络**利用**预先生成的订购者和渠道工件**来启动“**开发模式**”。这样，用户可以**立即进入编译链码和驱动回调**的过程。

## 安装`Fabric`示例代码

如果还没有这样做，请[安装示例代码和相关工具](hyperledger%20fabric%20建立你的第一个网络.md)。

切换到`fabric-samples clone`代码下的`chaincode-docker-devmode`目录：

```sh
$ cd fabric-samples/chaincode-docker-devmode
```


