# 升级 `Hyperledger Fabric` 网络组建

以 `BYFN `（`first-sample`）作为示例，进行升级操作。将 `BYFN` 从版本 `1.1.0` 升级到版本 `1.2.0` 的网络。

因为[构建第一个网络](https://hyperledger-fabric.readthedocs.io/en/release-1.2/build_network.html)（`BYFN`）教程默认采用**最新**二进制文件，如果您自`v1.2`发布以来已经运行它，您的计算机将安装`v1.2`二进制文件和工具，您将无法升级他们。

因此，本教程将提供基于`Hyperledger Fabric v1.1`二进制文件的网络以及您要升级到的`v1.2`二进制文件。此外，我们将展示如何**将通道配置更新为新的`v1.2`功能**，该功能将允许对等方正确处理[私有数据](https://hyperledger-fabric.readthedocs.io/en/release-1.2/private-data/private-data.html) 和[访问控制列表（ACL）](https://hyperledger-fabric.readthedocs.io/en/release-1.2/access_control.html)。

> 如果您的网络尚未在`Fabric v1.1`中，请按照将[网络升级到v1.1](http://hyperledger-fabric.readthedocs.io/en/release-1.1/upgrading_your_network_tutorial.html)的说明进行操作 。本文档中的说明仅涵盖从`v1.1`移至`v1.2`，而不是从任何其他版本移至`v1.2`。

由于`BYFN`不支持以下组件服务，因此我们用于升级`BYFN`的脚本不会涉及到它们：

- **Fabric CA**
- **Kafka**
- **CouchDB**
- **SDK**

 升级这些组件的过程（如有必要）将在本教程后面的部分中介绍。



