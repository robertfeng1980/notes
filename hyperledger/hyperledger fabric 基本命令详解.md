# HyperLedger Fabric 基本命令详解

# `cryptogen` 基本命令

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

# `configtxgen` 基本命令

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

peer chaincode命令允许管理员在对等体上执行链式代码相关操作，例如安装，实例化，调用，打包，查询和升级链式代码。

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
# 查询已安装的cc
$ peer chaincode list --installed
Get installed chaincodes on peer:
Name: marbles, Version: 1.0, Path: github.com/hyperledger/fabric/examples/chaincode/go/marbles02, Id: 99d0c46f16339a51bb38e925e1038c0feab708fc1dc7ee883151d3bb6a870d97

$ peer chaincode list -C mychannel --installed
Get installed chaincodes on peer:
Name: marbles, Version: 1.0, Path: github.com/hyperledger/fabric/examples/chaincode/go/marbles02, Id: 99d0c46f16339a51bb38e925e1038c0feab708fc1dc7ee883151d3bb6a870d97

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

```sh
# mycc 调用 init 方法，接收两个参数 a、b，具体可以查看链码的代码
$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $CC_CA -C my_tx_channel -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"

# marbles 调用init方法，没有传递额外参数
$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $CC_CA -C $CHANNEL_NAME -n marbles -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org2MSP.member','Org1MSP.member')"
```

## `invoke` 调用

调用指定的链码

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

```sh
# a 为 b 交易转账
$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile $CC_CA  -C my_tx_channel -n mycc -c '{"Args":["invoke","a","b","10"]}'

# -c 后面具体的参数需要查看 链码的源码
$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile $CC_CA -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble1","blue","35","tom"]}'
```

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
$ peer chaincode query -C my_tx_channel -n mycc -c '{"Args":["query","a"]}'

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

```sh
$ peer chaincode package -n marbles -p /tmp/marbles_cc -i "OR ('Org2MSP.member','Org1MSP.member')"
```