# HyperLedger Fabric 基本命令详解

# `cryptogen` 命令

```sh
$ cryptogen --help
usage: cryptogen [<flags>] <command> [<args> ...]
用于生成Hyperledger Fabric密钥材料的实用程序

Commands:
  generate [<flags>] 	# 生成密钥材料   
  showtemplate			# 显示默认的配置模板   
  extend [<flags>]		# 扩展存在的网络    
```

## `generate` 生成加密文件

```sh
# 生成密钥材料
$ cryptogen generate --help
usage: cryptogen generate [<flags>]

Flags:  
  --output="crypto-config"  # 放置工件的输出目录
  --config=CONFIG           # 要使用的配置模板文件
```

下面生成一个简单的加密文件，配置模板文件 `crypto_config.yaml`，输出目录 `crypto_out`。`crypto_config.yaml`模板文件内容如下：

```yaml
OrdererOrgs:
- Name: Orderer
  Domain: hoojo.cnblogs.com
  Specs:
  - Hostname: orderer_host
PeerOrgs:
- Name: Org1
  Domain: org1.hoojo.cnblogs.com
  EnableNodeOUs: false
  Template:
    Count: 1
  Users:
    Count: 1
- Name: Org2
  Domain: org2.hoojo.cnblogs.com
  EnableNodeOUs: false
  Template:
    Count: 1
  Users:
    Count: 1
```

如果不知道模板文件格式，可以利用命令查看模板文件格式 

```sh
$ cryptogen showtemplate
```

利用命令生成配置文件，再用 `tree crypto_out -d -L 4` 命令查看`crypto_out` 输出目录5层结构如下：

```sh
$ rm -rf crypto_out/*
$ cryptogen generate --output=crypto_out --config=crypto_config.yaml
$ tree crypto_out -d -L 4
crypto_out
├── ordererOrganizations
│   └── hoojo.cnblogs.com			# --> 对应 OrdererOrgs 下的 Domain 属性值
│       ├── ca
│       ├── msp
│       │   ├── admincerts
│       │   ├── cacerts
│       │   └── tlscacerts
│       ├── orderers
│       │   └── orderer_host.hoojo.cnblogs.com	# --> 对应 OrdererOrgs 下的 Specs.Hostname 属性值
│       ├── tlsca
│       └── users
│           └── Admin@hoojo.cnblogs.com #--> @ 后面会关联上面的 Domain 属性值
└── peerOrganizations
    ├── org1.hoojo.cnblogs.com	#--> org1 关联上面的 PeerOrgs.Domain 属性值
    │   ├── ca
    │   ├── msp
    │   │   ├── admincerts
    │   │   ├── cacerts
    │   │   └── tlscacerts
    │   ├── peers
    │   │   └── peer0.org1.hoojo.cnblogs.com	# --> 关联上面org1的 PeerOrgs.Template.Count 属性值
    │   ├── tlsca
    │   └── users
    │       ├── Admin@org1.hoojo.cnblogs.com
    │       └── User1@org1.hoojo.cnblogs.com	# --> PeerOrgs.{Orgs}.Users.Count #Orgs=org1
    └── org2.hoojo.cnblogs.com	#--> org2 关联上面的 PeerOrgs.Domain 属性值
        ├── ca
        ├── msp
        │   ├── admincerts
        │   ├── cacerts
        │   └── tlscacerts
        ├── peers
        │   └── peer0.org2.hoojo.cnblogs.com
        ├── tlsca
        └── users
            ├── Admin@org2.hoojo.cnblogs.com
            └── User1@org2.hoojo.cnblogs.com
```

也可以调整配置文件，生成不同数量的`Order、Peer、User`的加密文件配置。

## `extend` 扩展现有网络

扩展网络其实就是在之前的网络的目录中生成新加入的配置加密文件数据

```sh
# 扩展存在的网络
$ cryptogen extend --help
usage: cryptogen extend [<flags>]

Flags:
  --help                   Show context-sensitive help (also try --help-long and --help-man).
  --input="crypto-config"  # 现有网络所在的输入目录
  --config=CONFIG          # 要使用的配置模板文件
```

创建一个模板配置文件 `crypto_config_extend.yaml` 内容如下：

```yaml
OrdererOrgs:
  - Name: Orderer
    Domain: hoojo.csdn.com
    Specs:
      - Hostname: orderer_host_csdn

PeerOrgs:
  - Name: Org10
    Domain: org10.hoojo.csdn.net
    EnableNodeOUs: false
    Template:
      Count: 1
    Users:
      Count: 1

  - Name: Org11
    Domain: org11.hoojo.csdn.net
    EnableNodeOUs: false
    Template:
      Count: 2
      Start: 10
      Hostname: "{{.Prefix}}.extend.{{.Index}}"
    Users:
      Count: 1
```

执行 `extend`命令生成新加入的配置的加密文件，随后查看目录发现之前的配置没有改变，并且加入新的配置文件。

```sh
cryptogen extend --input=crypto_out --config=crypto_config_extend.yaml
org10.hoojo.csdn.net
org11.hoojo.csdn.net

$ tree crypto_out -d -L 4 
```

# `configtxgen` 命令

`configtxgen`命令允许用户**创建和检查**通道交易相关的文件。生成的文件的内容由`configtx.yaml`的内容决定。

```sh
$ configtxgen --help
  -asOrg #  将配置生成作为特定组织（按名称）执行，仅包含org（可能）有权设置的写入集中的值
  -channelID # 在configtx中使用的通道ID（默认“testchainid”）
  -inspectBlock # 打印指定路径中块中包含的组态
  -inspectChannelCreateTx # 以指定的路径打印交易中包含的配置
  -outputAnchorPeersUpdate # 创建配置更新以更新锚点对等点（仅适用于默认渠道创建，仅适用于第一次更新）
  -outputBlock # 写入生成块的路径（如果设置）
  -outputCreateChannelTx # 写入通道创建configtx的路径（如果设置）
  -printOrg #  将组织的定义显示为JSON。 （可用于手动添加组织到频道）
  -profile # 用于生成的configtx.yaml配置文件。 （默认“SampleInsecureSolo”）
```

`configtxgen` 工具用于创建四个文件： **orderer 创世块**、fabric **交易通道配置**和两个**anchor 节点**交易配置（每个`Peer Org`都有一个`anchor`）。

`orderer`区块是`orderer`服务的创世块，并且**通道交易配置文件**在通道**创建时被广播给orderer**。正如名称所示，`anchor`对等节点交易指定了此通道上的每个组织的`anchor peer`。

## 配置文件

`configtxgen`工具的输出很大程度上由`configtx.yaml`的内容控制。`configtx`文件在`FABRIC_CFG_PATH`配置的目录中查找，并且必须存在该文件才能运行`configtxgen`。

可以编辑此配置文件，或者可以通过设置环境变量（如`CONFIGTX_ORDERER_ORDERERTYPE = kafka`）来覆盖单个属性。

对于许多`configtxgen`操作，**必须提供配置文件名称**。配置文件是在单个文件中表示多个类似配置的一种方式。例如，一个配置文件可能会定义一个包含3个组织的通道，而另一个配置文件可能会使用4个组织定义一个。为了在文件长度变得不堪重负的情况下实现这一点，`configtx.yaml`依赖于`anchors`和`references `的标准`YAML`特性。配置的基础部分使用类似`＆OrdererDefaults`的`anchor`点进行标记，然后**合并**到带有诸如`<<：* OrdererDefaults`的引用的配置文件中。请注意，当`configtxgen`在配置文件下运行时，环境变量覆盖不需要包含配置文件前缀，并且可以相对于配置文件的**根元素**进行**引用**。例如，不要指定`CONFIGTX_PROFILE_SAMPLEINSECURESOLO_ORDERER_ORDERERTYPE`，而只需简单地省略配置文件细节，然后使用`CONFIGTX`前缀，然后使用相对于**配置文件名称**的元素，例如`CONFIGTX_ORDERER_ORDERERTYPE`。

有关所有可能的配置选项，请参阅Fabric附带的示例`configtx.yaml`。您可以在`release artifacts tar`的`config`目录中找到此文件，或者如果从源代码构建的，则可以在`sampleconfig`文件夹下找到它。



## 示例

### 创建一个创世块

为配置文件`SampleSingleMSPSoloV1_1`的`channel` `orderer-system-channel`创建一个`genesis`块写入到`genesis_block.pb`。

```sh
$ configtxgen -outputBlock genesis_block.pb -profile SampleSingleMSPSoloV1_1 -channelID orderer-system-channel
```

生成创世块，其中 `TwoOrgsOrdererGenesis`这个参数关联配置文件 `configtx.xml`中的 `Profiles.TwoOrgsOrdererGenesis` 

```sh
$ configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/man-genesis.block
```

### 创建一个通道交易配置

