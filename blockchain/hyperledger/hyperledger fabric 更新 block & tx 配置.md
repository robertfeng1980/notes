# Hyperledger Fabric 更新`configtx`配置



# 概述

`configtxlator`工具是为支持独立于SDK的**重新配置**而创建的。**通道配置**作为**交易**存储在通道的**配置块**中，并可以直接操作，例如在`bdd`行为测试中。然而，在写这篇文章的时候，**没有SDK本身支持直接操作配置**，因此`configtxlator`工具旨在提供API，供任何SDK的使用者与之交互以**协助配置更新**。

工具名称是 `configtx`和 `translator`的`portmanteau`，旨在表达该工具只是在不同的等效数据之间进行转换。它**不会生成配置**，**不会提交或检索配置**，也**不会修改配置**本身，它只是在`configtx`格式的**不同视图**之间提供了一些双向操作。

标准用法如下：
1. SDK **检索最新**的配置
2. `configtxlator`  转换人类**可读**的配置版本
3. 用户或应用程序**编辑**配置
4. `configtxlator` 用于计算更改后的**增量**配置
5. SDK **提交标志并提交配置**

`configtxlator`工具公开了一个真正的无状态`REST API`，用于与配置元素进行交互。这些`REST`组件支持将本机配置格式**转换为人类可读的JSON**，或者根据两种配置之间的差异计算增量配置更新。

由于`configtxlator`服务特意**不包含任何加密材料或其他秘密信息**，因此它**不包含任何授权或访问控制**。预期的典型部署将是在本地与应用程序一起作为沙盒容器运行，以便为其每个使用者提供专用的配置程序进程。

# 构建并运行 `configtxlator`

`configtxlator`二进制文件可以通过执行`make configtxlator`生成。

```sh
$ cd /opt/gopath/src/github.com/hyperledger/fabric

$ make configtxlator
```

或者通过HTTP方式去下载`configtxlator`二进制文件

```sh
$ curl https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric/hyperledger-fabric/${ARCH}-${VERSION}/hyperledger-fabric-${ARCH}-${VERSION}.tar.gz | tar xz
```

将上面的 `${ARCH}-${VERSION}`变量用 `linux-amd64-1.1.0` 替换即可。这里的 `linux-amd64-1.1.0` 是系统类型和当前`fabric`代码的版本。

要执行`configtxlator`，只需执行二进制文件，可选地使用`-serverPort`选项指定侦听端口，或接受默认监听端口`7059`。`configtxlator`二进制工具将启动一个监听指定端口的HTTP服务器，可以处理请求。

```sh
$ configtxlator start

2018-05-31 09:42:36.066 UTC [configtxlator] startServer -> INFO 001 Serving HTTP requests on 0.0.0.0:7059
```

# `proto` 转换

为了可扩展性，由于某些字段**必须被签名**，许多`proto`字段以**字节存储**。这使得`JSON`转换的自然`proto`语言使用`jsonpb`包无法生成**人类可读**版本的`protobufs`。相反，`configtxlator`公开`REST`组件可以做更复杂的转换。

为了将`proto`转换为人类可读的`JSON`配置，只需将二进制`proto`提交到目标`http://$SERVER:$PORT/protolator/decode/<message.Name>`，其中`<message.Name>`是完全限定的消息`proto`名称。

例如，要解码保存为`configuration_block.pb`的配置块，请运行以下命令：

```sh
$ curl -X POST --data-binary @configuration_block.pb http://127.0.0.1:7059/protolator/decode/common.Block
```

要将人类可读的`JSON`版本转换为`proto`原始消息，只需将`JSON`版本提交到`http://$SERVER:$PORT/protolator/encode/<message.Name>`其中`<message.Name>`又是消息的完全限定的`proto`名称。

例如，重新编码保存为`configuration_block.json`的块，运行命令：

```sh
$ curl -X POST --data-binary @configuration_block.json http://127.0.0.1:7059/protolator/encode/common.Block
```

任何配置相关的`PROTOS`的名称，其中包括`common.Block`，`common.Envelope`，`common.ConfigEnvelope`，`common.ConfigUpdateEnvelope`，`common.Config`，和`common.ConfigUpdate`都是这些URL的有效目标。未来，可能会添加其他原始解码类型，例如用于代言者交易。 

# 计算增量配置

给定**两种不同**的配置，可以**计算**在它们之间转换后的**增量更新配置**。只需将两个`common.Config`  `proto`编码的配置作为`multipart/formdata` （原始字段`original`并更新为字段`updated` ）`POST`到`http://$SERVER:$PORT/configtxlator/compute/update-from-configs`。 

