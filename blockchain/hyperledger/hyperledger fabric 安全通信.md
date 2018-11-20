# `hyperledger fabric` 安全通信

`Fabric`支持**使用`TLS`在节点之间进行安全通信**。`TLS`通信可以使用**单向**（仅服务器）和**双向**（服务器和客户端）身份验证。

# 为对等节点配置`TLS`

**对等节点既是`TLS`服务器又是`TLS`客户端**。它是前者，当另一个对等节点，应用程序或`CLI`与其建立连接时，后者在与另一个对等节点或订购者建立连接时。

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

当对等体**加入通道**时，从通道的**配置块**读取通道成员的**根CA证书链**，并将其**添加到`TLS`客户端和服务器根`CA`数据结构中**。因此，点对点通信，点对点订购者通信**应该无缝地工作**。