为配置文件`SampleSingleMSPChannelV1_1`创建通道交易配置写入文件`create_chan_tx.pb`。

```sh
$ configtxgen -outputCreateChannelTx create_chan_tx.pb -profile SampleSingleMSPChannelV1_1 -channelID application-channel-1

$ configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/man-channel.tx -channelID tx_channel
```

### 查看区块配置

将`genesis_block.pb`的创世块的内容打印为`JSON`。

```sh
$ configtxgen -inspectBlock genesis_block.pb

$ ../../release/linux-amd64/bin/configtxgen -inspectBlock ./channel-artifacts/genesis.block
```

### 查看通道交易配置

将`create_chan_tx.pb`的通道tx的内容作为`JSON`打印到屏幕上。

```sh
$ configtxgen -inspectChannelCreateTx create_chan_tx.pb

$ ../../release/linux-amd64/bin/configtxgen -inspectChannelCreateTx ./channel-artifacts/channel.tx
```

### 查看组织定义

根据`configtx.yaml`中的`Name`参数（如`MSPDir`）构造组织定义，并将其作为`JSON`打印到屏幕上。（此输出对通道重新配置文件流程很有用，例如添加成员）。

```sh
# org参数对应的 configtx.yaml中的 Name 参数
$ configtxgen -printOrg Org1
$ ../../release/linux-amd64/bin/configtxgen -printOrg Org1MSP
$ ../../release/linux-amd64/bin/configtxgen -printOrg OrdererOrg
```

### 创建 `anchor peer`节点交易配置

将更新交易配置输出到`anchor_peer_tx.pb`，该事件根据`configtx.yaml`为配置文件`SampleSingleMSPChannelV1_1`中的定义设置组织`Org1`的`anchor`节点。

```sh
$ configtxgen -outputAnchorPeersUpdate anchor_peer_tx.pb -profile SampleSingleMSPChannelV1_1 -asOrg Org1

$ ../../release/linux-amd64/bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID txchannel -asOrg Org1MSP

$ ../../release/linux-amd64/bin/configtxgen  -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID txchannel -asOrg Org2MSP
```

# `configtxlator` 命令

`configtxlator`命令允许用户在数据结构的`protobuf`和`JSON`版本之间进行**转换**并创建**配置更新**。该命令可以启动`REST`服务器以通过`HTTP`公开其功能，或者可以直接用作命令行工具。

```sh
configtxlator --help
usage: configtxlator [<flags>] <command> [<args> ...]

用于生成Hyperledger Fabric通道配置的实用程序

Commands: 
  start [<flags>] 					  # 启动configtxlator REST服务器
  proto_encode --type=TYPE [<flags>]	# 将JSON文档转换为protobuf。   
  proto_decode --type=TYPE [<flags>]	# 将原始消息转换为JSON    
  compute_update --channel_id=CHANNEL_ID [<flags>]  # 取两个marshaled common.Config消息并计算在两者之间转换的配置更新
  version 		# 版本信息
```

其中 `proto_encode` 、`proto_decode` 的 `--type`参数取值包括`common.Block`，`common.Envelope`，`common.ConfigEnvelope`，`common.ConfigUpdateEnvelope`，`common.Config`和`common.ConfigUpdate` 。

## `configtxlator start` 启动

启动`configtxlator REST`服务器

### 语法和选项

```sh
$ ./configtxlator start --help
usage: configtxlator start [<flags>]

Start the configtxlator REST server

Flags: 
  --hostname="0.0.0.0"  # REST服务器将侦听的主机名或IP
  --port=7059           # REST服务器将侦听的端口
```

### 示例

```sh
$ ./configtxlator start

2018-05-31 03:13:21.996 UTC [configtxlator] startServer -> INFO 001 Serving HTTP requests on 0.0.0.0:7059
```

## `configtxlator proto_decode` 解码

将原始`protobuf`格式消息转换为`JSON`。其中`--type=TYPE`需要指定`Protobuf`消息类型名称 。

### 语法和选项

```sh
usage: configtxlator proto_decode --type=TYPE [<flags>]
# 将原始消息转换为JSON。

Flags: 
  --type=TYPE           # 要解码的protobuf结构的类型。 For example, 'common.Config'.
  --input=/dev/stdin    # 包含原始消息的文件。
  --output=/dev/stdout  # 将JSON文档写入的文件。
```

### 示例

将名为`fabric_block.pb`的区块解码为`JSON`并打印到`stdout`。

```sh
# 解码 protobuf 格式消息 fabric_block.pb 配置，以JSON形式输出
$ configtxlator proto_decode --input fabric_block.pb --type common.Block

# 将 protobuf 格式消息解码后 genesis.block 的JSON 直接在屏幕打印输出
$ ./configtxlator proto_decode --input ../../../examples/e2e_cli/channel-artifacts/genesis.block --type common.Block

# 将 protobuf 格式消息解码后 genesis.block 的JSON写入到文件 block.json
$ ./configtxlator proto_decode --input ../../../examples/e2e_cli/channel-artifacts/genesis.block --type common.Block --output ./block.json

#  将 protobuf 格式通道文件 channel.tx 转换为 json 并输出到屏幕
$ ./configtxlator proto_decode --input ../../../examples/e2e_cli/channel-artifacts/channel.tx --type common.Envelope
```

或者，在启动`REST`服务器之后，以下`curl`命令通过`REST API`执行相同的操作。

```sh
$ cd examples/e2e_cli/channel-artifacts
$ CONFIGTXLATOR_URL=http://127.0.0.1:7059

# http 请求 将当前目录文件 genesis.block 解码
$ curl -X POST --data-binary @genesis.block "${CONFIGTXLATOR_URL}/protolator/decode/common.Block"
# 解码后输出到 block.json 文件
$ curl -X POST --data-binary @genesis.block "${CONFIGTXLATOR_URL}/protolator/decode/common.Block" > block.json

# 将 protobuf 格式通道文件 channel.tx 转换为 json 并写入到 channel.json
$ curl -X POST --data-binary @./channel.tx "${CONFIGTXLATOR_URL}/protolator/decode/common.Envelope" > channel.json
```

## `configtxlator proto_encode` 编码 

将`JSON`文档转换为`protobuf`。其中`--type=TYPE`需要指定`Protobuf`消息类型名称 。

### 语法和选项

```sh
usage: configtxlator proto_encode --type=TYPE [<flags>]
# 将JSON文档转换为protobuf。

Flags: 
  --type=TYPE           # 要编码的protobuf结构的类型。示例： 'common.Config'.
  --input=/dev/stdin    # 包含JSON文档的文件
  --output=/dev/stdout  # 将输出写入到的文件。
```

### 示例

将`stdin`的策略的`JSON`文档转换为名为`policy.pb`的文件。

```sh
# 将JSON文件 block.json 转换为 protobuf 格式，输出到屏幕
$ ./configtxlator proto_encode --type common.Block --input block.json

# 将JSON文件 block.json 转换为 protobuf 格式， 并写入到文件 block.pb
$ ./configtxlator proto_encode --type common.Block --input block.json --output block.pb

$ ./configtxlator proto_encode --type common.Envelope --input channel.json --output channel.pb
```

或者，在启动`REST`服务器之后，以下`curl`命令通过`REST API`执行相同的操作。

```sh
$ curl -X POST --data-binary @block.json "${CONFIGTXLATOR_URL}/protolator/encode/common.Block" > block.pb

# 将当前目录下的 channel.json 转换为 protobuf 格式的 channel.pb
$ curl -X POST --data-binary @channel.json "${CONFIGTXLATOR_URL}/protolator/encode/common.Envelope" > channel.pb
```

## `configtxlator compute_update` 更新

取两个`marshaled common.Config`消息并计算配置更新哪个两者之间的过渡。

### 语法和选项

```sh
usage: configtxlator compute_update --channel_id=CHANNEL_ID [<flags>]

Takes two marshaled common.Config messages and computes the config update which transitions between the two.

Flags: 
  --original=ORIGINAL      # 原始配置消息。
  --updated=UPDATED        # 更新的配置消息.
  --channel_id=CHANNEL_ID  # 更新的通道名称.
  --output=/dev/stdout     # 将JSON文档写入的文件
```

### 示例

从`original_config.pb`和`modified_config.pb`计算配置更新并将其解码为`JSON`以标准输出。

```sh
$ configtxlator compute_update --channel_id testchan --original original_config.pb --updated modified_config.pb | configtxlator proto_decode --type common.ConfigUpdate

# REST API 形式
$ curl -X POST -F channel=testchan -F "original=@original_config.pb" -F "updated=@modified_config.pb" "${CONFIGTXLATOR_URL}/configtxlator/compute/update-from-configs" | curl -X POST --data-binary /dev/stdin "${CONFIGTXLATOR_URL}/protolator/encode/common.ConfigUpdate"
```

## 示例演示

### 增量更新区块配置信息

---

