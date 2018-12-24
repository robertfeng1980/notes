# `hyperledger fabric` 运营服务

对等方和订货方托管提供`RESTful` 操作 `API`的`HTTP`服务器。**此`API`与`Fabric`网络服务无关**，旨在**供运营商使用**，而不是网络的管理员或用户。

`API`公开了以下功能：

+ 日志级别管理
+ 健康检查
+ `Prometheus`针对运营指标的目标（配置时）

# 运营服务配置

操作服务需要两个基本配置：

+ 要**监听**的**地址和端口**。
+ 用于**身份验证**和**加密的`TLS`证书和密钥**。请注意，这些证书应由**单独的专用`CA`生成**。不要使用为任何渠道中的任何组织生成证书的`CA`。

# `Peer`

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

# `Orderer`

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

# 运营安全

由于操作服务侧重于操作并且与`Fabric`网络故意无关，因此它不使用成员资格服务提供程序进行访问控制。相反，操作服务完全依赖于具有客户端证书身份验证的相互`TLS`。

强烈建议通过在生产环境中将`clientAuthRequired`的值设置为`true`来启用相互`TLS`。使用此配置，客户端需要提供有效的身份验证证书。如果客户端未提供证书或服务无法验证客户端的证书，则拒绝该请求。请注意，如果`clientAuthRequired`设置为`false`，则客户端不需要提供证书。但是，如果他们这样做，并且服务无法验证证书，则该请求将被拒绝。

禁用`TLS`时，将绕过授权，并且任何可以连接到操作端点的客户端都可以使用该`API`。

# 日志级别管理

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

# 健康检查

运营服务提供`/healthz`资源，运营商可以使用该资源来帮助确定同伴和订购者的**活跃度和健康状况**。该资源是支持`GET`请求的传统`REST`资源。该实现旨在与`Kubernetes`使用的**活性探针模型**兼容，但可以**在其他上下文中**使用。

收到`GET /healthz`请求后，操作服务将调用**所有已注册的健康检查程序**进行该过程。当所有健康检查程序成功返回时，操作服务将以`200 “OK”`和`JSON`正文响应：

```json
{
  "status": "OK",
  "time": "2009-11-10T23:00:00Z"
}
```

如果一个或多个运行状况检查程序**返回错误**，则操作服务将使用`503 "Service Unavailable" `和`JSON`正文进行响应，该**正文包含有关哪个运行状况检查**程序失败的信息：

```json
{
  "status": "Service Unavailable",
  "time": "2009-11-10T23:00:00Z",
  "failed_checks": [
    {
      "component": "docker",
      "reason": "failed to connect to Docker daemon: invalid endpoint"
    }
  ]
}
```

在当前版本中，**唯一注册的运行状况检查是针对`Docker`的**。将增强未来版本以添加其他健康检查。

启用`TLS`后，除非`requireClientAuth`设置为`true`，否则**不需要有效的客户端证书**来使用此服务。

# `Metrics`

`Fabric`对等和订购者的某些组件公开可以帮助深入了解系统行为的指标。操作员和管理员可以使用此信息更好地**了解系统的性能**。

## 配置度量标准

`Fabric`提供了两种公开指标的方法：基于`Prometheus`的**拉**模型和基于`StatsD`的**推**模型。

## `Prometheus`

典型的`Prometheus`部署通过从**检测目标公开的`HTTP`端点请求指标**来刮擦指标。由于`Prometheus`负责请求指标，因此它被视为**拉动系统**。

配置后，`Fabric`对等方或订货方将在操作服务上显示`/metrics`资源。

### `Peer`

通过在`core.yaml`的`/metrics`部分中将`metrics`提供程序设置为`prometheus`，可以将对等体配置为公开`metrics`端点以便`Prometheus`进行刮擦。

```yaml
metrics:
  provider: prometheus
```

### `Orderer`

通过在`orderer.yaml`的`Metrics`部分中将`metrics`度量提供程序设置为`prometheus`，可以将`orderer`配置为通过将`metrics`度量提供程序设置为`promeheus`来公开`/metrics`端点。

```yaml
Metrics:
  Provider: prometheus
```

## `StatsD`

`StatsD`是一个**简单的统计聚合**守护进程。度量标准被**发送到`statsd`守护程序**，在那里它们被**收集**，**聚合并推送到后端**以进行**可视化和警报**。由于此模型要求检测过程将度量数据发送到`StatsD`，因此这被视为**推送系统**。

### `Peer`

可以将对等体配置为通过在`core.yaml`的`metrics`部分中将度量提供者**设置为`statsd`来将度量发送到`StatsD`**。还**必须使用`StatsD`守护程序的地址**，要使用的网络类型（`tcp`或`udp`）以及发送度量标准的频率来配置`statsd`子部分。可以指定**可选前缀**以帮助**区分度量的来源**。例如，区分来自不同对等点的度量，将被添加到所有生成的度量之前。

```yaml
metrics:
  provider: statsd
  statsd:
    network: udp
    address: 127.0.0.1:8125
    writeInterval: 10s
    prefix: peer-0
```

### `Orderer`

可以将订购者配置为通过在`orderer.yaml`的“指标”部分中将指标提供程序设置为`statsd`来将指标发送到`StatsD`。还必须使用`StatsD`守护程序的地址，要使用的网络类型（`tcp`或`udp`）以及发送度量标准的频率来配置`Statsd`子部分。可以指定可选前缀以帮助区分度量的来源。

```yaml
Metrics:
    Provider: statsd
    Statsd:
      Network: udp
      Address: 127.0.0.1:8125
      WriteInterval: 30s
      Prefix: org-orderer
```

