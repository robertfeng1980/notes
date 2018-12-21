# `hyperledger fabric` 运营服务

对等方和订货方托管提供`RESTful` 操作 `API`的`HTTP`服务器。**此`API`与`Fabric`网络服务无关**，旨在**供运营商使用**，而不是网络的管理员或用户。

`API`公开了以下功能：

+ 日志级别管理
+ 健康检查
+ `Prometheus`针对运营指标的目标（配置时）

## 运营服务配置

操作服务需要两个基本配置：

+ 要**监听**的**地址和端口**。
+ 用于**身份验证**和**加密的`TLS`证书和密钥**。请注意，这些证书应由**单独的专用`CA`生成**。不要使用为任何渠道中的任何组织生成证书的`CA`。

## Peer

对于每个对等体，可以在`core.yaml`的操作部分中配置操作服务器：

```yaml
operations:
  # host and port for the operations server
  listenAddress: 127.0.0.1:9443

  # TLS configuration for the operations endpoint
  tls:
    # TLS enabled
    enabled: true

    # path to PEM encoded server certificate for the operations server
    cert:
      file: tls/server.crt

    # path to PEM encoded server key for the operations server
    key:
      file: tls/server.key

    # require client certificate authentication to access all resources
    # 要求客户端证书身份验证访问所有资源
    clientAuthRequired: false

    # paths to PEM encoded ca certificates to trust for client authentication
    # PEM编码的ca证书的路径，以信任客户端身份验证
    clientRootCAs:
      files: []
```

`listenAddress`键定义操作服务器将侦听的主机和端口。如果服务器应该监听所有地址，则可以省略主机部分。

`tls`部分用于指示是否为操作服务启用了`TLS`，服务的证书和私钥的位置以及应该为客户端身份验证信任的证书颁发机构根证书的位置。当`clientAuthRequired`为`true`时，将要求客户端提供用于身份验证的证书。

## Orderer

对于每个订货人，可以在`orderer.yaml`的“操作”部分中配置操作服务器：

```yml
Operations:
  # host and port for the operations server
  ListenAddress: 127.0.0.1:8443

  # TLS configuration for the operations endpoint
  TLS:
    # TLS enabled
    Enabled: true

    # PrivateKey: PEM-encoded tls key for the operations endpoint
    PrivateKey: tls/server.key

    # Certificate governs the file location of the server TLS certificate.
    Certificate: tls/server.crt

    # Paths to PEM encoded ca certificates to trust for client authentication
    RootCAs: []

    # Require client certificate authentication to access all endpoints
    ClientAuthRequired: false
```

`ListenAddress`键定义操作服务器将侦听的主机和端口。如果服务器应该监听所有地址，则可以省略主机部分。

`TLS`部分用于指示是否为操作服务启用了`TLS`，服务证书和私钥的位置以及应该为客户端身份验证信任的证书颁发机构根证书的位置。当`ClientAuthRequired`为`true`时，将要求客户端提供用于身份验证的证书。

## 运营安全

由于操作服务侧重于操作并且与`Fabric`网络故意无关，因此它不使用成员资格服务提供程序进行访问控制。相反，操作服务完全依赖于具有客户端证书身份验证的相互`TLS`。

强烈建议通过在生产环境中将`clientAuthRequired`的值设置为`true`来启用相互`TLS`。使用此配置，客户端需要提供有效的身份验证证书。如果客户端未提供证书或服务无法验证客户端的证书，则拒绝该请求。请注意，如果`clientAuthRequired`设置为`false`，则客户端不需要提供证书。但是，如果他们这样做，并且服务无法验证证书，则该请求将被拒绝。

禁用`TLS`时，将绕过授权，并且任何可以连接到操作端点的客户端都可以使用该`API`。

## 日志级别管理

操作服务提供`/logspec`资源，操作员可以使用该资源来管理对等方或订货方的活动日志记录规范。该资源是传统的`REST`资源，支持`GET`和`PUT`请求。

当操作服务收到`GET/logspec`请求时，它将使用包含当前日志记录规范的`JSON`有效内容进行响应：

```json
{"spec":"info"}
```

当操作服务收到`PUT / logspec`请求时，它将读取主体作为`JSON`有效负载。有效负载必须由名为`spec`的单个属性组成。

```yml
{"spec":"chaincode=debug:info"}
```

如果规范成功激活，该服务将响应`204` “无内容”响应。如果发生错误，服务将响应`400` “错误请求”和错误有效负载：

```json
{"error":"error message"}
```