计算配置更新的增量，对于给定的两个配置（`common.config` 结构），`configtxlater`还可以比对他们的不同，计算出更新到配置时的更新增量配置（`common.ConfigUpdate`结构）。 

```sh
$ docker exec -it cli bash
$ ./configtxlator start

$ wget -O jq https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64

# 1、获取已创建的应用通道 mychannel 的配置区块
$ peer channel fetch config mychannel.block -c mychannel -o orderer.example.com:7050 --tls --cafile $ORDERER_CA

# 2、将其转化为json格式
$ curl -X POST --data-binary @mychannel.block "http://127.0.0.1:7059/protolator/decode/common.Block" > ./mychannel.json

# 3、取出 config 域（该文件中包括了 common.Config 结构相关数据）
$ ./jq ".data.data[0].payload.data.config" mychannel.json > mychannel_config.json
    
# 4、对 mychannel_config.json 文件中的配置项（如区块最大交易消息数 Ordeer.values.Batch-Size.value.max_message_count 或 调整通道中组织等）进行修改，另存为 mychannel_config_new.cfg。并将源文件和修改后的文件转成二进制文件
$ ./jq ".channel_group.groups.Orderer.values.BatchSize.value.max_message_count" mychannel_config.json
10

# Set the new batch size
$ ./jq ".channel_group.groups.Orderer.values.BatchSize.value.max_message_count = 20" mychannel_config.json  > mychannel_config_new.json

# Display the new batch size
$ ./jq ".channel_group.groups.Orderer.values.BatchSize.value.max_message_count" mychannel_config_new.json
20

# 转换 旧的通道配置
$ curl -X POST --data-binary @mychannel_config.json "http://127.0.0.1:7059/protolator/encode/common.Config" > mychannel_config.cfg
# 转换 新的通道配置
$ curl -X POST --data-binary @mychannel_config_new.json "http://127.0.0.1:7059/protolator/encode/common.Config" > mychannel_config_new.cfg

# 5、利用这两个文件，通过configtxlator提供的接口，计算出更新配置时的更新量信息，为common.ConfigUpdate结构的二进制文件。并根据二进制文件生成更新量的Json格式。
$ curl -X POST -F original=@mychannel_config.cfg -F updated=@mychannel_config_new.cfg "http://127.0.0.1:7059/configtxlator/compute/update-from-configs" -F channel=mychannel > config_update.cfg

$ curl -X POST --data-binary @config_update.cfg http://127.0.0.1:7059/protolator/decode/common.ConfigUpdate > config_update.json      
```

### 增量更新通道配置

---

通过计算更新的增量配置，可以得到 `common.ConfigUpdate` 结构的更新信息。而对通道配置进行更新时，还需要封装为`common.Envelope` 结构的配置更新交易。因此需要将 `common.ConfigUpdate` 结构数据进行补全，补全后的文件命名为`config_update_envelope.json`

```sh
# 补全命令：
$ echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}}, "data":{"config_update":'$(cat config_update.json)'}}}' > config_update_envelope.json
    
# 利用 common.Envelope 结构编码为二进制交易配置文件，利用它对应用通道进行更新。（注意：更新配置需要指定相应的权限（OrdererMSP的Admin身份）
$ curl -X POST --data-binary @config_update_envelope.json "http://127.0.0.1:7059/protolator/encode/common.Envelope" > config_update_envelope.tx
  
# CORE_PEER_LOCALMSPID=OrdererMSP 
# CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp

$ peer channel update -o orderer.example.com:7050 -f config_update_envelope.tx -c mychannel
```



## 扩展阅读

工具`configtx`和翻译器的`portmanteau`，旨在表达该工具只是在**不同的等效数据之间进行转换**。它**不会生成配置**。它**不提交或检查配置**。它不会修改配置本身，它只是在`configtx`格式的**不同视图之间提供一些双向转换**操作。

没有配置文件`configtxlator`，也没有为`REST`服务器提供任何认证或授权工具。由于`configtxlator`无法访问数据，密钥文件或其他可能被视为敏感的信息，因此服务器的所有者**不得将其暴露给其他客户端**。但是，由于用户向REST服务器发送的数据可能是保密的，因此用户应该信任服务器的管理员，运行本地实例或通过CLI进行操作。

# `peer` 命令

`peer`命令有五个不同的子命令，每个子命令都针对不同的类型的主题进行操作。例如，可以使用对等节点通道子命令将对等节点加入通道，或使用`peer chaincode`命令将智能合约代码部署到对等节点。

```sh
$ peer --help
Usage:
  peer [flags]
  peer [command]

Available Commands:
  chaincode   # 操作chaincode: install|instantiate|invoke|package|query|signpackage|upgrade|list.
  channel     # 操作channel: create|fetch|join|list|update|signconfigtx|getinfo.
  logging     # 日志级别: getlevel|setlevel|revertlevels.
  node        # 操作对等节点: start|status.
  version     # Print fabric peer version.

Flags:
      --logging-level string   # 覆盖默认的日志级别
      -v, --version            # 显示fabric peer server的版本
```

## 全局选项

```sh
--logging-level string   # 覆盖默认的日志级别
-v, --version            # 显示fabric peer server的版本
```

## 版本信息

查询当前镜像版本。使用此标志来显示有关**对等节点构建的详细版本信息**。该标志不能应用于**对等节点子命令或其选项**。

```sh
$ peer -v
peer:
 Version: 1.1.0
 Go version: go1.9.2
 OS/Arch: linux/amd64
 Experimental features: false
 Chaincode:
  Base Image Version: 0.4.6
  Base Docker Namespace: hyperledger
  Base Docker Label: org.hyperledger.fabric
  Docker Namespace: hyperledger
```

## 日志级别
设置日志级别，该选项设置启动时对等节点的日志记录级别。有六个可选的值：`debug，info，notice，warning，error、critical`。 

如果未明确指定日志记录级别，则将其从`CORE_LOGGING_LEVEL`环境变量中取出（如果已设置）。如果`CORE_LOGGING_LEVEL`未设置，则使用文件`sampleconfig/core.yaml `来确定对等节点的日志记录级别。

可以通过运行`peer logging getlevel <component-name>`来找到对等节点上特定组件的当前日志记录级别。

```sh
# peer logging getlevel <module> [flags]
$ peer logging getlevel mychannel

2018-05-29 06:53:41.830 UTC [cli/logging] getLevel -> INFO 001 Current log level for peer module 'mychannel': DEBUG

# peer [command] --logging-level ERROR
$ peer version --logging-level ERROR
```

# `peer chaincode` 命令

`peer chaincode`命令允许管理员在对等节点上执行链码相关操作，例如安装，实例化，调用，打包，查询和升级链码。

```sh
$ peer chaincode --help
Operate a chaincode: install|instantiate|invoke|package|query|signpackage|upgrade|list.

Usage:
  peer chaincode [command]

Available Commands:
  install     # 将指定的链码打包到部署规范中并将其保存在对等节点路径中
  instantiate # 将指定的链码部署到网络中
  invoke      # 调用指定的链码
  list        # 获取通道上的实例化链码或对等节点上安装的链码
  package     # 指定的链码打包
  query       # 使用指定的链码查询
  signpackage # 签署指定的chaincode包
  upgrade     # 更新升级

Flags:
      --cafile string               # 包含orderer端点的PEM编码的可信证书的文件的路径
      --certfile string             # 包含PEM编码的X509公钥的文件路径，用于与orderer端点进行相互TLS通信
      --clientauth                   # 与orderer端点通信时使用相互TLS
      --keyfile string               # 包含PEM编码的私钥的文件路径，用于与orderer端点进行相互TLS通信
  -o, --orderer string                      # orderer服务端点
      --ordererTLSHostnameOverride string   # 在验证与orderer的TLS连接时使用的主机名覆盖。
      --tls                                 # 在与orderer端点通信时使用TLS
      --transient string                    # JSON编码中参数的临时映射
```

不同的子命令针对不同的对等节点链码操作有不同的含义和用法。例如，使用`peer chaincode install` 子命令选项在对等节点方上安装链代码，或使用`peer chaincode query `子命令选项查询链码以获取对等节点账本上的值。



## `list` 链码列表

获取通道上的实例化链码或对等节点上安装的链码

### 语法和选项

```sh
Usage:
  peer chaincode list [flags]

Flags:
  -C, --channelID string   # 执行命令的通道
      --installed          # 获取已安装的链码
      --instantiated       # 获取通道上的实例化链码
```

### 示例

