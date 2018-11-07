# `hyperledger fabric` 向通道添加组织

在开始之前，请确保已按照安装**示例，二进制文件和Docker镜像**以及符合本文档版本的二进制文件。特别是，`fabric-samples`文件夹版本必须包含`eyfn.sh`（扩展你的第一个网络）脚本及其相关脚本。

**本教程用作构建你的第一个网络（`BYFN`）教程的扩展**，并将演示向`BYFN`自动生成的应用程序通道（`mychannel`）添加新组织`Org3`。它假设对`BYFN`有很强的理解，包括上述实用程序的用法和功能。

虽然仅关注此处新组织的扩展集成，但在执行其他**通道配置更新**（例如更新修改策略或更改批量大小）时可以采用相同的方法。要了解有关通道配置更新的过程和可能性的更多信息，请查看[更新通道配置](https://hyperledger-fabric.readthedocs.io/en/latest/config_update.html)。同样值得注意的是，像这里演示的频道配置更新通常是**组织管理员**（而不是链码或应用程序开发人员）的责任。

> **注意**：在继续之前，请确保自动`byfn.sh`脚本在计算机上运行时没有错误。如果已将**二进制文件和相关工具（`cryptogen`，`configtxgen`等）导出到`PATH`变量中**，则可以相应地修改命令而不传递完全限定的路径。

# 设置环境

将在本地`Fabric-samples`克隆中的第一个网络子目录的根目录下运行。立即切换到该目录。还需要打开一些额外的终端以方便使用。

```sh
$ cd fabric-samples/first-network
```

首先，**使用`byfn.sh`脚本进行容器或数据、镜像清理工作**。此命令将**终止所有活动或过时的`docker`容器并删除以前生成的配置工件**。为了执行通道配置更新任务，无需关闭`Fabric`网络。但是，为了本教程的目的，希望从已知的初始状态进行操作。因此，让我们运行以下命令来清理以前的所有环境：

```sh
$ ./byfn.sh down
```

现在生成默认的`BYFN`配置工件：

```sh
$ ./byfn.sh generate
```

并使用`CLI`容器中的脚本执行启动网络：

```sh
$ ./byfn.sh up
```

现在机器上运行了一个干净的`BYFN`版本，可以使用两种不同的路径。首先，提供一个完全注释的脚本，它将执行配置事务更新以将`Org3`引入网络。

此外，将显示相同过程的“**手动**”版本，显示每个步骤并解释它完成的内容（因为展示如何在此手动过程之前关闭网络，还可以运行脚本然后查看每个步骤）。

# 用脚本将`Org3`加入到通道

首先使用如下脚本，只需发出以下命令：

```sh
$ ./eyfn.sh up
```

这里的输出非常值得一读。可以看到**添加了`Org3`加密文件，创建并签署了配置更新**，然后安装了链码以允许`Org3`执行分类帐查询。

如果一切顺利，你会得到这样的信息：

```sh
========= All GOOD, EYFN test execution completed ===========
```

通过执行以下命令（而不是`./byfn.sh up`），`eyfn.sh`可以与`byfn.sh`使用相同的`Node.js`链码和数据库选项：

```sh
$ ./byfn.sh up -c testchannel -s couchdb -l node
```

然后：

```sh
$ ./eyfn.sh up -c testchannel -s couchdb -l node
```

对于那些想要仔细研究这个过程的人来说，文档的其余部分将向你展示进行频道更新的每个命令以及它的作用。

# 手动加入`Org3`到频道

下面列出的手动步骤`cli`和`Org3 cli`容器中的`CORE_LOGGING_LEVEL`设置为`DEBUG`。

对于`cli`容器，可以通过修改`first-network`目录中的`docker-compose-cli.yaml`文件来设置它。例如：

```yml
cli:
  container_name: cli
  image: hyperledger/fabric-tools:$IMAGE_TAG
  tty: true
  stdin_open: true
  environment:
    - GOPATH=/opt/gopath
    - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
    #- CORE_LOGGING_LEVEL=INFO
    - CORE_LOGGING_LEVEL=DEBUG
```

对于`Org3cli`容器，可以通过修改`first-network`目录中的`docker-compose-org3.yaml`文件来设置它。例如：

```sh
Org3cli:
  container_name: Org3cli
  image: hyperledger/fabric-tools:$IMAGE_TAG
  tty: true
  stdin_open: true
  environment:
    - GOPATH=/opt/gopath
    - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
    #- CORE_LOGGING_LEVEL=INFO
    - CORE_LOGGING_LEVEL=DEBUG
```

如果使用过`eyfn.sh`脚本，则需要关闭网络。这将关闭网络，删除所有容器并撤消我们为添加`Org3`所做的工作。可以通过执行命令：

```sh
$ ./eyfn.sh down
```

当网络关闭时，请重新启动它：

```sh
$ ./byfn.sh generate

$ ./byfn.sh up
```

这将使网络恢复到执行`eyfn.sh`脚本之前的状态。现在准备手动添加`Org3`了。作为第一步，需要生成`Org3`的加密文件。

# 生成`Org3`加密文件

在另一个终端中，从`first-network`切换到`org3-artifacts`子目录。

```sh
$ cd org3-artifacts
```

这里有两个`yaml`文件：`org3-crypto.yaml`和`configtx.yaml`。首先，为`Org3`生成加密文件：

```sh
$ ../../bin/cryptogen generate --config=./org3-crypto.yaml
```

此命令读入新加密`yaml`文件 `org3-crypto.yaml` 并利用`cryptogen`为`Org3 CA`以及绑定到此新组织的两个对等方**生成密钥和证书**。与`BYFN`实现一样，此加密文件将放入当前工作目录（在示例中为`org3-artifacts`）中新生成的`crypto-config`文件夹中。

现在使用`configtxgen`实用程序在`JSON`中打印出特定于`Org3`的配置材料。通过告诉工具在当前目录中查找它需要提取的`configtx.yaml`文件来执行命令。

```sh
$ export FABRIC_CFG_PATH=$PWD && ../../bin/configtxgen -printOrg Org3MSP > ../channel-artifacts/org3.json
```

上面的命令创建一个`JSON`文件 `org3.json`，并将其输出到`first-network`根目录下的`channel-artifacts`子目录中。此文件包含`Org3`的**背书策略**定义，以及以`base 64`格式显示的三个重要证书：**管理员用户证书（稍后将充当`Org3`的管理员），`CA`根证书和`TLS`根目录证书**。在接下来的步骤中，将此`JSON`文件添加到通道配置。

最后的工作是将`Orderer Org`的**`MSP`材料移植到`Org3` `crypto-config`目录**中。特别是，**关注的是`Orderer`的`TLS`根证书，它将允许`Org3`实体与网络订购节点之间的安全通信**。

```sh
$ cd ../ && cp -r crypto-config/ordererOrganizations org3-artifacts/crypto-config/
```

