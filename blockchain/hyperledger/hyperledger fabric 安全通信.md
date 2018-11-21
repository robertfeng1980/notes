# `hyperledger fabric` 安全通信

`Fabric`支持**使用`TLS`在节点之间进行安全通信**。`TLS`通信可以使用**单向**（仅服务器）和**双向**（服务器和客户端）身份验证。

# 为对等节点配置`TLS`

**对等节点既是`TLS`服务器又是`TLS`客户端**。它是前者，当另一个对等节点，应用程序或`CLI`与其建立连接时，后者在与另一个对等节点或定序者建立连接时。

要在对等节点上启用`TLS`，请**设置以下对等配置属性**：

+ `peer.tls.enabled` = `true`
+ `peer.tls.cert.file` 包含`TLS`**服务器证书**的文件的完全限定路径
+ `peer.tls.key.file` 包含`TLS`**服务器私钥**的文件的完全限定路径
+ `peer.tls.rootcert.file` 包含颁发`TLS`服务器证书的**证书颁发机构（`CA`）的证书链**的文件的完全限定路径

默认情况下，在**对等节点上启用`TLS`时，将关闭`TLS`客户端身份验证**。这意味着**对等节点在`TLS`握手期间不会验证客户端**（另一个对等节点，应用程序或`CLI`）的证书。要在对等节点上**启用`TLS`客户端身份验证**，请将**对等配置属性`peer.tls.clientAuthRequired`设置为`true`**，并将`peer.tls.clientRootCAs.files`属性设置为包含`CA`证书链的`CA`链文件为贵组织的客户颁发`TLS`证书。

默认情况下，**对等节点**在充当`TLS`服务器和客户端时将**使用相同的证书和私钥对**。要为客户端**使用不同**的证书和私钥对，请将`peer.tls.clientCert.file`和`peer.tls.clientKey.file`配置属性**分别设置为客户端证书和密钥文件的完全限定路径**。

通过设置以下**环境变量**，也可以启用具有**客户端身份验证的`TLS`**：

- `CORE_PEER_TLS_ENABLED` = `true`

- `CORE_PEER_TLS_CERT_FILE` **服务器证书**的完全限定路径
- `CORE_PEER_TLS_KEY_FILE` **服务器私钥**的完全限定路径
- `CORE_PEER_TLS_ROOTCERT_FILE` **CA链文件**的完全限定路径
- `CORE_PEER_TLS_CLIENTAUTHREQUIRED` = `true`
- `CORE_PEER_TLS_CLIENTROOTCAS_FILES` **CA链文件**的完全限定路径
- `CORE_PEER_TLS_CLIENTCERT_FILE` **客户端证书**的完全限定路径
- `CORE_PEER_TLS_CLIENTKEY_FILE` **客户端密钥**的完全限定路径

在**对等节点上启用客户端身份验证**时，客户端需要**在`TLS`握手期间发送其证书**。如果客户端**未发送**其证书，则**握手将失败**，并且**对等方将关闭连接**。

当对等体**加入通道**时，从通道的**配置块**读取通道成员的**根CA证书链**，并将其**添加到`TLS`客户端和服务器根`CA`数据结构中**。因此，点对点通信，点对点定序者通信**应该无缝地工作**。

# 为`orderer`节点配置`TLS`

要在定序者节点上启用`TLS`，请设置以下**定序者配置属性**：

- `General.TLS.Enabled` = `true`
- `General.TLS.PrivateKey` 包含**服务器私钥**的文件的完全限定路径
- `General.TLS.Certificate`  包含**服务器证书**的文件的完全限定路径
- `General.TLS.RootCAs`  包含颁发`TLS`服务器证书的`CA`的证书链的文件的完全限定路径

默认情况下，**在`orderer`上关闭`TLS`客户端身份验证**，就像`peer`一样。要**启用`TLS`客户端身份验证**，请设置以下配置属性：

- `General.TLS.ClientAuthRequired` = `true`
- `General.TLS.ClientRootCAs`  包含**颁发`TLS`服务器证书的`CA`的证书链**的文件的完全限定路径

通过设置以下**环境变量**，也可以启用具有客户端身份验证的`TLS`：

- `ORDERER_GENERAL_TLS_ENABLED` = `true`
- `ORDERER_GENERAL_TLS_PRIVATEKEY` 包含**服务器私钥**的文件的完全限定路径
- `ORDERER_GENERAL_TLS_CERTIFICATE`  包含**服务器证书**的文件的完全限定路径
- `ORDERER_GENERAL_TLS_ROOTCAS`  包含**颁发`TLS`服务器证书的`CA`的证书链**的文件的完全限定路径
- `ORDERER_GENERAL_TLS_CLIENTAUTHREQUIRED` = `true`
- `ORDERER_GENERAL_TLS_CLIENTROOTCAS`  包含**颁发`TLS`服务器证书的`CA`的证书链**的文件的完全限定路径

# 为对等`CLI`配置`TLS`

对**启用`TLS`的对等节点运行对等`CLI` 命令**时，必须设置以下**环境变量**：

- `CORE_PEER_TLS_ENABLED` = `true`
- `CORE_PEER_TLS_ROOTCERT_FILE`  包含**颁发`TLS`服务器证书的`CA`的证书链**的文件的完全限定路径

如果在**远程服务器**上也启用了`TLS`客户端身份验证，则**除上述变量外，还必须设置以下变量**：

- `CORE_PEER_TLS_CLIENTAUTHREQUIRED` = `true`
- `CORE_PEER_TLS_CLIENTCERT_FILE`  **客户端证书**的完全限定路径
- `CORE_PEER_TLS_CLIENTKEY_FILE`  **客户端私钥**的完全限定路径

当**运行连接到`orderer`服务的命令**时，如**对等通道**`<create | update | fetch>`或**对等链码**`<invoke | instantiate>`，如果**在`orderer`上启用了`TLS`**，则还必须指定以下命令行参数：

- `–tls`
- `–cafile` 包含**定序人`CA`的证书链**的文件的完全限定路径

如果**在定序人上启用了`TLS`客户端身份验证**，则还必须指定以下参数：

- `–clientauth`
- `–keyfile`  包含**客户端私钥**的文件的完全限定路径
- `–certfile`  包含**客户端证书**的文件的完全限定路径

# 调试`TLS`问题

在调试`TLS`问题之前，建议**在`TLS`客户端和服务器端启用`GRPC`调试**以获取其他信息。要**启用`GRPC`调试**，请将**环境变量`FABRIC_LOGGING_SPEC`设置为包含`grpc = debug`**。例如，要将缺省日志记录级别设置为`INFO`并将`GRPC`日志记录级别设置为`DEBUG`，请将日志记录规范**设置为`grpc = debug:info`**。

如果在客户端看到错误消息`remote error：tls：bad certificate`，则通常**表示`TLS`服务器已启用客户端身份验证**，并且服务器**未收到正确**的客户端证书，或者它收到的客户端证书**不是信任**。确保客户端正在发送其证书，并且已由**对等方或定序方节点信任的其中一个`CA`证书进行签名**。

如果在**链码日志**中看到错误消息`remote error: tls: bad certificate`，请确保**使用`Fabric v1.1`或更新版本提供的`chaincode shim`构建链码**。如果**链码不包含`shim`的`vendored `副本**，**删除链码容器并重新启动**其对等端将使用当前的`shim`版本重建链码容器。