```sh
# 使用 --installed 标志列出安装在对等节点上的链码
# 查询已安装的cc
$ peer chaincode list --installed
Get installed chaincodes on peer:
Name: marbles, Version: 1.0, Path: github.com/hyperledger/fabric/examples/chaincode/go/marbles02, Id: 99d0c46f16339a51bb38e925e1038c0feab708fc1dc7ee883151d3bb6a870d97
# 你可以看到对方已经安装了一个名为marbles的chaincode，它的版本为1.0。

# 使用 --installed  与-C（通道ID）标志结合起来列举在通道上安装的链码。
$ peer chaincode list -C mychannel --installed
Get installed chaincodes on peer:
Name: marbles, Version: 1.0, Path: github.com/hyperledger/fabric/examples/chaincode/go/marbles02, Id: 99d0c46f16339a51bb38e925e1038c0feab708fc1dc7ee883151d3bb6a870d97

# 使用 --instantiated  与-C（通道ID）标志结合起来列举在通道上实例化的链码。
# 查询已安装/实例化的cc
$ peer chaincode list --channelID mychannel --instantiated
Get instantiated chaincodes on channel mychannel:
Name: marbles, Version: 1.0, Path: github.com/hyperledger/fabric/examples/chaincode/go/marbles02, Escc: escc, Vscc: vscc
```

## `install` 安装

将指定的链码打包到部署规范中并将其保存在对等节点路径中

### 语法和选项

```sh
Usage:
  peer chaincode install [flags]

Flags:
  -c, --ctor string      # JSON格式的链码的构造函数信息 (default "{}")
  -l, --lang string      # 链式代码写入的语言 (default "golang")
  -n, --name string      # 链码的名称
  -p, --path string      # 链码所在的路径
  -v, --version string   # 指定的chaincode的版本, 在 install/instantiate/upgrade 命令下
```

### 示例

```sh
# -o 为 chaincode 的 --orderer选项
$ peer chaincode install -o orderer.example.com:7050 -n marbles -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/marbles02

$ peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
```

## `instantiate` 实例化

将指定的链式代码部署到网络中

### 语法和选项

```sh
$ peer chaincode instantiate --help
Usage:
  peer chaincode instantiate [flags]

Flags:
  -C, --channelID string            # 执行命令的通道
      --collections-config string   # 包含chaincode集合配置的文件
  -c, --ctor string                 # JSON格式的链式代码的构造函数消息 (default "{}")
  -E, --escc string                 # 要用于此链码的认可系统（共识）链码的名称
  -l, --lang string                 # 链码使用的语言 (default "golang")
  -n, --name string                 # 链码的名称
  -P, --policy string               # 与此链式代码关联的（背书策略）认可策略
  -v, --version string              # 指定版本，在如下命令 install/instantiate/upgrade
  -V, --vscc string                 # 用于链码的身份验证系统链码的名称
```

### 示例

`end-to-end` 示例中的命令

```sh
# mycc 调用 init 方法，接收两个参数 a、b，具体可以查看链码的代码
$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $CC_CA -C mychannel -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"

# marbles 调用init方法，没有传递额外参数
$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $CC_CA -C $CHANNEL_NAME -n marbles -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org2MSP.member','Org1MSP.member')"
```

以下是一些`peer chaincode instantiate`命令的示例，它在通道`mychannel`上的版本`1.0`上实例化了名为`mycc`的链码：

+ 使用`--tls`和`--cafile`全局标志在启用`TLS`的网络中实例化链式代码：

  ```sh
  $ export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
  
  # 初始化金额 a=100，b=100，背书策略节点 Org1MSP.peer,Org2MSP.peer
  $peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C mychannel -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
  ```

+ 仅使用命令选项来实例化`TLS`**禁用**的网络中的链码：

  ```sh
  $ peer chaincode instantiate -o orderer.example.com:7050 -C mychannel -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
  ```



## `invoke` 调用

调用指定的链码。它会尝试将背书策略认可的交易提交给网络。

### 语法和选项

```sh
$ peer chaincode invoke --help
Usage:
  peer chaincode invoke [flags]

Flags:
  -C, --channelID string   # 执行命令的通道
  -c, --ctor string        # JSON格式的链式代码的构造函数消息 (default "{}")
  -n, --name string        # 链码的名称
```

### 示例

在通道`mychannel`上的版本`1.0`处调用名为`mycc`的`chaincode`，请求将`10`从变量`a`移动到变量`b`：

```sh
# a 为 b 交易转账
$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile $CC_CA  -C mychannel -n mycc -c '{"Args":["invoke","a","b","10"]}'

# -c 后面具体的参数需要查看 链码的源码
$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile $CC_CA -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble1","blue","35","tom"]}'
```

成功的响应表明交易已成功提交。然后交易将被添加到一个块中，并且最终由该通道上的每个对等节点验证是否有效。

## `query` 查询

使用指定的链码查询

### 语法和选项

```sh
$ peer chaincode query --help
Usage:
  peer chaincode query [flags]

Flags:
  -C, --channelID string   # 执行命令的通道
  -c, --ctor string        # JSON格式的链式代码的构造函数消息 (default "{}")
  -x, --hex                # 如果为true，则以十六进制形式输出查询值字节数组。 与--raw不兼容
  -n, --name string        # 链码的名称
  -r, --raw                # 如果为true，则输出查询值为原始字节，否则格式化为可打印字符串
  -t, --tid string         # 自定义ID生成算法的名称（哈希和解码）例如sha256base64
```

### 示例

```sh
# 查询a的账户数据
$ peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'

$ peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["readMarble","marble2"]}' -x
Query Result: 7b22636f6c6f72223a22726564222c22646f6354797065223a226d6172626c65222c226e616d65223a226d6172626c6532222c226f776e6572223a22746f6d222c2273697a65223a35307d
```

## `package` 打包

指定的链码打包

### 语法和选项

```sh
$ peer chaincode package --help
Usage:
  peer chaincode package [flags]

Flags:
  -s, --cc-package                  # 为所有者认可创建CC部署规范，而不是原始CC部署规范
  -c, --ctor string                 # JSON格式的链式代码的构造函数消息 (default "{}")
  -i, --instantiate-policy string   # 链码的实例化策略
  -l, --lang string                 # 链式代码写入的语言  (default "golang")
  -n, --name string                 # 链码的名称
  -p, --path string                 # 链码的路径
  -S, --sign                        # 如果为所有者认可创建CC部署规范包，也可以使用本地MSP进行签名
  -v, --version string              # 指定的chaincode的版本, 在 install/instantiate/upgrade 命令下
```

### 示例

以下是一个`peer chaincode package`命令的示例，该命令在版本`1.1`中打包名为`mycc`的`chaincode`，创建`chaincode`部署规范，使用本地`MSP`对软件包进行签名，并将其输出为`ccpack.out`：

```sh
$ peer chaincode package ccpack.out -n mycc -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -v 1.1 -s -S

$ peer chaincode package ccpack.out -n marbles -p github.com/hyperledger/fabric/examples/chaincode/go/marbles02 -v 1.0 -s -S

$ peer chaincode package -n mycc -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -v 0 -s -S -i "AND('OrgA.admin')" ccpack.out
```
## `signpackage` 签名包

签署指定的chaincode包

### 语法和选项

```sh
Usage:
  peer chaincode signpackage [src] [dest] 
```

### 示例

下面是一个`peer chaincode signpackage`命令的例子，它接受一个现有的签名包并创建一个新的签名添加本地MSP。

```sh
$ peer chaincode signpackage ccpack.out ccpack_new.out
```

## `upgrade` 升级

使用指定的链码升级现有的链码。新的链码将立即替换现有的链码。

### 语法和选项

```sh
$ peer chaincode upgrade --help
Usage:
  peer chaincode upgrade [flags]

Flags:
  -C, --channelID string   # 执行命令的通道
  -c, --ctor string        # JSON格式的链码的构造函数消息 (default "{}")
  -E, --escc string        # 要用于此链接代码的认可系统链码的名称
  -l, --lang string        # 链式代码写入的语言 (default "golang")
  -n, --name string        # 链码的名称
  -p, --path string        # 链式代码的路径
  -P, --policy string      # 与此链式代码关联的认可政策
  -v, --version string     # 指定版本，在如下命令 install/instantiate/upgrade
  -V, --vscc string        # 要用于此链接代码的验证系统链码的名称
```

### 示例

以下是`peer chaincode upgrade`命令的示例，该命令将通道`mychannel`上版本`1.0`的名为`mycc`的链码升级到`1.1`版，该版本包含一个新变量`c`：

+ 使用`--tls`和`--cafile`全局标志在启用`TLS`的情况下升级网络中的链式代码：

  ```sh
  $ export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
  
  $ peer chaincode upgrade -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C mychannel -n mycc -v 1.2 -c '{"Args":["init","a","100","b","200","c","300"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
  ```

+ 仅使用命令特定选项来升级**禁用**`TLS`的网络中的链码：

  ```sh
  $ peer chaincode upgrade -o orderer.example.com:7050 -C mychannel -n mycc -v 1.2 -c '{"Args":["init","a","100","b","200","c","300"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
  ```

# `peer channel` 命令

