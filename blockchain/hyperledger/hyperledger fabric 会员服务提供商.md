# `hyperledger fabric` 会员服务提供商

本文用于提供有关`MSP`的设置和最佳实践的详细信息。

成员服务提供者（`MSP`）是一个旨在**提供成员操作体系结构抽象的组件**。`MSP`抽象出**发布和验证证书**以及**用户身份验证**背后的所有**加密机制和协议**。`MSP`可以**定义他们自己的身份概念**，以及**管理这些身份（身份验证）和身份验证（签名生成和验证）的规则**。

`Hyperledger Fabric`区块链网络可以**由一个或多个`MSP`管理**。这提供了成员资格操作的模块化，以及跨不同成员标准和体系结构的**互操作性**。

在本文档的其余部分，详细介绍了`Hyperledger Fabric`支持的`MSP`实现的设置，并讨论了有关其使用的最佳实践。

# `MSP` 配置

要设置`MSP`的实例，需要在**每个对等方和定序方本地指定其配置**（以启用对等方和定序方签名），并在**通道上指定其配置**，以启用对等方、定序方、客户端身份标识验证以及相应的签名验证（身份验证）和所有通道成员。

首先，对于每个`MSP`，需要指定名称以便引用网络中的`MSP`（例如`msp1`，`org2`和`org3.divA`）。这是一个名称，在该名称下，代表**联合体，组织或组织部门**的`MSP`的成员资格规则将在通道中被引用。这也称为`MSP`身份标识符或`MSP ID`。每个`MSP`实例要求`MSP`是**唯一**的身份标识符。例如，在系统通道创建时将检测到**具有相同身份标识符的两个`MSP`实例，或者设置器将失败**。