要查看生成的不同指标，请查看 [指标参考](https://hyperledger-fabric.readthedocs.io/en/latest/metrics_reference.html)。

# `Metrics` 参考

## `Prometheus Metrics`

目前，以下指标已导出供`Prometheus`使用。

| Name                                                | Type      | Description                                        | Labels                                             |
| --------------------------------------------------- | --------- | -------------------------------------------------- | -------------------------------------------------- |
| blockcutter_block_fill_duration                     | histogram | 从第一个事务入队到块被切断的时间（以秒为单位）。   | channel                                            |
| broadcast_enqueue_duration                          | histogram | 在几秒钟内排队交易的时间。                         | channel type status                                |
| broadcast_processed_count                           | counter   | 处理的交易数量。                                   | channel type status                                |
| broadcast_validate_duration                         | histogram | 在几秒钟内验证交易的时间。                         | channel type status                                |
| chaincode_execute_timeouts                          | counter   | 已超时的链代码执行次数（Init或Invoke）。           | chaincode                                          |
| chaincode_launch_duration                           | histogram | 发布链码的时间。                                   | chaincode success                                  |
| chaincode_launch_failures                           | counter   | 已失败的链码启动次数。                             | chaincode                                          |
| chaincode_launch_timeouts                           | counter   | 已超时的链代码启动次数。                           | chaincode                                          |
| chaincode_shim_request_duration                     | histogram | 完成链码补丁请求的时间。                           | type channel chaincode success                     |
| chaincode_shim_requests_completed                   | counter   | 完成了链码填充请求的数量。                         | type channel chaincode success                     |
| chaincode_shim_requests_received                    | counter   | 收到的链码补丁请求数。                             | type channel chaincode                             |
| consensus_kafka_batch_size                          | gauge     | 发送到主题的平均批量大小（以字节为单位）           | topic                                              |
| consensus_kafka_compression_ratio                   | gauge     | 主题的平均压缩比（以百分比表示）。                 | topic                                              |
| consensus_kafka_incoming_byte_rate                  | gauge     | 字节/秒读取经纪人。                                | broker_id                                          |
| consensus_kafka_outgoing_byte_rate                  | gauge     | 字节/秒写给经纪人。                                | broker_id                                          |
| consensus_kafka_record_send_rate                    | gauge     | 发送到主题的每秒记录数。                           | topic                                              |
| consensus_kafka_records_per_request                 | gauge     | 每个请求发送到主题的平均记录数。                   | topic                                              |
| consensus_kafka_request_latency                     | gauge     | 对代理的平均请求延迟（以ms为单位）。               | broker_id                                          |
| consensus_kafka_request_rate                        | gauge     | 请求/秒发送给经纪人。                              | broker_id                                          |
| consensus_kafka_request_size                        | gauge     | 经纪人的平均请求大小（以字节为单位）。             | broker_id                                          |
| consensus_kafka_response_rate                       | gauge     | 请求/秒发送给经纪人。                              | broker_id                                          |
| consensus_kafka_response_size                       | gauge     | 经纪人的平均响应大小（以字节为单位）。             | broker_id                                          |
| couchdb_processing_time                             | histogram | 函数完成对CouchDB的请求所需的时间（以秒为单位）    | database function_name result                      |
| deliver_blocks_sent                                 | counter   | 交付服务发送的块数。                               | channel filtered                                   |
| deliver_requests_completed                          | counter   | 已完成的交付请求数。                               | channel filtered success                           |
| deliver_requests_received                           | counter   | 已收到的递送请求数。                               | channel filtered                                   |
| deliver_streams_closed                              | counter   | 已为传送服务关闭的GRPC流的数量。                   |                                                    |
| deliver_streams_opened                              | counter   | 已为传递服务打开的GRPC流的数量。                   |                                                    |
| dockercontroller_chaincode_container_build_duration | histogram | 在几秒钟内构建链代码图像的时间。                   | chaincode success                                  |
| fabric_version                                      | gauge     | Fabric的活动版本。                                 | version                                            |
| grpc_comm_conn_closed                               | counter   | gRPC连接已关闭。打开减去关闭是活动的连接数。       |                                                    |
| grpc_comm_conn_opened                               | counter   | gRPC连接已打开。打开减去关闭是活动的连接数。       |                                                    |
| grpc_server_stream_messages_received                | counter   | 收到的流消息数。                                   | service method                                     |
| grpc_server_stream_messages_sent                    | counter   | 发送的流消息数。                                   | service method                                     |
| grpc_server_stream_request_duration                 | histogram | 完成流请求的时间。                                 | service method code                                |
| grpc_server_stream_requests_completed               | counter   | 已完成的流请求数。                                 | service method code                                |
| grpc_server_stream_requests_received                | counter   | 收到的流请求数。                                   | service method                                     |
| grpc_server_unary_request_duration                  | histogram | 完成一元请求的时间。                               | service method code                                |
| grpc_server_unary_requests_completed                | counter   | 已完成的流请求数。                                 | service method code                                |
| grpc_server_unary_requests_received                 | counter   | 收到的流请求数。                                   | service method                                     |
| ledger_block_processing_time                        | histogram | 分类帐块处理的时间以秒为单位。                     | channel                                            |
| ledger_blockchain_height                            | gauge     | 块中链的高度。                                     | channel                                            |
| ledger_blockstorage_commit_time                     | histogram | 将块和私有数据提交到存储所需的时间（以秒为单位）。 | channel                                            |
| ledger_statedb_commit_time                          | histogram | 将块更改提交到状态db所需的时间（以秒为单位）。     | channel                                            |
| ledger_transaction_count                            | counter   | 处理的交易数量。                                   | channel transaction_type chaincode validation_code |