`channel`命令允许管理员在对等节点上执行与通道相关的操作，例如加入通道或列出对等节点加入的通道。

```sh
Operate a channel: create|fetch|join|list|update|signconfigtx|getinfo.

Usage:
  peer channel [command]

Available Commands:
  create       # 创建通道
  fetch        # 提取区块
  getinfo      # 获取指定通道的区块链信息
  join         # 将对等节点加入某个通道
  list         # 节点已加入的频道列表
  signconfigtx # 签署configtx更新
  update       # 发送configtx更新。

Flags:
      --cafile string               # 包含orderer端点的PEM编码的可信证书的文件的路径
      --certfile string             # 包含PEM编码的X509公钥的文件路径，用于与orderer端点进行相互TLS通信
      --clientauth                   # 与orderer端点通信时使用相互TLS
      --keyfile string               # 包含PEM编码的私钥的文件路径，用于与orderer端点进行相互TLS通信
  -o, --orderer string                      # orderer服务端点
      --ordererTLSHostnameOverride string   # 在验证与orderer的TLS连接时使用的主机名覆盖。
      --tls                                 # 在与orderer端点通信时使用TLS
```

## `create` 创建

创建一个通道并将创世块写入文件。

### 语法和选项

```sh
Usage:
  peer channel create [flags]

Flags:
  -c, --channelID string   # 在newChain命令的情况下，要创建的通道ID。
  						 #必须全部小写，长度少于250个字符，并且与正则表达式匹配：[a-z] [a-z0-9 .-] *
  -f, --file string        # 由configtxgen等工具生成的配置交易文件，用于提交给orderer
  -t, --timeout int        # 频道创建超时 (default 5)
  --outputBlock string     # 为通道写入生成块的路径。 (default ./<channelID>.block)
```

### 示例

创建一个由文件`./createchannel.txn`中包含的交易配置定义的样本通道`mychannel`。在`orderer.example.com:7050`上使用`orderer`。

```sh
$ peer channel create --orderer orderer.example.com:7050 -c mychannel -f ./createchannel.txn 
```

使用IP地址`orderer.example.com:7050`处的`orderer`为网络创建一个新的频道`mychannel`。创建此通道所需的交易配置更新文件为`./createchannel.txn`。等待`30`秒钟创建频道。

```sh
$ peer channel create --orderer orderer.example.com:7050 -c mychannel -f ./createchannel.txn -t 30
```

创建一个通道，使用ca证书进行验证

```sh
$ peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile $ORDERER_CA
```

可以看到通道`mychannel`已成功创建，并且**创世块 `0（零）`已添加到此通道**的区块链并返回到对等节点，并以`mychannel.block`的形式**存储在本地目录**中。

区块零通常被称为创世块，因为它提供了**通道的起始配置**。通道的所有后续更新都将作为通道区块链上的**配置块**捕获，**每个区块链取代先前的配置**。

## `fetch` 提取

获取指定的块，将其写入文件。

### 语法和选项

```sh
Usage:
  peer channel fetch <newest|oldest|config|(number)> [outputfile] [flags]

Flags:
  -c, --channelID string   # 通道id
```

### 示例

使用`newest `选项检索最近的通道块，并将其存储在文件`mychannel.block`中。

```sh
$ peer channel fetch newest mychannel.block -c mychannel --orderer orderer.example.com:7050
```

使用（`block number `）选项检索特定的程序段。在本例中为程序段号16 ，并将其存储在**默认**的程序块文件中。

```sh
$ peer channel fetch 16  -c mychannel --orderer orderer.example.com:7050
```

执行后可以看到检索到的区块的编号为16，并且该信息已写入默认文件`mychannel_16.block`。

对于配置区块文件可以使用`configtxlator`命令解码。有关解码输出的示例，请参阅此`configtxlator`命令。用户交易处理块也可以解码，但必须编写用户程序才能执行此操作。

```sh
$ peer channel fetch 0 -o orderer.example.com:7050 -c $CHANNEL_NAME

# 设置 outputfile 保存的文件名称
$ peer channel fetch 0 0_block.pb -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA

# 提取区块配置数据到 mychannel2.block
$ peer channel fetch config mychannel2.block -c mychannel -o orderer.example.com:7050 --tls --cafile $ORDERER_CA
```

## `getinfo` 获取信息

获取指定通道的区块链信息

### 语法和选项

```sh
Usage:
  peer channel getinfo [flags]

Flags:
  -c, --channelID string   # 通道Id
```

### 示例

获取本地通道`mychannel`的信息

```sh
$ peer channel getinfo -c mychannel
```

可以看到通道的最新区块是区块高度 8，还可以看到通道区块链中最新区块的加密哈希值。

## `join` 加入

将对等节点加入某个通道

### 语法和选项

```sh
Usage:
  peer channel join [flags]

Flags:
  -b, --blockpath string   # 包含生成块的文件路径
```

### 示例

将一个对等节点加入由文件`./mychannel.genesis.block`标识的区块中定义的通道。在此示例中，通道块先前由对等节点通道获取命令检索。

```sh
$ peer channel join -b ./mychannel.genesis.block
```

## `list` 列表

对等节点已加入的频道列表。

### 语法和选项

```sh
Usage:
  peer channel list [flags]
```

### 示例

列出对等体加入的通道

```sh
# 可以看到对等体已加入到通道mychannel。
$ peer channel list

Channels peers has joined:
mychannel
```

## `signconfigtx` 签名configtx 

在文件系统上提供签署的`configtx`更新文件。需要`'-f'`。

### 语法和选项

```sh
Usage:
  peer channel signconfigtx [flags]

Flags:
  -f, --file string   # 由configtxgen等工具生成的交易配置文件，用于提交给orderer
```

### 示例

签署文件`./updatechannel.txn`中定义的通道更新交易配置。该示例在命令之前和之后列出了配置交易文件。

```sh
# 通过将文件updatechannel.tx的大小从284字节增加到2180字节，可以看到对等节点已成功签署了配置事务。
$ peer channel signconfigtx -f updatechannel.tx
```

## `update` 更新

签署并将提供的`configtx`更新文件发送到通道。需要`'-f'，'-o'，'-c'`。

### 语法和选项

```sh
Usage:
  peer channel update [flags]

Flags:
  -c, --channelID string   # 通道id
  -f, --file string        # 由configtxgen等工具生成的配置交易文件，用于提交给orderer
```

### 示例

使用文件`./updatechannel.txn`中定义的配置交易更新通道`mychannel`。使用ip地址`orderer.example.com:7050`处的`orderer`将配置交易**发送到通道中的所有对等点**，以更新其通道配置的**副本**。

```sh
$ peer channel update -o orderer.example.com:7050 -c mychannel -f ./updatechannel.txn 

$ peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/${CORE_PEER_LOCALMSPID}anchors.tx

$ peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/${CORE_PEER_LOCALMSPID}anchors.tx --tls --cafile $ORDERER_CA
```

# `peer node` 命令

`peer node `命令允许启动对等节点或检查对等节点的状态。

```sh
$ peer node -h
Operate a peer node: start|status.

Usage:
  peer node [command]

Available Commands:
  start       # 启动节点
  status      # 查看节点状态
```

## `start` 启动

启动一个与网络交互的节点。

### 语法和选项

```sh
Usage:
  peer node start [flags]

Flags:
  -o, --orderer string      # orderer服务端点 (default "orderer:7050")
      --peer-chaincodedev   # 链码开发模式启动对等节点
```

### 示例

```sh
$ peer node start --peer-chaincodedev
```

**以链码开发模式启动对等节点**。通常链码容器是由**对等节点启动和维护**的。然而，在链码开发模式中，**链码是由用户构建和启动**的。在迭代开发的链式代码开发阶段，此模式非常有用。在`chaincode`教程中查看更多关于开发模式的信息。

## `status` 状态

```sh
# 查看状态
$ peer node status
status:STARTED
```

# `peer logging` 日志

`peer logging `子命令允许动态查看和配置对等节点的日志级别。

```sh
$ peer logging --help
Usage:
  peer logging [command]

Available Commands:
  getlevel     # 返回请求的模块日志记录的级别。
  revertlevels # 将日志记录级别恢复到对等节点启动时的级别
  setlevel     # 为所有与正则表达式匹配的模块设置日志级别。
```

## `getlevel` 获取

返回请求的模块日志的级别。注意：模块名称应该与日志中显示的名称完全匹配。

```sh
# peer logging getlevel <module> [flags]

$ peer logging getlevel mychannel
2018-05-30 08:49:52.799 UTC [cli/logging] getLevel -> INFO 001 Current log level for peer module 'mychannel': DEBUG

$ peer logging getlevel msp
2018-05-30 08:52:26.772 UTC [cli/logging] getLevel -> INFO 001 Current log level for peer module 'msp': WARNING

$ peer logging getlevel peer
2018-05-30 08:54:55.314 UTC [cli/logging] getLevel -> INFO 001 Current log level for peer module 'peer': DEBUG
```