在默认实现`MSP`的情况下，需要**指定一组参数以允许身份（证书）验证和签名验证**。这些参数由[`RFC5280`推导](http://www.ietf.org/rfc/rfc5280.txt)，包括：

+ 构成信任**根证书**的自签名（`X.509`）列表
+ `X.509`证书列表，代表此提供程序考虑用于证书验证的**中间**`CA`。这些证书应该由**信任根证书中的一个证书进行认证**，中间`CA`是可选参数
+ 一个`X.509`证书列表，其中包含一个**可验证证书路径**，该**证书路径恰好是信任根证书之一**，代表该`MSP`的**管理员**。这些证书的所有者**有权请求更改此`MSP`配置**（例如，根`CA`，中间`CA`）
+ 本`MSP`的**有效成员**应在其`X.509`证书中包含的组织单位列表。这是一个可选的配置参数，例如，当多个组织利用相同的信任根和中间`CA`，并为其成员保留`OU`字段时使用
+ 证书**撤销列表（`CRL`）列表**，每个列表对应于列出的（中间或根）`MSP`证书颁发机构中的一个。这是一个可选参数
+ 自签名（`X.509`）证书列表，**构成`TLS`证书**的`TLS`信任根。
+ 用于表示此提供程序考虑的**中间`TLS CA`的`X.509`证书列表**。这些证书应该由`TLS`信任根中的一个证书进行认证。中间`CA`是可选参数。

需要此`MSP`实例的**有效身份标识**才能满足以下条件：

+ 它们**采用`X.509`证书**的形式，并且具有**可验证的证书路径**，该证书路径恰好是**信任证书**的一个根
+ 它们不包括在任何`CRL`证书撤销列表中
+ 并且他们在`X.509`证书**结构的`OU`字段中**列出了`MSP`配置的**一个或多个组织单位**

有关当前`MSP`实施中身份有效性的更多信息，将读者引用到[`MSP`身份有效性规则](https://hyperledger-fabric.readthedocs.io/en/latest/msp-identity-validity-rules.html)。

除了与验证相关的参数之外，为了使`MSP`能够对其**实例化的节点进行签名或认证**，需要指定：

+ 用于节点签名的**签名密钥**（目前仅支持`ECDSA`密钥）
+ **节点的`X.509`证书**，即此`MSP`验证参数下的**有效身份标识**

值得注意的是，`MSP`身份**永不过期**。它们只能通过将它们**添加到适当的`CRL`来撤销**。此外，目前**不支持强制撤销`TLS`证书**。

# 如何生成`MSP`证书及其签名密钥？

要生成`X.509`证书以提供其`MSP`配置，应用程序可以使用[`Openssl`](https://www.openssl.org/)。系统强调在`Hyperledger Fabric`中不支持包括`RSA`密钥在内的证书。

或者，可以使用`cryptogen` 工具，操作方式可以在[入门文档中查看](https://hyperledger-fabric.readthedocs.io/en/latest/getting_started.html)。

[`Hyperledger Fabric CA`](http://hyperledger-fabric-ca.readthedocs.io/en/latest/)还可用于生成配置`MSP`所需的**密钥和证书**。

# 在对等方和定序方设置`MSP`

要设置本地`MSP`（对等或定序人），管理员应创建一个**包含六个子文件夹**和**一个文件**的文件夹（例如`$MY_PATH/mspconfig`）：

1. 文件夹`admincerts`包括每个对应于**管理员证书的`PEM`文件**
2. 文件夹`cacerts`包括每个对应于**根`CA`证书的`PEM`文件**
3. （可选）文件夹`intermediatecerts` 包含`PEM`文件，每个文件对应于**中间`CA`的证书**
4. （可选）文件`config.yaml`，用于**配置支持的组织单位和身份分类**（请参阅下面的相应部分）。
5. （可选）文件夹`crls`包括**所考虑的`CRL`证书撤销列表**
6. 文件夹`keystore`，包含带有**节点签名密钥的`PEM`文件**。强调**目前不支持`RSA`密钥**
7. 文件夹`signcerts`包含具有**节点的`X.509`证书的`PEM`文件**
8. （可选）文件夹`tlscacerts`包含`PEM`文件，**每个文件对应一个`TLS`根`CA`证书**
9. （可选）文件夹`tlsintermediatecerts`包括每个对应于**中间`TLS CA`证书的`PEM`文件**

在节点的配置文件（对等体的`core.yaml`文件和`orderer`的`orderer.yaml`）中，需要**指定`mspconfig`文件夹的路径**以及**节点`MSP`的`MSP`身份标识符**。`mspconfig`文件夹的路径应该**与`FABRIC_CFG_PATH`相关**，并且作为对等的**参数`mspConfigPath`**和`orderer`的**`LocalMSPDir`的值**提供。节点的`MSP`的身份标识符作为对等体的**参数`localMspId`**和`orderer`的**`LocalMSPID`的值**提供。可以使用**对等的`CORE`前缀**（例如`CORE_PEER_LOCALMSPID`）和**定序者的`ORDERER`前缀**（例如`ORDERER_GENERAL_LOCALMSPID`）通过**环境覆盖这些变量**。请注意，对于定序人设置，需要生成并向定序人提供系统通道的生成块。此块的`MSP`配置需求将在下一节中详细介绍。

只能手动重新配置本地`MSP`，并且**需要重新启动对等或定序者进程**。在后续版本中，官方的目标是提供**在线/动态重新配置**（即无需通过使用节点管理系统链码来停止节点）。

# 组织单位

为了配置此`MSP`的**有效成员**应在其`X.509`证书中包含的**组织单位列表**，`config.yaml`文件需要**指定组织单位身份标识符**。这是一个例子：

```yaml
OrganizationalUnitIdentifiers:
  - Certificate: "cacerts/cacert1.pem"
    OrganizationalUnitIdentifier: "commercial"
  - Certificate: "cacerts/cacert2.pem"
    OrganizationalUnitIdentifier: "administrators"
```

上面的示例声明了两个组织单位身份标识符：`commercial`和`administrators`。如果`MSP`身份**至少携带其中一个组织单位身份标识符，则该身份有效**。`Certificate`字段是**指`CA`或中间`CA`证书路径**，在该路径下，应验证具有该**特定`OU`的身份**。该路径**相对于`MSP`根文件夹，不能为空**。

# 身份分类

默认的`MSP`实现允许基于其`x509`证书的`OU`进一步将**身份分类到客户端和对等端**。如果身份**提交交易、查询同行**等，则该身份应被归类为**客户**。如果身份**认可或提交交易**，则该身份应被归类为**同等身份**。为了定义给定`MSP`的客户端和对等端，需要适当地设置`config.yaml`文件。这是一个例子：

```yaml
NodeOUs:
  Enable: true
  ClientOUIdentifier:
    Certificate: "cacerts/cacert.pem"
    OrganizationalUnitIdentifier: "client"
  PeerOUIdentifier:
    Certificate: "cacerts/cacert.pem"
    OrganizationalUnitIdentifier: "peer"
```

如上所示，`NodeOUs.Enable`设置为`true`，这将**启用身份标识分类**。然后，通过为`NodeOUs.ClientOUIdentifier（NodeOUs.PeerOUIdentifier）`键**设置以下属性来定义客户端（对等）身份标识符**：

+ `OrganizationalUnitIdentifier` 将此值设置为与客户端（对等方）的`x509`证书应包含的`OU`匹配的值。
+ `Certificate` 将此**设置为`CA`或中间`CA`**，在该`CA`下应**验证客户端（对等）身份标识**。该字段**相对于`MSP`根文件夹**。它可以为空，这意味着可以在`MSP`配置中定义的**任何`CA`下验证身份的`x509`证书**。

**启用分类后，`MSP`管理员需要成为该`MSP`的客户端**，这意味着他们的`x509`证书**需要携带身份标识客户端的`OU`**。另请注意，**身份可以是客户端也可以是对等端**。这两个分类是**相互排斥**的。如果身份**既不是客户端也不是对等方，则验证将失败**。

最后，请注意，对于升级环境，需要启用`1.1`通道功能才能使用身份标识分类。

# 通道`MSP`设置

在系统的起源时，需要指定网络中出现的**所有`MSP`的验证参数**，并将这些验证参数包含在系统通道的**生成块中**。回想一下，`MSP`验证参数**包括`MSP`身份标识符、信任证书根、中间`CA`和管理员证书、以及`OU`规范和`CRL`**。系统创建块在其设置阶段提供给定序者，并允许他们**验证通道创建请求**。如果后者包括两个具有**相同**身份标识符的`MSP`，那么`Orderers`将**拒绝**系统创建块，因此**网络的引导将失败**。

对于应用程序通道，仅**管理通道的`MSP`**的**验证组件**需要驻留在**通道的创建块**中。我们强调，**应用程序有责任确保在指示一个或多个对等方加入通道之前，将正确的`MSP`配置信息包含在通道的创世块（或最新的配置块）中**。

在`configtxgen`工具的帮助下引导通道时，可以通过**在`mspconfig`文件夹中包含`MSP`的验证参数**，并在`configtx.yaml`的相关部分中**设置该路径来配置通道`MSP`**。

通过`MSP`的一个**管理员证书的所有者，创建`config_update`对象**来实现在通道上**重新配置`MSP`**，包括与该`MSP`的`CA`相关联的**证书撤销列表**的通告。然后，管理员管理的客户端应用程序会将此**更新通知给出现此`MSP`的通道**。

# 最佳实践

在本节中，将详细介绍常见方案中`MSP`配置的最佳实践。

### **1、组织/公司与`MSP`之间的映射**

我们建议**组织和`MSP`之间存在一对一的映射**。如果选择了**不同类型**的映射，则需要考虑以下内容：

+ **一个采用各种`MSP`的组织**：这与组织的情况相对应，该组织包括由其`MSP`代表的各种部门，出于管理**独立性**原因或出于**隐私**原因。在这种情况下，对等体只能由**单个`MSP`拥有**，并且不会将具有来自其他`MSP`的身份的对等体识别为同一组织的对等体。这意味着**对等体可以通过`gossip`组织范围的数据与作为同一细分的成员的一组对等体共享，而不是与构成实际组织的全套提供者共享**。
+ **使用单个`MSP`的多个组织**：这对应于由类似成员资格架构管理的组织联盟的情况。需要知道的是，**对等体会将组织范围的消息传播到在同一`MSP`下具有身份的对等体**，而**不管它们是否属于同一个实际组织**。这是`MSP`定义和对等配置的**粒度**的限制。

### **2、一个组织有不同的部门（比如组织单位），它希望授予不同通道的访问权限**

处理这个的两种方法：

+ **定义一个`MSP`以适应所有组织成员的成员资格**：该`MSP`的配置将包括**根`CA`，中间`CA`和管理证书**的列表，以及成员身份将包括**成员所属的组织单位（`OU`）**。然后可以**定义策略以捕获特定`OU`的成员**，并且这些策略可以构成通道的**读/写策略或链码的认可策略**。这种方法的局限性在于，`gossip`同行节点会认为在其本地`MSP`下具有**成员身份的同伴作为同一组织的成员**，因此会与他们**闲聊组织范围的数据（例如他们的状态）**。
+ **定义一个`MSP`来代表每个部门**：这将涉及为每个部门**指定一组证书**，用于**根`CA`，中间`CA`和管理员证书**，这样`MSP`之间就**没有重叠**的证书路径。这意味着，例如采用**每个细分不同**的中间`CA`。这里的缺点是**管理多个`MSP`而不是一个`MSP`**，但这避免了前一种方法中存在的问题。还可以通过**利用`MSP`配置的`OU`扩展为每个部门定义一个`MSP`**。

### **3、将客户与同一组织的同行分开**

在许多情况下，**要求身份的“类型”可以从身份本身中检索**（例如，可能需要保证认可是由同伴**推导**出来的，而**不是仅仅作为定序者的客户或节点**）。对此类要求的支持有限。

允许这种**分离**的一种方法是为每个节点类型创建一个**单独的中间`CA`**：一个用于**客户端**，一个用于**对等节点或定序者**。并配置**两个不同的`MSP`**：一个用于**客户端**，一个用于**同行节点/定序者**。该组织应该访问的通道需要包括**两个`MSP`**，而认可政策将**仅利用引用同行的`MSP`**。这最终会导致**组织映射到两个`MSP`实例**，并且会对同行和客户端的交互方式产生某些影响。

由于同一组织的**所有同行仍然属于一个`MSP`**，因此`Gossip`不会受到严重影响。对等方可以将某些系统链码的执行**限制为基于本地`MSP`的策略**。例如，如果请求由其本地`MSP`的管理员签名，则对等体仅执行`joinChannel`请求，该管理员只能是客户端（最终用户应该坐在该请求的源头）。如果我们接受作为同行/定序者`MSP`成员的唯一客户将是该`MSP`的管理员，我们可以绕过这种不一致。

使用这种方法需要考虑的另一点是，**对等体基于其本地`MSP`内的请求发起者的成员资格来授权事件注册请求**。显然，由于**请求的发起者是客户端**，因此始终认为**请求发起者属于与所请求的对等体不同的`MSP`，并且对等体将拒绝该请求**。

### **4、管理员和`CA`证书**

**将`MSP`管理证书设置为与`MSP`为信任根**或**中间`CA`**考虑的任何证书**不同**非常重要。这是一种常见的（安全）实践，将**成员组件的管理职责与颁发新证书和验证现有证书分开**。

### **5、将中间`CA`列入黑名单**

如前几节所述，重新配置`MSP`是通过**重新配置机制**（本地`MSP`实例的手动重新配置，以及通道的`MSP`实例的**正确构造的`config_update`消息**）实现的。显然，有两种方法可以确保`MSP`中考虑的**中间`CA`不再考虑用于`MSP`的身份验证**：

+ 1、**重新配置`MSP`以不再在可信中间`CA`证书列表中包含该中间`CA`的证书**。对于本地配置的`MSP`，这意味着将**从`intermediatecerts`文件夹中删除此`CA`的证书**。
+ 2、重新配置`MSP`以包含**由信任根生成的`CRL`**，**该`CRL`谴责所提到的中间`CA`证书**。

在当前的`MSP`实现中，我们仅支持方法`1`，因为它**更简单**并且**不需要**将不再被认为是中间`CA`列入黑名单。

### **6、`CA`和`TLS CA`**

`MSP`身份的根`CA`和`MSP TLS`证书的根`CA`（以及相对的中间`CA`）需要在**不同的文件夹中声明**。这是为了避免不同类别的证书之间的**混淆**。**不禁止**为`MSP`身份和`TLS`证书重复使用相同的`CA`，但最佳实践建议在生产中**避免**这种情况。