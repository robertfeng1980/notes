# 升级 `Hyperledger Fabric` 网络组建

以 `BYFN `（`first-sample`）作为示例，进行升级操作。将 `BYFN` 从版本 `1.1.0` 升级到版本 `1.2.0` 的网络。

# 概述

因为[构建第一个网络](https://hyperledger-fabric.readthedocs.io/en/release-1.2/build_network.html)（`BYFN`）教程默认采用**最新**二进制文件，如果您自`v1.2`发布以来已经运行它，您的计算机将安装`v1.2`二进制文件和工具，您将无法升级他们。

因此，本教程将提供基于`Hyperledger Fabric v1.1`二进制文件的网络以及您要升级到的`v1.2`二进制文件。此外，我们将展示如何**将通道配置更新为新的`v1.2`功能**，该功能将允许对等方正确处理[私有数据](https://hyperledger-fabric.readthedocs.io/en/release-1.2/private-data/private-data.html) 和[访问控制列表（ACL）](https://hyperledger-fabric.readthedocs.io/en/release-1.2/access_control.html)。

> **注意**：如果您的网络尚未在`Fabric v1.1`中，请按照将[网络升级到v1.1](http://hyperledger-fabric.readthedocs.io/en/release-1.1/upgrading_your_network_tutorial.html)的说明进行操作 。本文档中的说明仅涵盖从`v1.1`移至`v1.2`，而不是从任何其他版本移至`v1.2`。

## 不被升级的组件

由于`BYFN`不支持以下组件服务，因此我们用于升级`BYFN`的脚本不会涉及到它们：

- **`Fabric CA`**
- **`Kafka`**
- **`CouchDB`**
- **`SDK`**

升级这些组件的过程（如有必要）将在本教程后面的部分中介绍。

## 升级流程

我们的升级教程将执行以下步骤：

1. 备份分类帐本和`MSP`：将包含 `orderer`、`peer` 容器中的账本数据和 `MSP` 证书配置。
2. 将`orderer`二进制文件升级到`Fabric v1.2`。
3. 将`peer`二进制文件升级到`Fabric v1.2`。
4. 启用新的`v1.2`功能。

> **注意**：在生产环境中，`orderer`和`peer`可以**同时进行滚动升级**。换句话说，您可以**按任何顺序升级二进制文件，而无需关闭网络**。因为`BYFN`使用**SOLO** 排序共识服务，我们的脚本会使整个网络崩溃。但这在生产环境中不是必需的。
>
> 但是，确保启用功能不会对当前正在运行的`orderer`和`peer`版本产生问题非常重要。对于`v1.2`，新功能位于应用程序组中，该应用程序组管理与对等方节点相关的功能，因此不会与排序服务冲突。

本教程将演示如何使用`CLI`命令单独执行每个步骤。 

## 运行环境

如果还没有这样做，请确保计算机上具有所有依赖项，如[环境条件中所述](https://hyperledger-fabric.readthedocs.io/en/release-1.2/prereqs.html)。

# 启动`v1.1`网络

首先，我们将提供运行`Fabric v1.1` 镜像的基本网络。该网络将由**两个组织**组成，**每个组织维护两个对等节点，以及一个`独立`排序服务**。

将在本地克隆的`fabric-samples`子目录中运行`first-network`。立即切换到该目录。还需要打开一些额外的终端以方便使用。

```sh
$ cd opt/gopath/src/github.com/hyperledger/fabric-samples
$ git clone -b master https://github.com/hyperledger/fabric-samples.git
$ cd fabric-samples
$ git checkout v1.1.0
```

## 清理

保证干净清洁的环境是必要的，因此将使用`byfn.sh`脚本进行初步整理工作。此命令将终止(**停止并删除**)所有活动或过时的`docker`容器，并删除任何以前生成的文件。运行以下命令：

```sh
$ ./byfn.sh down
```

## 生成加密文件并启动网络

在干净的环境中，使用以下四个命令启动我们的`v1.1 BYFN`网络： 

```sh
# 生成加密文件，证书、公钥、私钥
$ ./byfn.sh generate
# 启动网络，设置延时时间和 镜像版本
$ ./byfn.sh up -t 3000 -i 1.1.0
```

> **注意**：如果本地构建的`v1.1`镜像，则示例将使用它们。如果您遇到错误，请考虑清理本地构建的`v1.1`映像并再次运行该示例。这将从`docker hub`下载`v1.1`映像。

如果`BYFN`正确启动，你会看到：

```
===================== All GOOD, BYFN execution completed =====================
```

现在准备将我们的网络升级到`Hyperledger Fabric v1.2`。

## 获取最新示例代码

> **注意**：以下说明适用于最新发布的`v1.2.x`版本。请将`1.2.x`替换为正在测试的已发布版本的版本标识符。换句话说，如果正在测试第一个候选版本，请将“`1.2.x`”替换为“`1.2.0`”。

在完成本教程的其余部分之前，获取样本的`v1.2.x`版本非常重要，可以通过以下方式执行此操作：

```sh
$ git fetch origin
$ git checkout v1.2.x
```

## 想立即升级吗？

这里有一个脚本可以升级`BYFN`中的所有组件以及启用功能。如果正在运行生产网络，或者是网络某个部分的管理员，则此脚本可用作执行自己升级的模板。

然后，通过完成脚本中的步骤，并描述每个代码在升级过程中所执行的操作。

要运行该脚本，请发出以下命令：

```sh
# Note, replace '1.2.x' with a specific version, for example '1.2.0'.
# Don't pass the image flag '-i 1.2.x' if you prefer to default to 'latest' images.
# 如果您希望默认为“最新”图像，请不要传递图像标记'-i 1.2.x'。

$ ./byfn.sh upgrade -i 1.2.x
```

如果升级成功，您应该看到以下内容：

```sh
======== All GOOD, End-2-End UPGRADE Scenario execution completed ================
```

如果要手动升级网络，只需再次运行`./byfn.sh down`并执行步骤 `./byfn.sh upgrade -i 1.2.x`。然后继续下一部分。

# 升级orderer容器

