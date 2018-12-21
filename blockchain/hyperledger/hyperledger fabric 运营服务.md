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