## `revertlevels` 恢复

将日志记录级别恢复到对等节点启动时的级别

```sh
# 要将日志级别恢复为启动值：
# peer logging revertlevels [flags]
$ peer logging revertlevels
```

## `setlevel` 设置

为所有与正则表达式匹配的模块设置日志级别。

```sh
# peer logging setlevel <module regular expression> <log level> [flags]

# 为匹配正则表达式对等模块的日志级别设置日志级别 warning
$ peer logging setlevel peer warning

# 将与正则表达式 ^gossip 匹配的模块（即形式为gossip/<submodule>的所有日志记录子模块）的日志级别设置为日志级别ERROR
$ peer logging setlevel ^gossip error
```

# `orderer` 命令

`Hyperledger Fabric Orderer` 服务节点操作

```sh
$ ./orderer --help
usage: orderer [<flags>] <command> [<args> ...]
Hyperledger Fabric 服务节点操作

Commands: 
  start*		# 启动 orderer 节点
  benchmark		# 用 benchmark 模式运行orderer节点    
```



`start` 启动

```sh
$ ./orderer start
2018-05-31 09:52:27.338 UTC [orderer/common/server] prettyPrintStruct -> INFO 001 Orderer config values:
        General.LedgerType = "file"
        General.ListenAddress = "127.0.0.1"
        General.ListenPort = 7050
        General.TLS.Enabled = false
        General.TLS.PrivateKey = "/opt/gopath/src/github.com/hyperledger/fabric/sampleconfig/tls/server.key"
        General.TLS.Certificate = "/opt/gopath/src/github.com/hyperledger/fabric/sampleconfig/tls/server.crt"
        General.TLS.RootCAs = [/opt/gopath/src/github.com/hyperledger/fabric/sampleconfig/tls/ca.crt]
        General.TLS.ClientAuthRequired = false
        General.TLS.ClientRootCAs = []
        General.Keepalive.ServerMinInterval = 1m0s
        General.Keepalive.ServerInterval = 2h0m0s
        General.Keepalive.ServerTimeout = 20s
        General.GenesisMethod = "provisional"
        General.GenesisProfile = "SampleInsecureSolo"
        General.SystemChannel = "testchainid"
        General.GenesisFile = "/opt/gopath/src/github.com/hyperledger/fabric/sampleconfig/genesisblock"
        General.Profile.Enabled = false
        General.Profile.Address = "0.0.0.0:6060"
        General.LogLevel = "info"
        General.LogFormat = "%{color}%{time:2006-01-02 15:04:05.000 MST} [%{module}] %{shortfunc} -> %{level:.4s} %{id:03x}%{color:reset} %{message}"
        General.LocalMSPDir = "/opt/gopath/src/github.com/hyperledger/fabric/sampleconfig/msp"
        General.LocalMSPID = "DEFAULT"
        General.BCCSP.ProviderName = "SW"
        General.BCCSP.SwOpts.SecLevel = 256
        General.BCCSP.SwOpts.HashFamily = "SHA2"
        General.BCCSP.SwOpts.Ephemeral = false
        General.BCCSP.SwOpts.FileKeystore.KeyStorePath = "/opt/gopath/src/github.com/hyperledger/fabric/sampleconfig/msp/keystore"
        General.BCCSP.SwOpts.DummyKeystore =
        General.BCCSP.PluginOpts =
        General.Authentication.TimeWindow = 15m0s
        FileLedger.Location = "/var/hyperledger/production/orderer"
        FileLedger.Prefix = "hyperledger-fabric-ordererledger"
        RAMLedger.HistorySize = 1000
        Kafka.Retry.ShortInterval = 5s
        Kafka.Retry.ShortTotal = 10m0s
        Kafka.Retry.LongInterval = 5m0s
        Kafka.Retry.LongTotal = 12h0m0s
        Kafka.Retry.NetworkTimeouts.DialTimeout = 10s
        Kafka.Retry.NetworkTimeouts.ReadTimeout = 10s
        Kafka.Retry.NetworkTimeouts.WriteTimeout = 10s
        Kafka.Retry.Metadata.RetryMax = 3
        Kafka.Retry.Metadata.RetryBackoff = 250ms
        Kafka.Retry.Producer.RetryMax = 3
        Kafka.Retry.Producer.RetryBackoff = 100ms
        Kafka.Retry.Consumer.RetryBackoff = 2s
        Kafka.Verbose = false
        Kafka.Version = 0.10.2.0
        Kafka.TLS.Enabled = false
        Kafka.TLS.PrivateKey = ""
        Kafka.TLS.Certificate = ""
        Kafka.TLS.RootCAs = []
        Kafka.TLS.ClientAuthRequired = false
        Kafka.TLS.ClientRootCAs = []
        Debug.BroadcastTraceDir = ""
        Debug.DeliverTraceDir = ""
```

`benchmark` 模式

```sh
$ ./orderer benchmark
```

启动后可以看到一些默认的配置信息。也可以设置日志级别

```sh
$ ORDERER_GENERAL_LOGLEVEL=debug orderer
```

# `fabric-ca-client`  客户端命令

`fabric-ca-client` 命令允许管理身份（包括属性管理）和证书（包括续订和撤销）。

```sh
Hyperledger Fabric Certificate Authority Client

Usage:
  fabric-ca-client [command]

Available Commands:
  affiliation Manage affiliations
  enroll      Enroll an identity
  gencrl      Generate a CRL
  gencsr      Generate a CSR
  getcacert   Get CA certificate chain
  identity    Manage identities
  reenroll    Reenroll an identity
  register    Register an identity
  revoke      Revoke an identity
  version     Prints Fabric CA Client version

Flags:
      --caname string                  Name of CA
      --csr.cn string                  The common name field of the certificate signing request
      --csr.hosts stringSlice          A list of space-separated host names in a certificate signing request
      --csr.names stringSlice          A list of comma-separated CSR names of the form <name>=<value> (e.g. C=CA,O=Org1)
      --csr.serialnumber string        The serial number in a certificate signing request
  -d, --debug                          Enable debug level logging
      --enrollment.attrs stringSlice   A list of comma-separated attribute requests of the form <name>[:opt] (e.g. foo,bar:opt)
      --enrollment.label string        Label to use in HSM operations
      --enrollment.profile string      Name of the signing profile to use in issuing the certificate
  -H, --home string                    Client's home directory (default "/home/vagrant/.fabric-ca-client")
      --id.affiliation string          The identity's affiliation
      --id.attrs stringSlice           A list of comma-separated attributes of the form <name>=<value> (e.g. foo=foo1,bar=bar1)
      --id.maxenrollments int          The maximum number of times the secret can be reused to enroll (default CA's Max Enrollment)
      --id.name string                 Unique name of the identity
      --id.secret string               The enrollment secret for the identity being registered
      --id.type string                 Type of identity being registered (e.g. 'peer, app, user') (default "client")
  -M, --mspdir string                  Membership Service Provider directory (default "msp")
  -m, --myhost string                  Hostname to include in the certificate signing request during enrollment (default "ubuntu-xenial")
  -a, --revoke.aki string              AKI (Authority Key Identifier) of the certificate to be revoked
  -e, --revoke.name string             Identity whose certificates should be revoked
  -r, --revoke.reason string           Reason for revocation
  -s, --revoke.serial string           Serial number of the certificate to be revoked
      --tls.certfiles stringSlice      A list of comma-separated PEM-encoded trusted certificate files (e.g. root1.pem,root2.pem)
      --tls.client.certfile string     PEM-encoded certificate file when mutual authenticate is enabled
      --tls.client.keyfile string      PEM-encoded key file when mutual authentication is enabled
  -u, --url string                     URL of fabric-ca-server (default "http://localhost:7054")
```



# `fabric-ca-server` 服务端命令

`fabric-ca-server` 命令允许初始化和启动一个服务器进程，该进程可以托管一个或多个证书颁发机构。