例如，给定`proto`配置文件为`original_config.pb`，并将更新后的配置文件`updated_config.pb`作为所需`desiredchannel`通道的文件：

```sh
$ curl -X POST -F channel=desiredchannel -F original=@original_config.pb -F updated=@updated_config.pb http://127.0.0.1:7059/configtxlator/compute/update-from-configs
```

# 示例演示

下面用示例演示完整的流程，包括转换 `proto` 和 `json`，计算增量更新配置文件。

## 环境变量设置

如果在执行 `make` 脚本期间出现错误，表示 `环境变量`设置不对。下面三种方法设置环境变量：

```sh
$ sudo make configtxlator
Makefile:92: *** "No go in PATH: Check dependencies".  Stop.
```

### `export` 命令设置

```sh
# 直接用export命令 设置变量
$ export GOPATH="/opt/gopath" \
export GOROOT="/opt/go" \
export PATH=$GOROOT/bin:$GOPATH/bin:$PATH
# 查看变量
$ export
```

### 修改`profile`文件

```sh
$ vi /etc/profile
# 在里面加入:  
export GOPATH="/opt/gopath"
export GOROOT="/opt/go"
export PATH=$GOROOT/bin:$GOPATH/bin:$PATH

# 让环境变量立即生效需要执行如下命令：  
$ source /etc/profile  
```

### 修改`.bashrc`文件：  

```sh
$ vi /root/.bashrc

# 在里面加入：  
export GOPATH="/opt/gopath"
export GOROOT="/opt/go"
export PATH=$GOROOT/bin:$GOPATH/bin:$PATH
```

设置完成后查看变量是否生效 `echo $PATH`，查看是否有命令存在。不存在的情况对应修改文件的方式，需要注销当前用户重新登入。

## 将二进制文件配置到标准命令

如果不在标准位置下，每次运行工具需要在文件名前面需要加上完整的路径。 

### 临时配置

---

有一个方法就是设置环境变量，将当前路径设置到环境变量中，但是**退出这个命令行就失效**了。

```sh
$ export PATH=$PATH:$GOPATH/src/github.com/hyperledger/fabric/release/linux-amd64/bin
```

### 永久配置

---

要想永久生效，需要把这行添加到环境变量文件里。有两个文件可选：`/etc/profile`和用户主目录下的`.bash_profile`，`etc/profile`对系统里**所有用户都有效**，用户主目录下的`.bash_profile`只对当前主目录对应的用户有效。

```sh
# 所有用户有效
$ vi /etc/profile
# 加入配置
export PATH="$PATH:$GOPATH/src/github.com/hyperledger/fabric/release/linux-amd64/bin"
$ source /etc/profile

# 当前用户有效
$ cd ~
$ vi .bash_profile
export PATH="$PATH:$GOPATH/src/github.com/hyperledger/fabric/release/linux-amd64/bin"
```

## 构建并启动 `configtxlator`

```sh
$ cd /opt/gopath/src/github.com/hyperledger/fabric

# 构建 configtxlator
$ make configtxlator
# 或者
$ make release

# linux-amd64-1.1.0 对应系统版本和源码版本
$ curl https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric/hyperledger-fabric/linux-amd64-1.1.0/hyperledger-fabric-linux-amd64-1.1.0.tar.gz | tar xz

# 启动 configtxlator
$ configtxlator start

# 或者
$ cd relase/linux-amd64/bin
$ ./configtxlator start
2017-05-31 12:57:22.499 EDT [configtxlator] main -> INFO 001 Serving HTTP requests on port: 7059
```

## 构建 `configtxgen`

在新的窗口中打开`shell`，完成下面的命令行

```sh
$ make configtxgen
```

## 生成块配置示例

建议通过调用脚本`fabric/examples/configtxupdate/bootstrap_batchsize/script.sh `来运行该示例，如下所示：

```sh
$ INTERACTIVE=true ./script.sh
```

这将输出每一步发生的事情，并允许暂停检查之前生成的配置文件。通过暂停期间执行命令：

```sh
/configtxupdate/bootstrap_batchsize/example_output$ ll
total 73
drwxrwxrwx 1 vagrant vagrant  4096 Jun  1 07:50 ./
drwxrwxrwx 1 vagrant vagrant     0 Jun  1 03:23 ../
-rwxrwxrwx 1 vagrant vagrant 30182 Jun  1 07:49 genesis_block.json*
-rwxrwxrwx 1 vagrant vagrant  9842 Jun  1 07:48 genesis_block.pb*
-rwxrwxrwx 1 vagrant vagrant 30182 Jun  1 07:50 updated_genesis_block.json*
```

