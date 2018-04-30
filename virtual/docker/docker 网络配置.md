# docker 网络配置

[TOC]

# 概述

Docker容器和服务如此强大的原因之一是可以将它们连接在一起，或将它们连接到非Docker工作环境进行负载。Docker容器和服务甚至不需要知道它们是否部署在Docker上，或者它们的对等端是否也是Docker工作负载。无论Docker主机是运行Linux、Windows还是两者的组合，都可以使用Docker以平台无关的方式管理它们。


## 网络驱动

Docker的网络子系统是可插拔的，使用驱动程序。默认情况下存在几个驱动程序，并提供核心网络功能：

- `bridge`：默认的网络驱动程序。如果您不指定驱动程序，则这是您正在创建的网络类型。**当您的应用程序运行在需要通信的独立容器中时，通常会使用桥接网络。**请参阅 [桥接网络](https://docs.docker.com/network/bridge/)。
- `host`：对于独立容器，删除容器和Docker主机之间的网络隔离，并直接使用主机的网络。`host` 仅适用于Docker 17.06及更高版本的群集服务。请参阅 [使用主机网络](https://docs.docker.com/network/host/)。
- `overlay`：覆盖网络**将多个Docker守护进程连接在一起，并使群集服务能够相互通信**。您还可以使用覆盖网络来促进**swarm服务和独立容器之间的通信**，或者不同Docker守护进程上的两个独立容器之间的通信。这种策略消除了**在这些容器之间进行操作系统级路由**的需求。请参阅[覆盖网络](https://docs.docker.com/network/overlay/)。
- `macvlan`：Macvlan网络允许您为容器**分配MAC地址，使其显示为网络上的物理设备**。Docker守护进程通过**MAC地址将流量路由到容器**。使用`macvlan` 驱动程序有时是处理希望直接连接到物理网络的传统应用程序的最佳选择，而不是通过Docker主机的网络堆栈进行路由。请参阅 [Macvlan网络](https://docs.docker.com/network/macvlan/)。
- `none`：对于此容器，**禁用所有网络**。通常与自定义网络驱动程序一起使用。`none`不适用于群组服务。请参阅 [禁用容器联网](https://docs.docker.com/network/none/)。
- [Network plugins 网络插件](https://docs.docker.com/engine/extend/plugins_services/)：您可以在Docker上安装和使用第三方网络插件。这些插件可从 [Docker Store](https://store.docker.com/search?category=network&q=&type=plugin) 或第三方供应商处获得。

### 网络驱动总结

- **用户定义的网桥**：需要多个容器在同一个Docker主机上进行通信。
- **主机网络**： 当网络堆栈不应该与Docker主机隔离时，但您希望隔离容器的其他方面。
- **覆盖网络**： 当需要运行在不同Docker主机上的容器进行通信，或者当多个应用程序使用群集服务一起工作。
- **Macvlan网络**： 最适合从虚拟机设置迁移或需要容器看起来像网络上的物理主机，每个物理主机都有一个唯一的MAC地址。
- **第三方网络插件**： 允许将Docker与专用网络堆栈集成。

## Docker EE 网络

以下两个功能只有在使用Docker EE和使用通用控制平面（UCP）管理Docker服务时才有可能：

- 该[HTTP路由网格](https://docs.docker.com/datacenter/ucp/2.2/guides/admin/configure/use-domain-names-to-access-services/) 可以让你分享多个服务之间的相同的网络IP地址和端口。根据客户端的要求，UCP使用主机名和端口的组合将流量路由到相应的服务。
- [会话粘性](https://docs.docker.com/datacenter/ucp/2.2/guides/user/services/use-domain-names-to-access-services/#sticky-sessions)允许您指定UCP用于将后续请求路由到相同服务任务的HTTP标头中的信息，用于需要有状态会话的应用程序。

## 网络教程

本主题定义了一些基本的Docker网络概念，并为您准备设计和部署应用程序以充分利用这些功能。
大部分内容适用于所有Docker安装。但是 [一些高级功能](https://docs.docker.com/network/#docker-ee-networking-features)仅适用于Docker EE。

现在已经了解了Docker网络的基本知识，使用以下教程加深理解：

- [独立网络教程](https://docs.docker.com/network/network-tutorial-standalone/)
- [主机网络教程](https://docs.docker.com/network/network-tutorial-host/)
- [覆盖网络教程](https://docs.docker.com/network/network-tutorial-overlay/)
- [Macvlan网络教程](https://docs.docker.com/network/network-tutorial-macvlan/)

# docker network 基本命令

```shell
$ docker network -h
管理网络
Usage:  docker network COMMAND

命令：
  connect     # 将容器连接到网络
  create      # 创建一个网络
  disconnect  # 从网络断开容器
  inspect     # 在一个或多个网络上显示详细信息
  ls          # 列出网络
  prune       # 删除所有未使用的网络
  rm          # 删除一个或多个网络
```

## ls 查看


### 命令参数选项

---

```shell
Options:
  -f, --filter filter   #提供过滤器值 (e.g. 'driver=bridge')
      --format string   #使用Go模板的漂亮打印网络
      --no-trunc        #不要截断输出
  -q, --quiet           #只显示网络ID
```
### 示例

---

#### 查看

`docker network ls`查看网络列表，列举出已经存在的网络数据

```shell
$ docker network ls
NETWORK ID          NAME                                             DRIVER              SCOPE
0daf44c54e3b        bridge                                           bridge              local
99b53a220971        host                                             host                local
lzq7bs60xe53        ingress                                          overlay             swarm
56d47aac58e8        my-bri-0                                         bridge              local
b9997816c388        my-bridge-net                                    bridge              local
a1fcabf9c7b3        my-mac-net                                       macvlan             local
qqabvnny71wz        my-overlay-inte-net                              overlay             swarm
6ei164q3kk09        my-overlay-net                                   overlay             swarm
91efc6379be0        none                                             null                local
```

#### 过滤

使用`filter`过滤查看

```shell
$ docker network ls --filter 'driver=host'
NETWORK ID          NAME                DRIVER              SCOPE
99b53a220971        host                host                local
```

过滤标志（`-f`或`--filter`）格式是一`key=value`对。如果有多个过滤器，则传递多个标志（例如`--filter "foo=bar" --filter "bif=baz"`）。多个过滤器标志被组合为一个`OR`过滤器。例如，`-f type=custom -f type=builtin`返回两者`custom`和`builtin`网络。

目前支持的过滤器是：

- driver
- id (network’s id)
- label (`label=<key>` or `label=<key>=<value>`)
- name (`network’s name`)
- scope (`swarm|global|local`)
- type (`custom|builtin`) `type`过滤器支持两个值; `builtin`显示预定义的网络（`bridge`，`none`，`host`），而`custom`显示用户定义的网络。

```shell
$ docker network ls --filter 'driver=host'
$ docker network ls --filter scope=swarm
# 查看自定义网络
$ docker network ls -f type=custom
$ docker network ls -f type=builtin
NETWORK ID          NAME                DRIVER              SCOPE
0daf44c54e3b        bridge              bridge              local
99b53a220971        host                host                local
91efc6379be0        none                null                local
```

#### 格式化

格式化输出列表

```shell
$ docker network ls --format "{{.Name}} \t {{.Driver}} \t {{.IPv6}} \t {{.Internal}} \t {{.Labels}}"
bridge   bridge          false   false
com.docker.network.bridge.name=my-bridge-opt-1   bridge          false   false
compose_sample_default   bridge          false   false   com.docker.compose.network=default,com.docker.compose.project=compose_sample
composeexample_default   bridge          false   false   com.docker.compose.network=default,com.docker.compose.project=composeexample
deploy_test_overlay      overlay         false   false   com.docker.stack.namespace=deploy_test
docker_gwbridge          bridge          false   false
host     host    false   false
```

模板占位符

| 占位符       | 描述                                                     |
| ------------ | -------------------------------------------------------- |
| `.ID`        | 网络ID                                                   |
| `.Name`      | 网络名字                                                 |
| `.Driver`    | 网络驱动                                                 |
| `.Scope`     | 网络范围（本地，全球）                                   |
| `.IPv6`      | 是否在网络上启用IPv6。                                   |
| `.Internal`  | 网络是否是内部的。                                       |
| `.Labels`    | 所有分配给网络的标签。                                   |
| `.Label`     | 此网络的特定标签的值。例如`{{.Label "project.version"}}` |
| `.CreatedAt` | 网络创建的时间                                           |

#### 其他

其他命令操作，显示完整`network id`：

```shell
$ docker network ls --filter 'driver=host' -q
99b53a220971

$ docker network ls --filter 'driver=host' -q --no-trunc
99b53a220971c01d6f4bf1cf9571d85f5cabd08f3e308cedaf96f4a0a18e0789
```

## create 创建

### 命令参数选项

---

```shell
$ docker network create -h
Usage:  docker network create [OPTIONS] NETWORK

创建一个网络

Options:
      --attachable           # 启用手动容器附件
      --aux-address map      # 使用的辅助IPv4或IPv6地址网络驱动程序（默认 map[])
      --config-from string   # 复制配置的网络
      --config-only          # 创建仅配置网络
  -d, --driver string        # Driver管理网络 (default "bridge")
      --gateway strings      # 主站子网的IPv4或IPv6网关
      --ingress              # 创建群组路由 - 网状集群网络
      --internal             # 限制对网络的外部访问
      --ip-range strings     # 从子网范围分配容器ip
      --ipam-driver string   # IP地址管理驱动程序 (default "default")
      --ipam-opt map         # 设置IPAM驱动程序特定选项 (default map[])
      --ipv6                 # 启用IPv6网络
      --label list           # 在网络上设置元数据
  -o, --opt map              # 设置驱动程序特定的选项 (default map[])
      --scope string         # 控制网络的范围
      --subnet strings       # 以CIDR格式表示的子网串网段
```

### 扩充阅读

---

创建一个新的网络。`DRIVER`接受内置网络驱动程序的`bridge`或者`overlay`。如果已经安装了第三方或自己的自定义网络驱动程序，也可以指定`DRIVER`。如果不指定该 `--driver`选项，该命令将自动创建一个`bridge`网络。当安装`Docker Engine`时，它会自动创建一个`bridge`网络。该网络对应于`docker0`引擎传统依赖的桥梁。当`docker run`启动一个新的容器时， 它会自动连接到这个网桥。您无法删除此默认桥接网络，但可以使用`network create`命令创建新桥接网络。

```shell
$ docker network create -d bridge my-bridge-network
```

桥接网络是单个引擎安装中的隔离网络。如果您想创建一个跨多个运行引擎的多个Docker主机的网络，则必须创建一个`overlay`网络。与`bridge`网络不同，`overlay`覆盖网络在创建之前需要一些预先存在的条件。这些条件是：

- 访问键值存储。引擎支持`Consul`、`Etcd`和`ZooKeeper`（分布式存储）键值存储。
- 连接到键值存储的主机集群。
- 集群中每台主机上配置正确的引擎`daemon`。

`dockerd`支持`overlay`网络的选项是：

- `--cluster-store`
- `--cluster-store-opt`
- `--cluster-advertise`

要详细了解这些选项以及如何配置它们，请参阅[“ *多主机网络入门* ”](https://docs.docker.com/engine/userguide/networking/get-started-overlay)。

虽然不是必需的，但安装`Docker Swarm`来管理组成网络的集群是一个不错的主意。`Swarm`提供了复杂的发现和服务器管理工具，可以帮助您实施。

准备好`overlay`网络必须条件后，只需在集群中选择Docker主机并发出以下命令即可创建网络：

```shell
$ docker network create -d overlay my-multihost-network
```

网络名称**必须是唯一**的。Docker守护进程尝试识别命名冲突，但**不能保证**。避免名称冲突是用户的责任。

#### 覆盖网络限制

当您使用默认的基于VIP的端点模式创建网络时，您应该创建带有24个块（默认值）的覆盖网络，这会将您限制为256个IP地址。该建议解决了 [群集模式的限制](https://github.com/moby/moby/issues/30820)。如果您需要超过256个IP地址，请勿增加IP块大小。您可以通过`dnsrr`外部负载均衡器使用端点模式，也可以使用多个较小的覆盖网络。 有关不同端点模式的更多信息，请参阅 [配置服务发现](https://docs.docker.com/engine/swarm/networking/#configure-service-discovery)。

### 示例

---

#### 创建网络

下面利用命令简单的创建一些网络，其中`host`网络已经默认创建过了，就不能再重复创建。

网络类型固定常用的有：`bridge`、`host`、`overlay`、`macvlan`、自定义网络。

```shell
# 创建网络，默认桥接网络
$ docker network create my-default-bri-net

# 创建桥接网络
$ docker network create -d bridge my-bridge-net
b9997816c38807c37fb2f9f2cabcac1017ce21e05f925033fda0481e5ef35aca

$ docker network create -d host my-host-net
Error response from daemon: only one instance of "host" network is allowed

# 创建覆盖型网络
$ docker network create -d overlay my-overlay-net
6ei164q3kk099o4jcmj0lhtt8

# 创建mac网络
$ docker network create -d macvlan my-mac-net
a1fcabf9c7b362ea558b4669bbb71b5c0b65661be8140200f4d36f33e20c1680
```

#### 连接容器

当您启动容器时，请使用`--network`选项将其连接到网络。本例将`busybox`容器添加到`mynet`网络中：

```shell
$ docker network create -d bridge mynet
$ docker run -itd --network=mynet busybox
```

如果要将已经运行的容器添加到网络，请使用`docker network connect`命令。

您可以将多个容器连接到同一个网络。连接后，容器只能使用另一个容器的IP地址或名称进行通信。对于`overlay`支持多主机连接的网络或自定义插件，连接到相同多主机网络但从不同引擎启动的容器也可以通过这种方式进行通信。

您可以使用`docker network disconnect`命令从网络断开容器。

#### 高级选项

在创建网络时，Engine默认会为网络创建一个不重叠的子网。这个子网不是现有网络的细分。这目的纯粹是为了ip地址。可以覆盖此默认值并直接使用`--subnet`选项指定子网值。在 `bridge`网络上只能创建一个子网：

```shell
$ docker network create --driver=bridge --subnet=192.168.0.0/18 my-bri-0
56d47aac58e8b819cc3ad5637a5da1a3a252406594af354d03eed6ab966357f2
```

另外，还可以指定`--gateway` `--ip-range`和`--aux-address` 选项。

```shell
$ docker network create \
  --driver=bridge \
  --subnet=172.28.0.0/16 \
  --ip-range=172.28.5.0/24 \
  --gateway=172.28.5.254 \
  br0
```

如果您忽略该`--gateway`标志，引擎会从首选游泳池内为您选择一个。对于`overlay`网络和支持它的网络驱动程序插件，您可以创建多个子网。本示例使用两个`/25` 子网掩码来坚持当前的指导，即在单个覆盖网络中没有超过256个IP。每个子网络有126个可用地址。

```shell
$ docker network create -d overlay \
  --subnet=192.168.1.0/25 \
  --subnet=192.170.2.0/25 \
  --gateway=192.168.1.100 \
  --gateway=192.170.2.100 \
  --aux-address="my-router=192.168.1.5" --aux-address="my-switch=192.168.1.6" \
  --aux-address="my-printer=192.170.1.5" --aux-address="my-nas=192.170.1.6" \
  my-multihost-network
```

确保你的子网不重叠。如果他们这样做，网络创建失败，引擎返回错误。

#### 桥模式选项

在创建自定义网络时，默认网络驱动程序（即`bridge`）具有可以传递的其他选项。以下是用于docker0桥的那些选项和等效的docker守护进程标志：

| 选项                                             | 等价        | 描述                        |
| ------------------------------------------------ | ----------- | --------------------------- |
| `com.docker.network.bridge.name`                 | -           | 创建Linux桥时要使用的桥名称 |
| `com.docker.network.bridge.enable_ip_masquerade` | `--ip-masq` | 启用IP伪装                  |
| `com.docker.network.bridge.enable_icc`           | `--icc`     | 启用或禁用容器间连接        |
| `com.docker.network.bridge.host_binding_ipv4`    | `--ip`      | 绑定容器端口时的默认IP      |
| `com.docker.network.driver.mtu`                  | `--mtu`     | 设置容器网络MTU             |

以下参数可以传递给`docker network create`任何网络驱动程序，同样也可以传递给它们的等价`docker daemon`。

| Argument     | 等价           | 描述                   |
| ------------ | -------------- | ---------------------- |
| `--gateway`  | -              | 主子网的IPv4或IPv6网关 |
| `--ip-range` | `--fixed-cidr` | 从一个范围分配IP       |
| `--internal` | -              | 限制对网络的外部访问   |
| `--ipv6`     | `--ipv6`       | 启用IPv6网络           |
| `--subnet`   | `--bip`        | 网络子网               |

例如，让我们使用`-o`或`--opt`选项在发布端口时指定IP地址绑定：

```shell
$ docker network create \
    -o "com.docker.network.bridge.host_binding_ipv4"="172.19.0.1" \
    simple-network
```

#### 网络内部模式

默认情况下，当您将容器连接到`overlay`网络时，Docker也会将桥接网络连接到它以提供外部连接。如果您想创建一个外部隔离的`overlay`网络，您可以指定该 `--internal`选项。

```shell
$ docker network create --internal -d overlay my-overlay-inte-net
```

#### 网络入口模式

您可以创建将用于在群集中提供路由网格的网络。您通过在创建网络时指定`--ingress`来完成此操作。当时只能创建一个入口网络。只有在没有服务依赖它的情况下才能删除网络。除了`--attachable`选项之外，在创建入口网络时，创建覆盖网络时可用的任何选项也可用。

```shell
$ docker network create -d overlay \
  --subnet=10.11.0.0/16 \
  --ingress \
  --opt com.docker.network.driver.mtu=9216 \
  --opt encrypted=true \
  my-ingress-network
```

## connect 连接

### 命令参数选项

---

```shell
Options:
      --alias strings           # 为容器添加网络范围的别名
      --ip string               # IPv4地址 (e.g., 172.30.100.104)
      --ip6 string              # IPv6地址 (e.g., 2001:db8::33)
      --link list               # 将链接添加到其他容器
      --link-local-ip strings   # 为容器添加链接本地地址
```

### 示例

---

将容器连接到网络。您可以按名称或ID连接容器。连接后，容器可以与同一网络中的其他容器进行通信。

#### 为正在运行的容器连接网络

```shell
# 运行一个镜像，没有会自动去下载
$ docker run -it busybox
$ docker network ls
$ docker container ls
# 使用id 
$ docker network connect 61b951b60b24 91efc6379be0
# 使用name
$ docker network connect my-bridge-net suspicious_kepler
```

#### 将容器启动时连接到网络

可以使用`docker run --network=<network-name>`选项启动容器并立即将其连接到网络。

```shell
$ docker run -it --network=my-bridge-net helloworld-jdk-9:latest
$ docker run -itd --network=multi-host-network busybox
```

#### 容器在给定网络上使用指定IP

可以指定要分配给容器固定的IP地址。对于不同的网络，分配的IP网段跟网络有关，所以先查看网络的子网网段，再分配合法的IP。

```shell
$ docker network inspect my-bri-0
$ docker network inspect my-bri-0 --format={{.IPAM.Config}}
[{192.168.0.0/18   map[]}]

$ docker network connect --ip 192.168.5.122 my-bri-0 suspicious_kepler
```

#### 使用旧版`--link`选项

可以使用`--link`选项将另一个容器与首选别名链接起来

```shell
$ docker network connect --link silly_curie:my_container my-bridge suspicious_kepler
```

#### 为容器连接网络创建别名

`--alias` 选项可用于通过连接到的网络中的其他名称来解析容器。取别名方便查看链接的容器。

```shell
$ docker network connect --alias db --alias mysql multi-host-network container2
$ docker network connect --alias linux_os --alias ubuntu ps upbeat_ptolemy
```

通过调用命令`docker container inspect`查看配置中的NetWork的Aliases

```shell
$ docker container inspect upbeat_ptolemy
"Networks": {                
                "ps": {
                    "IPAMConfig": {},
                    "Links": null,
                    "Aliases": [
                        "linux_os",
                        "ubuntu",
                        "798c3d2e4643"
                    ],
                    "NetworkID": "eca59402fd5ab6e5c51aa0e94be275599f148199dfea681aae8c6c90c21f33be",
                    "EndpointID": "df84d4f02f7a1843ab3620cdae2c5826908ab7100f0918c49da47aee9b37e477",
                    "Gateway": "172.23.0.1",
                    "IPAddress": "172.23.0.3",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:17:00:03",
                    "DriverOpts": null
                }

```

#### 停止，暂停或重新启动容器的网络

您可以暂停，重新启动并停止连接到网络的容器。**容器运行时会连接到其配置的网络**。

如果设定过网络，则在**重新启动容器时重新应用**容器的IP地址。**如果IP地址不再可用，则容器无法启动**。确保IP地址可用的一种方法是`--ip-range`在创建网络时指定一个IP地址，并从该范围之外**选择静态IP**地址。这确保了当该容器不在网络上时，IP地址**不会被另一个容器占用**。

```shell
$ docker network create --subnet 172.20.0.0/16 --ip-range 172.20.240.0/20 multi-host-network
$ docker network connect --ip 172.20.128.2 multi-host-network container2
```

要验证容器已连接，请使用`docker network inspect`命令。从网络中删除容器用`docker network disconnect`。

在网络连接后，容器只能使用另一个容器的**IP地址或名称**进行通信。对于`overlay`支持多主机连接的网络或自定义插件，连接到相同多主机网络但从不同引擎启动的容器也可以通过这种方式进行通信。

您可以将容器连接到一个或多个网络。网络不需要是相同的类型。例如，您可以连接单个容器桥和覆盖网络。

## disconnect 断开

断开容器网络连接，容器必须运行才能将其与网络断开连接。<br/>选项 `-f`强制断开`docker network disconnect [OPTIONS] NETWORK CONTAINER`

```shell
$ docker network disconnect -f my-bridge-net friendly_jepsen
# 或者用id
$ docker network disconnect -f 568e61b512d0 c7fd77e7f301
```

## inspect 检查

显示一个或多个网络的详细信息，选项`-f` `format` 格式化输出，默认情况下，此命令将呈现JSON对象中的所有结果。

```shell
$ docker network inspect my-mac-net
[
    {
        "Name": "my-mac-net",
        "Id": "a1fcabf9c7b362ea558b4669bbb71b5c0b65661be8140200f4d36f33e20c1680",
        "Created": "2018-04-29T03:45:13.331146449Z",
        "Scope": "local",
        "Driver": "macvlan",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.22.0.0/16",
                    "Gateway": "172.22.0.1"
                }
            ]
        },
.......

$ docker network inspect my-mac-net -v -f={{.IPAM.Config}}
[{172.22.0.0/16  172.22.0.1 map[]}]
$ docker network inspect my-mac-net -v -f={{.Name}}
my-mac-net
```

## prune 未用删除

删除所有未使用的网络。未使用的网络是那些没有被任何容器引用的网络，选项支持 `--filter` 过滤，`--force , -f`强制删除。

```shell
$ docker network prune
WARNING! This will remove all networks not used by at least one container.
Are you sure you want to continue? [y/N] y
Deleted Networks:
my-overlay-net
```

### 过滤

过滤标志（`--filter`）格式为“key = value”。如果有多个过滤器，则传递多个标志（例如`--filter "foo=bar" --filter "bif=baz"`）

目前支持的过滤器是：

- until （<timestamp>） - 只删除给定时间戳之前创建的网络
- label（`label=<key>`，`label=<key>=<value>`，`label!=<key>`，或`label!=<key>=<value>`） -仅删除与网络（或没有，如果`label!=...`被使用）指定的标签。

该`until`过滤器可以是Unix的时间戳，日期格式的时间戳，或持续时间字符串（例如，去`10m`，`1h30m`）计算相对于守护机器的时间。

该`label`过滤器接受两种格式：一个是`label=...`（`label=<key>`或`label=<key>=<value>`），它删除具有指定标签的网络。另一种格式是`label!=...`（`label!=<key>`或`label!=<key>=<value>`），它删除没有指定标签的网络。

以下内容将删除超过5分钟前创建的网络。请注意，系统网络（如`bridge`，`host`）`none`将永远不会被删除：

```shell
$ docker network prune --force --filter until=5m
```

## rm 删除

通过名称或标识符或ID删除一个或多个网络。要删除网络，必须首先断开与其连接的所有容器。

```shell
$ docker network rm my-network
$ docker network rm 3695c422697f my-network
```

# bridge 网络使用

就网络而言，桥接网络是一种链路层设备，用于转发网段之间的流量。网桥可以是在主机内核中运行的硬件设备或软件设备。

就Docker而言，桥接网络使用允许连接到同一桥网络的容器进行通信的软件桥，同时提供与未连接到该桥网络的容器的隔离。Docker桥接驱动程序自动在主机中安装规则，以便不同桥接网络上的容器不能直接相互通信。

桥接网络适用于在**同一个** Docker守护程序主机上运行的容器。对于在不同Docker守护进程主机上运行的容器之间的通信，您可以在操作系统级别管理路由，也可以使用[覆盖网络](https://docs.docker.com/network/overlay/)。

启动Docker时，会自动创建一个[默认桥接网络](https://docs.docker.com/network/bridge/#use-the-default-bridge-network)（也称为`bridge`），除非另行指定，否则新启动的容器将连接到该网络。您还可以创建用户定义的自定义网桥。**用户定义的网桥优于默认bridge 网络。**

## 自定义网桥和默认网桥之间的差异

- **用户定义的网桥提供更好的隔离和集装箱化应用程序之间的互操作性**。

  连接到同一个用户定义的网桥网络的容器会自动将**所有端口**相互暴露，并且**无法**连接到外部世界。这使得集装箱化的应用程序能够轻松地相互通信，而不会意外地打开对外部世界的访问。

  想象一下带有Web前端和数据库后端的应用程序。外部世界需要访问Web前端（可能在端口80上），但只有前端本身需要访问数据库主机和端口。使用用户定义的桥接器，只需要打开Web端口，并且数据库应用程序不需要打开任何端口，因为Web前端可以通过用户定义的桥接器访问它。

  如果您在默认桥接网络上运行相同的应用程序堆栈，则需要使用`-p`或`--publish` 每个标志打开Web端口和数据库端口。这意味着Docker主机需要通过其他方式阻止对数据库端口的访问。

- **用户定义的网桥提供容器之间的自动DNS解析**。

  默认网桥网络上的容器只能通过IP地址访问彼此，除非您使用该[`--link`选项](https://docs.docker.com/network/links/)，这被认为是遗留的。在用户定义的桥接网络上，容器可以通过名称或别名来解析对方。

  想象一下与前一点相同的应用程序，它具有Web前端和数据库后端。如果你打电话给你的容器`web`和`db`，Web容器可以在连接到数据库容器`db`，无论哪个码头工人托管应用程序堆栈上运行。

  如果您在默认桥接网络上运行相同的应用程序堆栈，则需要在容器之间手动创建链接（使用旧`--link` 标志）。这些链接需要在两个方向上创建，所以您可以看到，这需要两个以上需要通信的容器。或者，您可以操纵`/etc/hosts`容器中的文件，但这会产生难以调试的问题。

- **容器可以在运行中与用户定义的网络连接和分离**。

  在容器的生命周期中，您可以在运行中将其与用户定义的网络连接或断开连接。要从默认桥接网络中移除容器，您需要停止容器并使用不同的网络选项重新创建容器。

- **每个用户定义的网络都会创建一个可配置的网桥**。

  如果您的容器使用默认桥接网络，则可以对其进行配置，但所有容器都使用相同的设置，例如MTU和`iptables`规则。另外，配置默认桥接网络发生在Docker本身之外，并且需要重新启动Docker。

  用户定义的网桥是使用 `docker network create`创建和配置的。如果不同的应用程序组具有不同的网络要求，则可以在创建时分别配置每个用户定义的桥接器。

- **链接的容器在默认网桥共享环境变量上**。

  最初，在两个容器之间共享环境变量的唯一方式是使用[`--link`标志](https://docs.docker.com/network/links/)链接它们。这种类型的变量共享对于用户定义的网络是不可能的。但是，有共享环境变量的更好方式做法：

  - 多个容器可以使用Docker**卷来挂载**包含**共享信息的文件或目录**。
  - 多个容器可以一起使用`docker-compose`，组合文件可以定义**共享变量**。
  - 您可以使用**swarm服务**而不是独立容器，并利用共享[秘密](https://docs.docker.com/engine/swarm/secrets/)和 [配置](https://docs.docker.com/engine/swarm/configs/)。

连接到同一个用户定义的网桥网络的容器有效地将所有端口相互暴露。要使端口可以在不同容器网络上或非Docker主机上访问，必须使用`-p` `--publish`标志来**发布**该端口。

## 管理自定义网络

使用该`docker network create`命令创建用户定义的网桥。

```shell
$ docker network create my-net
```

您可以指定子网、IP地址范围、网关和其他选项。

使用该`docker network rm`命令删除用户定义的网桥。如果有容器连接到当前网络， 请先断开它们。

```shell
$ docker network disconnect my-net
$ docker network rm my-net
```

> **docker做了什么？**
>
> 当您创建或删除自定义的网桥或连接或断开自定义的网桥的容器时，Docker使用特定于操作系统的工具来管理底层网络基础设施（例如`iptables`，在Linux上添加或删除网桥设备或配置规则）。这些细节应该被视为实施细节。让Docker为您管理自定义网络。

## 连接到网络

当你创建一个新的容器时，你可以指定一个或多个`--network`标志。本例将一个`Nginx`容器连接到`my-net`网络。它还将容器中的端口80发布到Docker主机上的端口8080，以便外部客户端可以访问该端口。连接到`my-net` 网络的任何其他容器都可以访问`my-nginx`容器上的所有端口，反之亦然。

```shell
$ docker create --name my-nginx \
  --network my-net \
  --publish 8080:80 \
  nginx:latest
```

要将**正在运行的**容器连接到现有的自定义桥，请使用 `docker network connect`命令。以下命令将一个已经运行的`my-nginx`容器连接到一个已经存在的`my-net`网络上：

```shell
$ docker network connect my-net my-nginx
```

## 断开连接网络

要断开正在运行的容器与用户定义的桥的连接，请使用该`docker network disconnect`命令。以下命令将`my-nginx` 容器与`my-net`网络断开连接。

```shell
$ docker network disconnect my-net my-nginx
```

## 使用IPv6

如果您需要对Docker容器进行IPv6支持，则需要在创建任何IPv6网络或分配容器IPv6地址之前，在Docker守护程序上[启用该选项](https://docs.docker.com/config/daemon/ipv6/)并重新加载其配置。

在创建网络时，可以指定`--ipv6`启用IPv6 的标志。不能有选择地禁用默认`bridge`网络上的IPv6支持。

## 启用从容器转发到外部网络

默认情况下，来自连接到默认桥接网络的容器的数据**不会**转发到外部网络。要启用转发，您需要更改两个设置。这些不是Docker命令，它们影响Docker主机的内核。

1. 配置Linux内核以允许IP转发。

   ```shell
   $ sudo sysctl net.ipv4.conf.all.forwarding=1
   ```

2. 将策略的`iptables` `FORWARD`策略从更改`DROP`为 `ACCEPT`。

   ```shell
   $ sudo iptables -P FORWARD ACCEPT
   ```

这些设置在重新启动时不会持续存在，因此需要将它们添加到启动脚本中。

## 使用默认桥接网络

默认`bridge`网络被视为Docker的传统网络，不建议用于生产。配置它是一个手动操作，它有 [技术上的缺点](https://docs.docker.com/network/bridge/#differences-between-user-defined-bridges-and-the-default-bridge)。

### 将容器连接到默认网桥

如果没有使用`--network`指定网络，并且指定了网络驱动`--driver`程序，则默认情况下容器已连接到默认`bridge`网络。连接到默认`bridge`网络的容器可以进行通信，但只能通过**IP地址**进行通信，除非它们使用[旧`--link`标志](https://docs.docker.com/network/links/)进行链接。

### 配置默认网桥

要配置默认`bridge`网络，请在`daemon.json`中指定选项。这里有一个`daemon.json`指定了几个选项的例子。只指定需要自定义的设置。

```json
{
  "bip": "192.168.1.5/24",
  "fixed-cidr": "192.168.1.5/25",
  "fixed-cidr-v6": "2001:db8::/64",
  "mtu": 1500,
  "default-gateway": "10.20.1.1",
  "default-gateway-v6": "2001:db8:abcd::89",
  "dns": ["10.20.1.2","10.20.1.3"]
}
```

重新启动Docker以使更改生效。

### 将IPv6与默认网桥一起使用

如果将Docker配置为支持IPv6（请参阅[使用IPv6](https://docs.docker.com/network/bridge/#use-ipv6)），则默认桥接网络也会自动配置为IPv6。与用户定义的网桥不同，不能在默认网桥上选择性地禁用IPv6。

### 操作流程

演示如何使用`bridge`自动为Docker设置默认网络。**这个网络不是生产系统的最佳选择**。在这个例子中，使用`alpine`在同一个Docker主机上启动两个不同的容器，并进行一些测试以了解它们如何相互通信。