```sh
Hyperledger Fabric Certificate Authority Server

Usage:
  fabric-ca-server [command]

Available Commands:
  init        Initialize the fabric-ca server
  start       Start the fabric-ca server
  version     Prints Fabric CA Server version

Flags:
      --address string                            Listening address of fabric-ca-server (default "0.0.0.0")
  -b, --boot string                               The user:pass for bootstrap admin which is required to build default config file
      --ca.certfile string                        PEM-encoded CA certificate file (default "ca-cert.pem")
      --ca.chainfile string                       PEM-encoded CA chain file (default "ca-chain.pem")
      --ca.keyfile string                         PEM-encoded CA key file
  -n, --ca.name string                            Certificate Authority name
      --cacount int                               Number of non-default CA instances
      --cafiles stringSlice                       A list of comma-separated CA configuration files
      --cfg.affiliations.allowremove              Enables removal of affiliations dynamically
      --cfg.identities.allowremove                Enables removal of identities dynamically
      --crl.expiry duration                       Expiration for the CRL generated by the gencrl request (default 24h0m0s)
      --crlsizelimit int                          Size limit of an acceptable CRL in bytes (default 512000)
      --csr.cn string                             The common name field of the certificate signing request to a parent fabric-ca-server
      --csr.hosts stringSlice                     A list of space-separated host names in a certificate signing request to a parent fabric-ca-server
      --csr.serialnumber string                   The serial number in a certificate signing request to a parent fabric-ca-server
      --db.datasource string                      Data source which is database specific (default "fabric-ca-server.db")
      --db.tls.certfiles stringSlice              A list of comma-separated PEM-encoded trusted certificate files (e.g. root1.pem,root2.pem)
      --db.tls.client.certfile string             PEM-encoded certificate file when mutual authenticate is enabled
      --db.tls.client.keyfile string              PEM-encoded key file when mutual authentication is enabled
      --db.type string                            Type of database; one of: sqlite3, postgres, mysql (default "sqlite3")
  -d, --debug                                     Enable debug level logging
  -H, --home string                               Server's home directory (default "/etc/hyperledger/fabric-ca")
      --intermediate.enrollment.label string      Label to use in HSM operations
      --intermediate.enrollment.profile string    Name of the signing profile to use in issuing the certificate
      --intermediate.parentserver.caname string   Name of the CA to connect to on fabric-ca-server
  -u, --intermediate.parentserver.url string      URL of the parent fabric-ca-server (e.g. http://<username>:<password>@<address>:<port)
      --intermediate.tls.certfiles stringSlice    A list of comma-separated PEM-encoded trusted certificate files (e.g. root1.pem,root2.pem)
      --intermediate.tls.client.certfile string   PEM-encoded certificate file when mutual authenticate is enabled
      --intermediate.tls.client.keyfile string    PEM-encoded key file when mutual authentication is enabled
      --ldap.attribute.names stringSlice          The names of LDAP attributes to request on an LDAP search
      --ldap.enabled                              Enable the LDAP client for authentication and attributes
      --ldap.groupfilter string                   The LDAP group filter for a single affiliation group (default "(memberUid=%s)")
      --ldap.tls.certfiles stringSlice            A list of comma-separated PEM-encoded trusted certificate files (e.g. root1.pem,root2.pem)
      --ldap.tls.client.certfile string           PEM-encoded certificate file when mutual authenticate is enabled
      --ldap.tls.client.keyfile string            PEM-encoded key file when mutual authentication is enabled
      --ldap.url string                           LDAP client URL of form ldap://adminDN:adminPassword@host[:port]/base
      --ldap.userfilter string                    The LDAP user filter to use when searching for users (default "(uid=%s)")
  -p, --port int                                  Listening port of fabric-ca-server (default 7054)
      --registry.maxenrollments int               Maximum number of enrollments; valid if LDAP not enabled (default -1)
      --tls.certfile string                       PEM-encoded TLS certificate file for server's listening port (default "tls-cert.pem")
      --tls.clientauth.certfiles stringSlice      A list of comma-separated PEM-encoded trusted certificate files (e.g. root1.pem,root2.pem)
      --tls.clientauth.type string                Policy the server will follow for TLS Client Authentication. (default "noclientcert")
      --tls.enabled                               Enable TLS on the listening port
      --tls.keyfile string                        PEM-encoded TLS key for server's listening port
```

# `Service Discovery CLI` 服务发现命令

发现服务具有自己的命令行界面（`CLI`），该界面使用`YAML`配置文件来保存证书和私钥路径等属性以及`MSP ID`。

`discover`命令具有以下子命令：

- `saveConfig`
- `peers`
- `config`
- `endorsers`

命令的用法如下所示：

```sh
# discover --help
usage: discover [<flags>] <command> [<args> ...]

用于fabric发现服务的命令行客户端

Flags:
  --help                   Show context-sensitive help (also try --help-long and --help-man).
  --configFile=CONFIGFILE  # 指定要从中加载配置的配置文件
  --peerTLSCA=PEERTLSCA    # 设置验证TLS对等方证书的TLS CA证书文件路径
  --tlsCert=TLSCERT        # （可选）设置对等体强制执行客户端身份验证时使用的客户端TLS证书文件路径
  --tlsKey=TLSKEY          # （可选）设置对等体强制执行客户端身份验证时使用的客户端TLS密钥文件路径
  --userKey=USERKEY        # 设置用户的密钥文件路径，该路径用于对发送给对等方的消息进行签名
  --userCert=USERCERT      # 设置用户的证书文件路径，该路径用于验证发送给对等方的消息
  --MSP=MSP                # 设置用户的MSP ID，表示颁发其用户证书的CA.

Commands:
  help [<command>...]	Show help.
  peers [<flags>]		# 发现 peers
  config [<flags>]		# 发现通道配置
  endorsers [<flags>]	# 发现链码背书参与者
  saveConfig	# 将由flags传递的配置保存到指定的文件中 --configFile
```

## 切换到`CLI`容器

`discover` 命令需要在客户端容器中进行运行，执行命令如下：

```sh
$ docker exec -it cli bash

# 切换到其中一个user目录
$ cd crypto/peerOrganizations/org1.example.com/users/User1@org1.example.com

# 修改命令提示符
$ PS1='[\[\033[0;32m\]\H@\u\[\033[0m\] \[\033[0;33m\]\w\[\033[0m\]]\r\n\$ '
```

## 配置外部端点

目前，要查看**服务发现中的对等体**，需要为它们配置`EXTERNAL_ENDPOINT`。否则，`Fabric`假定**不应该透露**对等体。要定义这些端点，需要在对等体的`core.yaml`中指定它们，将下面的示例端点替换为你的端点。

```sh
CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.example.com:7051
```

## 持久配置

要持久保持配置，应通过标志`--configFile`以及`saveConfig`命令提供配置文件名：

```sh
$ cd fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/User1@org1.example.com

$ discover --configFile conf.yaml --peerTLSCA tls/ca.crt --userKey msp/keystore/ea4f6a38ac7057b6fa9502c2f5f39f182e320f71f667749100fe7dd94c23ce43_sk --userCert msp/signcerts/User1\@org1.example.com-cert.pem  --MSP Org1MSP saveConfig
```

通过执行上述命令，将创建配置文件：

```yaml
$ cat conf.yaml
version: 0
tlsconfig:
  certpath: ""
  keypath: ""
  peercacertpath: /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/User1@org1.example.com/tls/ca.crt
  timeout: 0s
signerconfig:
  mspid: Org1MSP
  identitypath: /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/signcerts/User1@org1.example.com-cert.pem
  keypath: /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/keystore/ea4f6a38ac7057b6fa9502c2f5f39f182e320f71f667749100fe7dd94c23ce43_sk
```

当对等体在**启用`TLS`**的情况下运行时，对等体上的发现服务要求**客户端使用相互`TLS`连接**到它，这意味着它需要提供`TLS`证书。**默认情况下，对等体配置为请求（但不验证）客户端`TLS`证书**，因此不需要提供`TLS`证书（除非**对等体的`tls.clientAuthRequired`设置为`true`**）。

当发现`CLI`的配置文件**具有`peercacertpath`的证书路径**，但未按上述配置`certpath`和`keypath`时，发现`CLI`会**生成自签名`TLS`证书**并使用它来**连接到对等方**。

如果**未配置`peercacertpath`，则发现`CLI`将在没有`TLS`的情况下进行连接**，因此**不建议这样做**，因为**信息是通过纯文本发送的，未加密**。

## 查询发现服务

`discovery CLI`充当**发现客户端**，需要**针对对等体执行**。这是通过**指定`--server`标志**来完成的。此外，**查询是通道范围的，因此必须使用`--channel`标志**。**唯一不需要通道**的查询是**本地成员身份对等查询**，默认情况下只能由被查询的对等方的**管理员**使用。

`discover CLI`支持所有服务器端查询：

+ **对等成员查询**
+ **配置查询**
+ **背书人查询**

回顾一下，看看它们应该如何被调用和解析：

### 同行会员查询