都可以看到新的文件产生。或者，可以手动单独运行以下步骤。

### 手动执行脚本中的命令

---

首先为 `orderer` 系统通道生成一个生成块

```sh
$ configtxgen -outputBlock 'example_output/genesis_block.pb' -profile SampleSingleMSPSolo
2018-06-01 07:56:25.652 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-06-01 07:56:25.879 UTC [common/tools/configtxgen] doOutputBlock -> INFO 002 Generating genesis block
2018-06-01 07:56:25.880 UTC [common/tools/configtxgen] doOutputBlock -> INFO 003 Writing genesis block
```

将生成块解码为人类可读的形式

```sh
$ curl -X POST --data-binary @example_output/genesis_block.pb http://127.0.0.1:7059/protolator/decode/common.Block > example_output/genesis_block.json
```

在`JSON`编辑器中编辑`genesis_block.json`文件，或以编程方式操作它，在这里，我们使用`JSON CLI`工具`jq`。为了简单起见，我们正在编辑通道的**批量大小**这个值，因为它是单个数字字段，但可以在此处进行任何编辑，包括策略和MSP编辑。

```sh
# 设置变量，方便下面操作变量值
$ MAXBATCHSIZEPATH=".data.data[0].payload.data.config.channel_group.groups.Orderer.values.BatchSize.value.max_message_count"
# 显示批量大小属性值
$ jq "$MAXBATCHSIZEPATH" example_output/genesis_block.json
10

# 设置新值
$ jq "$MAXBATCHSIZEPATH = 20" example_output/genesis_block.json  > example_output/updated_genesis_block.json

# 查看新值
$ jq "$MAXBATCHSIZEPATH" example_output/updated_genesis_block.json
20
```

现在已经准备好将创世块重新编码为本地`proto `。

```sh
$ curl -X POST --data-binary @example_output/updated_genesis_block.json http://127.0.0.1:7059/protolator/encode/common.Block > example_output/updated_genesis_block.pb
```

`updated_genesis_block.pb`文件现在可以用作引导`orderer`系统通道的创世块。

## 构建并启动 `orderer`

```sh
$ cd /opt/gopath/src/github.com/hyperledger/fabric
$ make orderer
```

使用默认选项启动`orderer`，并设置日志级别为`debug`，包括将创建`orderer`系统通道 `testchainid` 作为临时引导程序

```sh
$ ORDERER_GENERAL_LOGLEVEL=debug orderer
```

## 构建并启动 `peer`

构建`peer`二进制程序比构建`orderer`复杂得多。

```sh
$ make peer
mkdir -p build/image/ccenv/payload
cp build/docker/gotools/bin/protoc-gen-go build/bin/chaintool build/goshim.tar.bz2 build/image/ccenv/payload
cp: cannot stat 'build/docker/gotools/bin/protoc-gen-go': No such file or directory
Makefile:288: recipe for target 'build/image/ccenv/payload' failed
make: *** [build/image/ccenv/payload] Error 1
```

构建已经错误，下面开始构建 `gotools`工具。

```sh
$ cd /opt/gopath/src/github.com/hyperledger/fabric/gotools
$ make install

$ mkdir -p ../build/docker/gotools/bin
$ cp build/gopath/bin/* ../build/docker/gotools/bin/
```

因为链码是运行在`Peer`所在的机器上，所以需要给`ChainCode`准备运行基础环境，也就是`ccenv`和`javaenv`两个镜像。在`make peer`之前我们必须先保证本地没有对应的镜像文件，如果有，那么就用`docker rmi`命令删除之前下载或者编译好的镜像。 **以上操作纯粹是为了绕开墙**，如果你本身不存在墙的问题，那么完全可以不用如此多此一举。 

接下来直接运行以下命令即可编译生成`Peer`节点的`Docker`镜像：

```sh
$ make peer
```

运行该命令后，系统先会从官网下载`chaintool`的jar包，下载地址：

```sh
https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric/hyperledger-fabric/chaintool-$(CHAINTOOL_RELEASE)/hyperledger-fabric-chaintool-$(CHAINTOOL_RELEASE).jar

# 对应的版本
https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric/hyperledger-fabric/chaintool-1.1.0/hyperledger-fabric-chaintool-1.1.0.jar
```

这个地址没有被墙，一般来说是能够正常下载下来的。随后会下载`hyperledger/fabric-baseimage:x86_64-0.3.2`这个镜像到本地，这是所有`Fabric`镜像的基础镜像文件。

下载完毕后，接下来就开始创建`fabric-ccenv`镜像。所有创建Docker镜像都是基于`Dockerfile`，我们可以在`Fabric`目录下的`images`文件夹看到所有`Fabric`镜像`build`的步骤配置。