```sh
$ discover --configFile conf.yaml peers --channel mychannel  --server peer0.org1.example.com:7051

[
    {
        "MSPID": "Org2MSP",
        "LedgerHeight": 5,
        "Endpoint": "peer0.org2.example.com:7051",
        "Identity": "-----BEGIN CERTIFICATE-----\nMIICKTCCAc+gAwIBAgIRANK4WBck5gKuzTxVQIwhYMUwCgYIKoZIzj0EAwIwczEL\nMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG\ncmFuY2lzY28xGTAXBgNVBAoTEG9yZzIuZXhhbXBsZS5jb20xHDAaBgNVBAMTE2Nh\nLm9yZzIuZXhhbXBsZS5jb20wHhcNMTgwNjE3MTM0NTIxWhcNMjgwNjE0MTM0NTIx\nWjBqMQswCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMN\nU2FuIEZyYW5jaXNjbzENMAsGA1UECxMEcGVlcjEfMB0GA1UEAxMWcGVlcjAub3Jn\nMi5leGFtcGxlLmNvbTBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABJa0gkMRqJCi\nzmx+L9xy/ecJNvdAV2zmSx5Sf2qospVAH1MYCHyudDEvkiRuBPgmCdOdwJsE0g+h\nz0nZdKq6/X+jTTBLMA4GA1UdDwEB/wQEAwIHgDAMBgNVHRMBAf8EAjAAMCsGA1Ud\nIwQkMCKAIFZMuZfUtY6n2iyxaVr3rl+x5lU0CdG9x7KAeYydQGTMMAoGCCqGSM49\nBAMCA0gAMEUCIQC0M9/LJ7j3I9NEPQ/B1BpnJP+UNPnGO2peVrM/mJ1nVgIgS1ZA\nA1tsxuDyllaQuHx2P+P9NDFdjXx5T08lZhxuWYM=\n-----END CERTIFICATE-----\n",
        "Chaincodes": [
            "mycc"
        ]
    },
    {
        "MSPID": "Org2MSP",
        "LedgerHeight": 5,
        "Endpoint": "peer1.org2.example.com:7051",
        "Identity": "-----BEGIN CERTIFICATE-----\nMIICKDCCAc+gAwIBAgIRALnNJzplCrYy4Y8CjZtqL7AwCgYIKoZIzj0EAwIwczEL\nMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG\ncmFuY2lzY28xGTAXBgNVBAoTEG9yZzIuZXhhbXBsZS5jb20xHDAaBgNVBAMTE2Nh\nLm9yZzIuZXhhbXBsZS5jb20wHhcNMTgwNjE3MTM0NTIxWhcNMjgwNjE0MTM0NTIx\nWjBqMQswCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMN\nU2FuIEZyYW5jaXNjbzENMAsGA1UECxMEcGVlcjEfMB0GA1UEAxMWcGVlcjEub3Jn\nMi5leGFtcGxlLmNvbTBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABNDopAkHlDdu\nq10HEkdxvdpkbs7EJyqv1clvCt/YMn1hS6sM+bFDgkJKalG7s9Hg3URF0aGpy51R\nU+4F9Muo+XajTTBLMA4GA1UdDwEB/wQEAwIHgDAMBgNVHRMBAf8EAjAAMCsGA1Ud\nIwQkMCKAIFZMuZfUtY6n2iyxaVr3rl+x5lU0CdG9x7KAeYydQGTMMAoGCCqGSM49\nBAMCA0cAMEQCIAR4fBmIBKW2jp0HbbabVepNtl1c7+6++riIrEBnoyIVAiBBvWmI\nyG02c5hu4wPAuVQMB7AU6tGSeYaWSAAo/ExunQ==\n-----END CERTIFICATE-----\n",
        "Chaincodes": [
            "mycc"
        ]
    },
    {
        "MSPID": "Org1MSP",
        "LedgerHeight": 5,
        "Endpoint": "peer0.org1.example.com:7051",
        "Identity": "-----BEGIN CERTIFICATE-----\nMIICKDCCAc6gAwIBAgIQP18LeXtEXGoN8pTqzXTHZTAKBggqhkjOPQQDAjBzMQsw\nCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMNU2FuIEZy\nYW5jaXNjbzEZMBcGA1UEChMQb3JnMS5leGFtcGxlLmNvbTEcMBoGA1UEAxMTY2Eu\nb3JnMS5leGFtcGxlLmNvbTAeFw0xODA2MTcxMzQ1MjFaFw0yODA2MTQxMzQ1MjFa\nMGoxCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1T\nYW4gRnJhbmNpc2NvMQ0wCwYDVQQLEwRwZWVyMR8wHQYDVQQDExZwZWVyMC5vcmcx\nLmV4YW1wbGUuY29tMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEKeC/1Rg/ynSk\nNNItaMlaCDZOaQvxJEl6o3fqx1PVFlfXE4NarY3OO1N3YZI41hWWoXksSwJu/35S\nM7wMEzw+3KNNMEswDgYDVR0PAQH/BAQDAgeAMAwGA1UdEwEB/wQCMAAwKwYDVR0j\nBCQwIoAgcecTOxTes6rfgyxHH6KIW7hsRAw2bhP9ikCHkvtv/RcwCgYIKoZIzj0E\nAwIDSAAwRQIhAKiJEv79XBmr8gGY6kHrGL0L3sq95E7IsCYzYdAQHj+DAiBPcBTg\nRuA0//Kq+3aHJ2T0KpKHqD3FfhZZolKDkcrkwQ==\n-----END CERTIFICATE-----\n",
        "Chaincodes": [
            "mycc"
        ]
    },
    {
        "MSPID": "Org1MSP",
        "LedgerHeight": 5,
        "Endpoint": "peer1.org1.example.com:7051",
        "Identity": "-----BEGIN CERTIFICATE-----\nMIICJzCCAc6gAwIBAgIQO7zMEHlMfRhnP6Xt65jwtDAKBggqhkjOPQQDAjBzMQsw\nCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMNU2FuIEZy\nYW5jaXNjbzEZMBcGA1UEChMQb3JnMS5leGFtcGxlLmNvbTEcMBoGA1UEAxMTY2Eu\nb3JnMS5leGFtcGxlLmNvbTAeFw0xODA2MTcxMzQ1MjFaFw0yODA2MTQxMzQ1MjFa\nMGoxCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1T\nYW4gRnJhbmNpc2NvMQ0wCwYDVQQLEwRwZWVyMR8wHQYDVQQDExZwZWVyMS5vcmcx\nLmV4YW1wbGUuY29tMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEoII9k8db/Q2g\nRHw5rk3SYw+OMFw9jNbsJJyC5ttJRvc12Dn7lQ8ZR9hW1vLQ3NtqO/couccDJcHg\nt47iHBNadaNNMEswDgYDVR0PAQH/BAQDAgeAMAwGA1UdEwEB/wQCMAAwKwYDVR0j\nBCQwIoAgcecTOxTes6rfgyxHH6KIW7hsRAw2bhP9ikCHkvtv/RcwCgYIKoZIzj0E\nAwIDRwAwRAIgGHGtRVxcFVeMQr9yRlebs23OXEECNo6hNqd/4ChLwwoCIBFKFd6t\nlL5BVzVMGQyXWcZGrjFgl4+fDrwjmMe+jAfa\n-----END CERTIFICATE-----\n",
        "Chaincodes": null
    }
]
```

如图所示，此命令输出一个`JSON`，其中包含有关对等查询**所拥有的通道中所有对等方的成员资格**信息。返回的标识是对等方的注册证书，可以使用`jq`和`openssl`的组合进行解析：

```sh
$ discover --configFile conf.yaml peers --channel mychannel  --server peer0.org1.example.com:7051  | jq .[0].Identity | sed "s/\\\n/\n/g" | sed "s/\"//g"  | openssl x509 -text -noout

Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            55:e9:3f:97:94:d5:74:db:e2:d6:99:3c:01:24:be:bf
    Signature Algorithm: ecdsa-with-SHA256
        Issuer: C=US, ST=California, L=San Francisco, O=org2.example.com, CN=ca.org2.example.com
        Validity
            Not Before: Jun  9 11:58:28 2018 GMT
            Not After : Jun  6 11:58:28 2028 GMT
        Subject: C=US, ST=California, L=San Francisco, OU=peer, CN=peer0.org2.example.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:f5:69:7a:11:65:d9:85:96:65:b7:b7:1b:08:77:
                    43:de:cb:ad:3a:79:ec:cc:2a:bc:d7:93:68:ae:92:
                    1c:4b:d8:32:47:d6:3d:72:32:f1:f1:fb:26:e4:69:
                    c2:eb:c9:45:69:99:78:d7:68:a9:77:09:88:c6:53:
                    01:2a:c1:f8:c0
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Authority Key Identifier:
                keyid:8E:58:82:C9:0A:11:10:A9:0B:93:03:EE:A0:54:42:F4:A3:EF:11:4C:82:B6:F9:CE:10:A2:1E:24:AB:13:82:A0

    Signature Algorithm: ecdsa-with-SHA256
         30:44:02:20:29:3f:55:2b:9f:7b:99:b2:cb:06:ca:15:3f:93:
         a1:3d:65:5c:7b:79:a1:7a:d1:94:50:f0:cd:db:ea:61:81:7a:
         02:20:3b:40:5b:60:51:3c:f8:0f:9b:fc:ae:fc:21:fd:c8:36:
         a3:18:39:58:20:72:3d:1a:43:74:30:f3:56:01:aa:26
```