在创建`fabric-ccenv`镜像后，接下来是创建`java`版`ChainCode`的基础镜像，也就是`fabric-javaenv`，最后才是`build peer`程序，运行成功后看到的日志大概是这样的：

```sh
Successfully built e6f02ce5f226
Successfully tagged hyperledger/fabric-javaenv:latest
docker tag hyperledger/fabric-javaenv hyperledger/fabric-javaenv:x86_64-1.1.0
build/bin/peer
CGO_CFLAGS=" " GOBIN=/opt/gopath/src/github.com/hyperledger/fabric/build/bin go install -tags "" -ldflags "-X github.com/hyperledger/fabric/common/metadata.Version=1.1.0 -X github.com/hyperledger/fabric/common/metadata.BaseVersion=0.4.6 -X github.com/hyperledger/fabric/common/metadata.BaseDockerLabel=org.hyperledger.fabric -X github.com/hyperledger/fabric/common/metadata.DockerNamespace=hyperledger -X github.com/hyperledger/fabric/common/metadata.BaseDockerNamespace=hyperledger -X github.com/hyperledger/fabric/common/metadata.Experimental=false" github.com/hyperledger/fabric/peer
Binary available as build/bin/peer
```

## 增量更新配置示例

重新配置通道的方式和上面创建或修改区块配置方式是一样的。在路径`fabric/examples/configtxupdate/reconfigure_batchsize`运行脚本 `script.sh `

```sh
$ INTERACTIVE=true ./script.sh
```

这将输出每一步发生的事情，并允许暂停检查之前生成的配置文件。通过暂停期间可以看到对应目录有新文件生成或修改。

### 手动执行脚本中的命令

---

可以按照以下步骤交互执行该过程，获取`testchainid`通道上的`peer`的区块配置

```sh
$ peer channel fetch config config_block.pb -o 127.0.0.1:7050 -c testchainid
```

将配置块发送到`configtxlator`服务进行解码：

```sh
$ curl -X POST --data-binary @config_block.pb http://127.0.0.1:7059/protolator/decode/common.Block > config_block.json
```

从区块JSON中提取配置部分：

```sh
$ jq .data.data[0].payload.data.config config_block.json > config.json
```

编辑`config.json`配置，将其保存为新的`updated_config.json`。在这里，我们将批量大小设置为30

```sh
$ jq ".channel_group.groups.Orderer.values.BatchSize.value.max_message_count = 30" config.json  > updated_config.json
```

将**原始**配置和**更新**后的配置重新编码为`proto`

```sh
# 原始配置
$ curl -X POST --data-binary @config.json http://127.0.0.1:7059/protolator/encode/common.Config > config.pb

# 修改后的配置
$ curl -X POST --data-binary @updated_config.json http://127.0.0.1:7059/protolator/encode/common.Config > updated_config.pb
```

现在，在两个配置都正确编码的情况下，将它们发送到`configtxlator`服务以计算在两者之间增量更新配置

```sh
$ curl -X POST -F original=@config.pb -F updated=@updated_config.pb http://127.0.0.1:7059/configtxlator/compute/update-from-configs -F channel=testchainid > config_update.pb
```

此时，计算出的增量更新配置文件已经完成。将使用SDK来签署和包装此消息，但为了仅使用对等节点 `cli，configtxlator`也可用于此任务。

首先，我们解码`ConfigUpdate`，以便我们可以将它作为文本进行处理。

```sh
$ curl -X POST --data-binary @config_update.pb http://127.0.0.1:7059/protolator/decode/common.ConfigUpdate > config_update.json
```

然后重新封装下修改后的配置信息

```sh
$ echo '{"payload":{"header":{"channel_header":{"channel_id":"testchainid", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' > config_update_as_envelope.json
```

最后，将其转换回完整的配置交易的`proto`形式

```sh
$ curl -X POST --data-binary @config_update_as_envelope.json http://127.0.0.1:7059/protolator/encode/common.Envelope > config_update_as_envelope.pb
```

提交配置更新交易以进行配置更新

```sh
$ peer channel update -f config_update_as_envelope.pb -c testchainid -o 127.0.0.1:7050
2018-06-01 10:13:01.758 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-06-01 10:13:01.834 UTC [channelCmd] update -> INFO 002 Successfully submitted channel update
2018-06-01 10:13:01.834 UTC [main] main -> INFO 003 Exiting.....
```

## 更新组织示例

使用`SampleDevModeSolo`配置文件选项和`debug`日志级别选项启动 `orderer`服务

```sh
$ vi github.com/hyperledger/fabric/sampleconfig/orderer.yaml

GenesisMethod: file
GenesisProfile: SampleInsecureSolo
GenesisFile: genesisblock

# 或者 设置变量
$ ORDERER_GENERAL_GENESISMETHOD=file

$ ORDERER_GENERAL_LOGLEVEL=debug ORDERER_GENERAL_GENESISMETHOD=file ORDERER_GENERAL_GENESISPROFILE=SampleDevModeSoloV1_1 orderer

# ORDERER_GENERAL_LOGLEVEL=debug ORDERER_GENERAL_GENESISMETHOD=file ORDERER_GENERAL_GENESISPROFILE=SampleSingleMSPSolo orderer
```

本示例和之前的示例完全相同，但是，不必设置批量大小，而是在应用程序级别定义新的组织。添加一个组织会稍微牵扯一点，因为我们必须先创建一个通道，然后修改通道成员集合。

进入目录`fabric/examples/configtxupdate/reconfig_membership `执行脚本`script.sh `：

```sh
$ cd fabric/examples/configtxupdate/reconfig_membership
$ INTERACTIVE=true ./script.sh
```

以交互方式运行脚本，可以出现在`example_output`目录中的生成的配置文件。

### 手动执行脚本中的命令

```sh

$ docker exec -it cli bash

# 启动 configtxlator
$ configtxlator start

$ mkdir example_output
# 由于通道已创建，这一步可以省略
# configtxgen -channelID 'mychannel' -outputCreateChannelTx 'example_output/channel_create_tx.pb' -profile TwoOrgsChannel

# 这一步也可以省略
# peer channel create -f 'example_output/channel_create_tx.pb' -c 'mychannel' -o 'orderer.example.com:7050'

# 提取配置
$ export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

$ peer channel fetch config 'example_output/channel_create_tx.pb' -c mychannel -o orderer.example.com:7050 --tls --cafile $ORDERER_CA

# 移动 通道区块配置
#$ mv 'mychannel.block' 'example_output/config_block.pb'
$ mv 'example_output/channel_create_tx.pb' 'example_output/config_block.pb'

$ curl -X POST --data-binary @example_output/config_block.pb http://127.0.0.1:7059/protolator/decode/common.Block > example_output/config_block.json

# 安装jq
$ wget -O jq https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64
$ chmod +x jq

# 提取 config 配置
$ ./jq .data.data[0].payload.data.config 'example_output/config_block.json' > 'example_output/config.json'

# 修改 config 配置中的 组织 Org1MSP，将 Org1MSP 替换为 ExampleOrg
$ ./jq '. * {"channel_group":{"groups":{"Application":{"groups":{"ExampleOrg": .channel_group.groups.Application.groups.Org1MSP}}}}}'  "example_output/config.json"  > "example_output/updated_config2.json"

$ ./jq '.channel_group.groups.Application.groups.ExampleOrg.values.MSP.value.config.name = "ExampleOrg"' "example_output/updated_config2.json" > "example_output/updated_config.json"

# 转换为 proto原始类型 
$ curl -X POST --data-binary @example_output/config.json http://127.0.0.1:7059/protolator/encode/common.Config > example_output/config.pb

# 转换为 proto 原始类型
$ curl -X POST --data-binary @example_output/updated_config.json http://127.0.0.1:7059/protolator/encode/common.Config > example_output/updated_config.pb

# 提取增量或修改的部分
$ curl -X POST -F channel=mychannel -F original=@example_output/config.pb -F updated=@example_output/updated_config.pb http://127.0.0.1:7059/configtxlator/compute/update-from-configs > example_output/config_update.pb

# 转换为 json
$ curl -X POST --data-binary @example_output/config_update.pb http://127.0.0.1:7059/protolator/decode/common.ConfigUpdate > example_output/config_update.json

# 提供写入后的完整 json
$ echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":'$(cat example_output/config_update.json)'}}}' > example_output/envelope.json
$ ./jq . example_output/envelope.json > example_output/config_update_in_envelope.json

# 转换为 proto 类型
$ curl -X POST --data-binary @example_output/config_update_in_envelope.json http://127.0.0.1:7059/protolator/encode/common.Envelope > example_output/config_update_in_envelope.pb

# 最终修改更新
$ peer channel update -f 'example_output/config_update_in_envelope.pb' -c 'mychannel' -o 'orderer.example.com:7050' --tls --cafile $ORDERER_CA

# 出现bug
Error: got unexpected status: BAD_REQUEST -- error authorizing update: error validating DeltaSet: policy for [Group]  /Channel/Application not satisfied: Failed to reach implicit threshold of 2 sub-policies, required 1 remaining
```



