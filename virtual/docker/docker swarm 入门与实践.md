# docker swarm 

[TOC]

# 集群模式概述

当前版本的Docker包括*swarm模式，*用于本地管理称为群集的Docker引擎*群集*。使用Docker CLI创建群集，将应用程序服务部署到群集，并管理群体行为。

如果您之前使用的是Docker版本`1.12.0`，则可以使用[独立群集](https://docs.docker.com/swarm/)，但我们建议进行更新。

## 集群特点

- **与Docker Engine集成的集群管理：**使用Docker Engine CLI创建一群Docker引擎，可以在其中部署应用程序服务。不需要额外的编排软件来创建或管理群。
- **分散式设计：** Docker Engine在部署时不需要处理节点角色之间的差异，而是在运行时处理任何专业化。您可以使用Docker Engine部署这两种节点，管理员和工作人员。这意味着您可以从单个磁盘映像构建整个群集。
- **声明式服务模型：** Docker Engine使用声明式方法让您在应用程序堆栈中定义各种服务的所需状态。例如，您可能会描述一个由带有消息队列服务和数据库后端的Web前端服务组成的应用程序。
- **伸缩：**对于每个服务，您可以声明要运行的任务数量。当您向上或向下缩放时，swarm管理器会通过添加或删除任务来自动调整以保持所需的状态。
- **期望的状态协调：** swarm manager节点持续监视集群状态，并协调实际状态与表达期望状态之间的任何差异。例如，如果您设置了一个服务来运行容器的10个副本以及承载其中两个副本崩溃的工作计算机，则该管理器会创建两个新副本来替换发生崩溃的副本。swarm manager将新副本分配给正在运行且可用的工作人员。
- **多主机联网：**您可以为您的服务指定覆盖网络。swarm管理器在初始化或更新应用程序时自动为覆盖网络上的容器分配地址。
- **服务发现：** Swarm管理器节点为swarm中的每个服务分配一个唯一的DNS名称并负载平衡正在运行的容器。您可以通过群集中嵌入的DNS服务器查询群集中运行的每个容器。
- **负载平衡：**您可以将服务的端口暴露给外部负载平衡器。在群集内部，您可以指定如何在节点之间分发服务容器。
- **默认情况下为安全：**群中的每个节点都强制进行TLS相互认证和加密，以保护其自身与所有其他节点之间的通信。您可以选择使用自定义根证书或来自自定义根CA的证书。
- **滚动更新：**在推出时，您可以逐步将服务更新应用于节点。swarm管理器允许您控制服务部署到不同节点集之间的延迟。如果出现任何问题，您可以将任务回滚到以前版本的服务。

# 群模式关键概念

## 什么是集群？

嵌入在Docker Engine中的集群管理和编排功能是使用[swarmkit构建的](https://github.com/docker/swarmkit/)。Swarmkit是一个独立的项目，它实现了Docker的编排层，并直接在Docker中使用。

一个集群由多个Docker主机组成，这些主机以**集群模式**运行并充当管理 者（管理成员和委派）和工作 者（运行 [群集服务](https://docs.docker.com/engine/swarm/key-concepts/#services-and-tasks)）。给定的Docker主机可以是管理员、工作者或执行这两种角色。当您创建服务时，可以定义最佳状态（副本数量，可用的网络和存储资源，将服务暴露给外界等等）。Docker的工作是维持这个理想的状态。例如，如果工作节点变得不可用，Docker会在其他节点上调度该节点的任务。一个**任务** 是运行的容器集群服务的一部分，并通过集群管理节点管理，而不是一个独立的容器。

群集服务相对于独立容器的主要优势之一是**可以修改服务的配置**，包括连接的**网络和卷**，而**无需手动重新启动**服务。**Docker将更新配置，停止使用过时配置的服务任务，并创建与所需配置相匹配的新服务**。

当Docker以群集模式运行时，仍然可以在参与群集的任何Docker主机以及群集服务上运行独立容器。独立容器和群集服务之间的一个**主要区别**是，只有群集管理员可以管理群集，而独立容器可以在任何守护进程上启动。Docker守护进程可以作为管理者，工作者或两者参与群体。

与使用[Docker Compose](https://docs.docker.com/compose/)定义和运行容器的方式相同，您可以定义并运行swarm [堆栈](https://docs.docker.com/get-started/part5/)服务。

## 节点

一个**节点**是docker引擎参与集群的一个实例。您也可以将其视为Docker节点。您可以在单台物理计算机或云服务器上运行一个或多个节点，但生产群部署通常包括分布在多台物理机和云计算机上的Docker节点。

要将您的应用程序部署到群集，您需要向**管理节点**提交服务定义 。管理节点将称为[任务](https://docs.docker.com/engine/swarm/key-concepts/#services-and-tasks)的工作单元分派 给工作节点。

Manager节点还执行维护群体所需状态所需的编排和群集管理功能。经理节点选择一位领导者来执行编排任务。

**工作者节点**接收并执行从管理器节点分派的任务。默认情况下，管理器节点也可以将服务作为工作节点运行，但您可以将它们配置为独占运行管理器任务，并且是纯管理器节点。代理在每个工作节点上运行并报告分配给它的任务。工作节点通知管理节点其分配任务的当前状态，以便管理人员可以维护每个工作人员所需的状态。

## 服务和任务

一个**服务**是任务的定义，经理或工作节点上执行。它是群体系统的中心结构，也是群体与用户互动的主要根源。

在创建服务时，您可以指定要使用哪个容器映像以及要在正在运行的容器中执行哪些命令。

在**复制服务**模型中，swarm管理器根据您在所需状态中设置的比例在节点之间分配特定数量的副本任务。

对于**全局服务**，群集为群集中每个可用节点上的服务运行一个任务。

**任务**携带docker容器和在容器内部运行的命令。它是群体的原子调度单位。管理器节点根据服务规模中设置的副本数量将任务分配给工作节点。一旦任务分配给节点，它就不能移动到另一个节点。它只能在分配的节点上运行或失败。

## 负载均衡

swarm管理器使用**入口负载平衡**来公开要在群集外部提供的服务。swarm manager可以自动为服务分配一个**PublishedPort，**或者您可以为该服务配置一个PublishedPort。您可以指定任何未使用的端口。如果您不指定端口，那么swarm管理器将为该服务分配一个30000-32767范围内的端口。

外部组件（如云负载平衡器）可以访问群集中任何节点的PublishedPort上的服务，而不管该节点当前是否正在运行该服务的任务。集群路由中的**所有节点都会连接到正在运行的任务实例**。

Swarm模式有一个**内部DNS组件**，可自动为群集中的每个服务分配一个DNS条目。swarm管理器使用**内部负载平衡**根据服务的**DNS名称在群集内的服务之间分配请求**。

# docker swarm 基本命令

```sh
$ docker swarm -h
Usage:  docker swarm COMMAND

集群管理

Commands:
  ca          # 显示并轮着 CA 根证书
  init        # 初始化集群
  join        # 加入群体作为节点或管理
  join-token  # 管理连接令牌
  leave       # 离开集群
  unlock      # 解锁群集
  unlock-key  # 管理解锁密钥
  update      # 更新集群
```



## ca 证书

查看或轮转当前**群集CA证书**。该命令必须以**管理节点**为目标。

 

### 命令参数选项

---

| 选项，简写      | 默认        | 描述                                                      |
| --------------- | ----------- | --------------------------------------------------------- |
| `--ca-cert`     |             | 用于新集群的PEM格式的根CA证书的路径                       |
| `--ca-key`      |             | 用于新集群的PEM格式化根CA密钥的路径                       |
| `--cert-expiry` | `2160h0m0s` | 节点证书的有效期（ns \| us \| ms \| s \| m \| h）         |
| `--detach , -d` |             | 立即退出，而不是等待根旋转结束                            |
| `--external-ca` |             | 一个或多个证书签名端点的规范格式                          |
| `--quiet , -q`  |             | 控制进度输出                                              |
| `--rotate`      |             | 旋转集群CA - 如果未提供证书或密钥，则会生成新的证书或密钥 |

### `--rotate`

如果一个或多个**群集管理节点遭到入侵，建议使用根CA轮换**，以便这些管理节点**不能再连接到群集中的任何其他节点**或受其信任。或者，可以使用**根CA旋转来将群集CA控制权授予外部CA**，或从**外部CA获取控制权**。

`--rotate`标志不需要任何参数进行轮换，但可以选择**指定证书和密钥**，或者**证书和外部CA URL**，并且将使用这些参数代替自动生成的证书/密钥对。

由于根CA密钥应该保密，如果提供，通过CLI或API查看群集任何信息时都不可见。直到**所有注册节点**都旋转了他们的TLS证书后，根CA轮换才能完成。如果旋转**没有在合理的时间**内完成，请尝试运行`docker node ls --format '{{.ID}} {{.Hostname}} {{.Status}} {{.TLSStatus}}'`以查看是否有节点关闭或无法旋转TLS证书。

```sh
$ docker node ls --format '{{.ID}} {{.Hostname}} {{.Status}} {{.TLSStatus}}'
888gi7byqi5p7wtd2yv1z465i default Ready Ready
jx49db999birkx3myeorh9qvg my-vm-node-1 Ready Ready
```

### `--detach`

启动根CA旋转，但**不要等待完成或显示旋转的进度**。也就是**后台运行**模式！

### 示例

---

#### 生成证书

运行`docker swarm ca`没有任何选项的命令，以PEM格式查看当前的根CA证书。 

```sh
$ docker swarm ca
-----BEGIN CERTIFICATE-----
MIIBazCCARCgAwIBAgIUItJPmJ9DupeZ5VuoTjAV9mc/1/IwCgYIKoZIzj0EAwIw
EzERMA8GA1UEAxMIc3dhcm0tY2EwHhcNMTgwNTAxMTAyMjAwWhcNMzgwNDI2MTAy
MjAwWjATMREwDwYDVQQDEwhzd2FybS1jYTBZMBMGByqGSM49AgEGCCqGSM49AwEH
A0IABFzfcww26FTn48sBKJ28+U0/d0tAdSZ2MO70brm2fUhx8k6xxAZimDyY0rd3
sowuFJV76BnlojpH9s0r7xamd1qjQjBAMA4GA1UdDwEB/wQEAwIBBjAPBgNVHRMB
Af8EBTADAQH/MB0GA1UdDgQWBBSCih3fQ/zm/2ctt6PcynAACVkxpTAKBggqhkjO
PQQDAgNJADBGAiEA+6v6LGFkoJNIftNag3AigbQA3acDYZ/TbtXuMZAgF08CIQCb
CVjyhJCYGgNONh1c/RzztCXuYguJwWOjNxAx/n+aLA==
-----END CERTIFICATE-----
```



#### 轮转证书

传递`--rotate`标志（以及可选的 `--ca-cert`，连同一个`--ca-key`或 `--external-ca`参数标志），以便轮转当前集群根CA.

```sh
$ docker swarm ca --rotate
desired root digest:
  rotated TLS certificates:
  rotated CA certificates:
desired root digest: sha256:5ba0eaf2ea13378c8ebdf1ae4270f0b613157edd22ce4787f394b4ffd371ddc9
desired root digest: sha256:0e9754f56f57148cdb74730a0ff2da7a6194393c5a244baa744349ddd2b12a76
-----BEGIN CERTIFICATE-----
MIIBazCCARCgAwIBAgIUREpgWF1pBhouuU69P2t5tBnzb/cwCgYIKoZIzj0EAwIw
EzERMA8GA1UEAxMIc3dhcm0tY2EwHhcNMTgwNTA3MDI1ODAwWhcNMzgwNTAyMDI1
ODAwWjATMREwDwYDVQQDEwhzd2FybS1jYTBZMBMGByqGSM49AgEGCCqGSM49AwEH
A0IABCOKEBJDMyRwd6r2dBiMR+hL5PKyXZcD0OwyzqnqGbgHDkS7mNCOmASurVrI
glhfMVFqMsaOvmYX/0rKjePtdXijQjBAMA4GA1UdDwEB/wQEAwIBBjAPBgNVHRMB
Af8EBTADAQH/MB0GA1UdDgQWBBRDdoQTNQbma1hZy3OF3JYdvGlQ5DAKBggqhkjO
PQQDAgNJADBGAiEAyDRC8tOaNqxmaADcNXlj8JqGHFgyMJbDRq+ORCtaIk0CIQDR
er3hEhPf6PRvToBfD7Scy49EpBVkB9OCcNJqENg26g==
-----END CERTIFICATE-----
```

## init 初始化

初始化一个群 。该命令定位的docker引擎成为新创建的单节点集群中的管理节点。 

### 命令参数选项

---

| 选项，简写               | 默认           | 描述                                                         |
| ------------------------ | -------------- | ------------------------------------------------------------ |
| `--advertise-addr`       |                | 广播通知地址（格式：<ip \| interface> [：port]）             |
| `--autolock`             |                | 启用管理器自动锁定（需要解锁密钥才能启动停止的管理器）       |
| `--availability`         | `active`       | 节点的可用性（ “active”\|”pause”\|”drain” ——“活动”\|“暂停”\|“漏”） |
| `--cert-expiry`          | `2160h0m0s`    | 节点证书的有效期（ns \| us \| ms \| s \| m \| h）            |
| `--data-path-addr`       |                | 用于数据路径传输的地址或接口（格式：<ip \| interface>）      |
| `--dispatcher-heartbeat` | `5s`           | 调度心跳周期（ns \| us \| ms \| s \| m \| h）                |
| `--external-ca`          |                | 一个或多个证书签名端点的格式规范                             |
| `--force-new-cluster`    |                | 强制从当前状态创建一个新的群集                               |
| `--listen-addr`          | `0.0.0.0:2377` | 监听地址（格式：<ip \| interface> [：port]）                 |
| `--max-snapshots`        |                | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) 要保留的额外Raft快照的数量 |
| `--snapshot-interval`    | `10000`        | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) Raft快照之间的日志条目数 |
| `--task-history-limit`   | `5`            | 任务历史保留限制                                             |

#### `--autolock`

该标志可以使用加密密钥**自动锁定管理节点**。所有管理节点存储的私钥和数据将受到输出中打印的加密密钥的保护，如果没有它，将**无法访问**。因此，为了在重新启动后激活管理器，存储此密钥非常重要。密钥可以传递给`docker swarm unlock`重新激活管理器。自动锁定可以通过运行`docker swarm update --autolock=false`禁用 。在禁用它之后，加密密钥不再需要启动管理器，并且它将在没有用户干预的情况下自行启动。

#### `--cert-expiry`

该标志设置节点证书的**有效期**。

#### `--dispatcher-heartbeat`

该标志设置节点被告知使用的频率作为**报告健康状况**的时间段。

#### `--external-ca`

此标志设置群体使用**外部CA颁发节点**证书。该值采取的形式`protocol=X,url=Y`。值`protocol`指定应使用什么协议将签名请求发送到外部CA. 目前，唯一支持的值是`cfssl`。该URL指定了应该提交签名请求的端点。

#### `--force-new-cluster`

此标志强制作为单个节点管理器重新启动时丢失的管理的一部分的现有节点**不丢失数据**。

#### `--listen-addr`

该节点在此地址上侦听加入集群管理节点数据。默认是在0.0.0.0:2377上进行监听。也可以指定一个网络接口来侦听该接口的地址; 例如`--listen-addr eth0:2377`。

指定端口是可选的。如果该值为IP地址或接口名称，则将使用**默认端口2377**。

#### `--advertise-addr`

该标志指定将**通知给群集的其他成员进行API访问和覆盖网络的地址**。如果未指定，Docker将检查系统是否具有**单个IP地址**，并将该IP地址与侦听端口一起使用（请参阅参考资料`--listen-addr`）。如果系统有多个IP地址，则`--advertise-addr`必须指定该地址， 以便为经理间通信和覆盖网络选择正确的地址。

也可以指定一个网络接口来通告该接口的地址; 例如`--advertise-addr eth0:2377`。

指定端口是可选的。如果该值为IP地址或接口名称，则将使用默认端口2377。

#### `--data-path-addr`

此标志指定**全局范围网络驱动程序**将发布到其他节点的地址，以便到达在此节点上运行的容器。然后使用此参数可以将容器的**数据流量与群集的管理流量分开**。如果未指定，Docker将使用与**广播地址相同的IP地址或接口**。

#### `--task-history-limit`

此标志设置**任务历史保留**限制。

#### `--max-snapshots`

除了当前的**Raft快照**之外，该标志还设置要保留的旧Raft快照的数量。默认情况下，**不保留旧的快照**。该选项可用于**调试**，或用于存储swarm状态的旧快照以实现**灾难恢复**。

#### `--snapshot-interval`

该标志指定在**Raft快照之间允许的日志条目数量**。将其设置为**更高的数字将会减少快照次数**。快照缩小了Raft日志，并允许更高效地将状态转移给新管理节点。但是，经常拍摄快照会带来**性能**成本。

#### `--availability`

该标志指定**节点加入主设备时节点的可用性**。可能的可用性值`active`，`pause`或`drain`。

这个标志在某些情况下很有用。例如，群集可能希望具有**专用管理节点**，这些节点**不用作工作节点**。这可以通过传递`--availability=drain`来实现`docker swarm init`。

### 示例

---

#### 创建集群

```sh
$ docker swarm init --advertise-addr 192.168.99.121
Swarm initialized: current node (bvz81updecsj6wjz393c09vti) is now a manager.
To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-3pu6hszjas19xyp7ghgosyx9k8atbfcr8p2is99znpy26u2lkl-1awxwuwd3z9j1z3puu7rcgdbx \
    172.17.0.2:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

`docker swarm init`生成两个随机令牌，一个工作节点令牌和一个关键节点令牌。当新节点加入到群中时，该节点将根据传递给[群集加入](https://docs.docker.com/engine/reference/commandline/swarm_join/)的令牌作为工作节点或管理节点[加入](https://docs.docker.com/engine/reference/commandline/swarm_join/)。

创建群集后，可以使用[群集连接令牌](https://docs.docker.com/engine/reference/commandline/swarm_join_token/)显示或旋转该 [令牌](https://docs.docker.com/engine/reference/commandline/swarm_join_token/)。

## join 加入

将一个节点加入集群中。根据使用`--token`标志传递的令牌，该节点将作为管理节点或工作节点加入。如果传递管理令牌，则该节点将作为管理节点加入。如果传递工作令牌，则该节点将作为工作节点加入。

### 命令参数选项

---

| 选项，简写         | 默认           | 描述                                                         |
| ------------------ | -------------- | ------------------------------------------------------------ |
| `--advertise-addr` |                | 通知地址（格式：<ip \| interface> [：port]）                 |
| `--availability`   | `active`       | 节点的可用性（“活动”\|“暂停”\|“漏”— “active”\|”pause”\|”drain” ） |
| `--data-path-addr` |                | 用于数据路径传输的地址或接口（格式：<ip \| interface>）      |
| `--listen-addr`    | `0.0.0.0:2377` | 监听地址（格式：<ip \| interface> [：port]）                 |
| `--token`          |                | 进入集群的令牌                                               |

### `--listen-addr value`

如果该节点是经理，它将监听此地址上的入站群管理器流量。默认是在0.0.0.0:2377上进行监听。也可以指定一个网络接口来侦听该接口的地址; 例如`--listen-addr eth0:2377`。

指定端口是可选的。如果该值为裸IP地址或接口名称，则将使用默认端口2377。

加入现有群体时，此标志通常不是必需的。

### `--advertise-addr value`

此标志指定将通告给群集的其他成员进行API访问的地址。如果未指定，Docker将检查系统是否具有单个IP地址，并将该IP地址与侦听端口一起使用（请参阅参考资料 `--listen-addr`）。如果系统有多个IP地址，则`--advertise-addr` 必须指定该地址，以便为经理间通信和覆盖网络选择正确的地址。

也可以指定一个网络接口来通告该接口的地址; 例如`--advertise-addr eth0:2377`。

指定端口是可选的。如果该值为裸IP地址或接口名称，则将使用默认端口2377。

加入现有群体时，此标志通常不是必需的。如果您通过负载平衡器加入新节点，则应使用此标志来确保节点通告其IP地址，而不是负载平衡器的IP地址。

### `--data-path-addr`

此标志指定全局范围网络驱动程序将发布到其他节点的地址，以便到达在此节点上运行的容器。然后使用此参数可以将容器的数据流量与群集的管理流量分开。如果未指定，Docker将使用与广告地址相同的IP地址或接口。

### `--token string`

节点加入群体所需的秘密值

### `--availability`

该标志指定节点加入主设备时节点的可用性。可能的可用性值`active`，`pause`或`drain`。

这个标志在某些情况下很有用。例如，群集可能希望具有专用管理器节点，这些节点不用作工作者节点。这可以通过传递`--availability=drain`来实现`docker swarm join`。

 ### 示例

---

#### 加入管理节点

下面的示例演示了如何使用管理器令牌来加入管理节点。

```sh
$ docker swarm join --token SWMTKN-1-3pu6hszjas19xyp7ghgosyx9k8atbfcr8p2is99znpy26u2lkl-7p73s1dx5in4tatdymyhg9hu2 192.168.99.121:2377
This node joined a swarm as a manager.
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
dkp8vy1dq1kxleu9g4u78tlag *  manager2  Ready   Active        Reachable
dvfxp4zseq4s0rih1selh0d20    manager1  Ready   Active        Leader
```

一个集群最多只能有**3-7个管理节点**，因为大多数管理者必须可以使集群发挥作用。不打算参与此管理法定人数的节点应该以工作节点身份加入。管理员应该是具有**静态IP地址的稳定主机**。

#### 加入工作节点

下面的示例演示如何使用工作者令牌来加入工作节点。

```sh
$ docker swarm join --token SWMTKN-1-3pu6hszjas19xyp7ghgosyx9k8atbfcr8p2is99znpy26u2lkl-1awxwuwd3z9j1z3puu7rcgdbx 192.168.99.121:2377
This node joined a swarm as a worker.
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
7ln70fl22uw2dvjn2ft53m3q5    worker2   Ready   Active
dkp8vy1dq1kxleu9g4u78tlag    worker1   Ready   Active        Reachable
dvfxp4zseq4s0rih1selh0d20 *  manager1  Ready   Active        Leader
```

## join-token 令牌

管理加入令牌

### 命令参数选项

```sh
$ docker swarm join-token -h
Usage:  docker swarm join-token [OPTIONS] (worker|manager)

管理加入令牌

Options:
  -q, --quiet    # 只显示标记
      --rotate   # 轮转连接令牌
```



### 示例

---

#### 获取加入工作节点的token
```sh
$ docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0d3bbbyzap5snxrtfdlabfjkaeateg39qzcurj9xgtdnmu7r06-9ry3p245yzfpoyx8qekpanre3 192.168.99.100:2377
```

#### 获取加入管理节点的token
```sh
$ docker swarm join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0d3bbbyzap5snxrtfdlabfjkaeateg39qzcurj9xgtdnmu7r06-9qz1lxu17zx9ztajfza6hyv1z 192.168.99.100:2377
```

#### 查看token
```sh
$ docker swarm join-token manager -q
SWMTKN-1-0d3bbbyzap5snxrtfdlabfjkaeateg39qzcurj9xgtdnmu7r06-9qz1lxu17zx9ztajfza6hyv1z
```

#### 轮转
```sh
$ docker swarm join-token manager -q --rotate
SWMTKN-1-0d3bbbyzap5snxrtfdlabfjkaeateg39qzcurj9xgtdnmu7r06-d7cbnxkkru5p9wh3qxzfrcuwe    
```

## leave 离开

当在工作节点上运行此命令时，该工作节点离开群集。

可以使用`--force`管理器上的选项将其从群中删除。但是，这不会重新配置群体以确保有足够的管理节点维护群体中的法定人数。从群体中删除经理的安全方法是将其**降级为工作节点**，然后指定离开法定人数而不使用`--force`。仅在管理节点离开后不再使用集群的情况下使用，例如在单节点群中。

### 命令参数选项

---

| 选项，简写     | 默认 | 描述                         |
| -------------- | ---- | ---------------------------- |
| `--force , -f` |      | 强制此节点离开群集，忽略警告 |

### 示例

---

从管理节点查看集群节点：

```sh
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
7ln70fl22uw2dvjn2ft53m3q5    worker2   Ready   Active
dkp8vy1dq1kxleu9g4u78tlag    worker1   Ready   Active
dvfxp4zseq4s0rih1selh0d20 *  manager1  Ready   Active        Leader
```

要删除`worker2`，请从其`worker2`节点上执行以下命令：

```sh
$ docker swarm leave
Node left the default swarm.
```

该节点仍将出现在节点列表中，并标记为`down`。它不再影响群体操作，但是一长串`down`节点可能会混淆节点列表。要从列表中删除非活动节点，请使用该[`node rm`](https://docs.docker.com/engine/reference/commandline/node_rm/) 命令。

## unlock 解锁

使用用户提供的解锁密钥**解锁锁定的管理节点**。如果**自动锁定**设置处于打开状态，则在Docker守护程序重新启动后，必须使用此命令**重新激活**管理节点。解锁键在自动锁定启用时打印，也可从`docker swarm unlock-key`命令中使用。 

```sh
$ docker swarm unlock
Please enter unlock key:
```

## unlock-key 解锁秘钥

管理解锁密钥

 ### 命令参数选项

---

| 选项，简写     | 默认 | 描述       |
| -------------- | ---- | ---------- |
| `--quiet , -q` |      | 只显示令牌 |
| `--rotate`     |      | 旋转解锁键 |

### 示例

---

```sh
$ docker swarm unlock-key
no unlock key is set
```

## update 更新

用新的参数值更新群。该命令必须以**管理节点**为目标。

 ### 命令参数选项

---

| 名字，简写               | 默认        | 描述                                                         |
| ------------------------ | ----------- | ------------------------------------------------------------ |
| `--autolock`             |             | 更改管理自动锁定设置（true \| false）                        |
| `--cert-expiry`          | `2160h0m0s` | 节点证书的有效期（ns \| us \| ms \| s \| m \| h）            |
| `--dispatcher-heartbeat` | `5s`        | 调度心跳周期（ns \| us \| ms \| s \| m \| h）                |
| `--external-ca`          |             | 一个或多个证书签名端点的规范格式                             |
| `--max-snapshots`        |             | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) 要保留的额外Raft快照的数量 |
| `--snapshot-interval`    | `10000`     | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) Raft快照之间的日志条目数 |
| `--task-history-limit`   | `5`         | 任务历史保留限制                                             |

### 示例

---

```sh
$ docker swarm update --cert-expiry 720h
```

# docker node 基本命令

```sh
$ docker node -h
Usage:  docker node COMMAND

管理集群节点

Commands:
  demote      # 从集群中的管理节点中降级一个或多个节点
  inspect     # 在一个或多个节点上显示详细信息
  ls          # 集群节点列表
  promote     # 将一个或多个节点提升为群中的管理节点
  ps          # 列出在一个或多个节点上运行的任务，默认为当前节点
  rm          # 从群中删除一个或多个节点
  update      # 更新节点

```

## demote 降级

降级现有管理节点，使其不再是管理角色。

 ```sh
$ docker node demote my-vm-node-1
 ```

## promote 升级

将节点提升为经理。

 ```sh
$ docker node promote my-vm-node-1
 ```
## inspect 检查

返回有关节点的信息。默认情况下，该命令将所有结果呈现在JSON数组中。可以指定一个替代格式来为每个结果执行给定的模板。Go的 [文本/模板](http://golang.org/pkg/text/template/)包描述了格式的所有细节。

### 命令参数选项

---

| 选项，简写      | 默认 | 描述                       |
| --------------- | ---- | -------------------------- |
| `--format , -f` |      | 使用给定的Go模板格式化输出 |
| `--pretty`      |      | 以人性化的格式打印信息     |

### 示例

---

```sh
$ docker node inspect swarm-manager
$ docker node inspect --format '{{ .ManagerStatus.Leader }}' node-1
$ docker node inspect --pretty node-1
```

## ls 列表

列出Docker Swarm经理知道的所有节点。您可以使用`-f`或`--filter`标志进行过滤。有关可用过滤器选项的更多信息，请参阅[过滤](https://docs.docker.com/engine/reference/commandline/node_ls/#filtering)部分。

 ### 命令参数选项

---

| 名字，简写      | 默认 | 描述                     |
| --------------- | ---- | ------------------------ |
| `--filter , -f` |      | 根据提供的条件过滤输出   |
| `--format`      |      | 使用Go模板的漂亮打印节点 |
| `--quiet , -q`  |      | 只显示ID                 |

### 示例

---

#### 查看节点

```sh
$ docker node ls
ID                           HOSTNAME        STATUS  AVAILABILITY  MANAGER STATUS
1bcef6utixb0l0ca7gxuivsj0    swarm-worker2   Ready   Active
38ciaotwjuritcdtn9npbnkuz    swarm-worker1   Ready   Active
e216jshn25ckzbvmwlnh5jr3g *  swarm-manager1  Ready   Active        Leader
```

> **注意**：在上面的示例输出中，有一个隐藏列`.Self`，指示该节点是否与当前docker守护进程相同。 `*`（例如，`e216jshn25ckzbvmwlnh5jr3g *`）表示这个节点是当前的docker守护进程。

#### 过滤

过滤标志（`-f`或`--filter`）格式为“key = value”。如果有多个过滤器，则传递多个标志（例如`--filter "foo=bar" --filter "bif=baz"`）

目前支持的过滤器是：

- [id](https://docs.docker.com/engine/reference/commandline/node_ls/#id)
- [label](https://docs.docker.com/engine/reference/commandline/node_ls/#label)
- [membership](https://docs.docker.com/engine/reference/commandline/node_ls/#membership)
- [name](https://docs.docker.com/engine/reference/commandline/node_ls/#name)
- [role](https://docs.docker.com/engine/reference/commandline/node_ls/#role)

```sh
$ docker node ls -f id=1
# 标签与节点进行匹配
$ docker node ls -f "label=foo"
# membership过滤器相匹配的基础上一个存在的节点membership和一个值 accepted或pending。
$ docker node ls -f "membership=accepted"
# 将名称等于swarm-master字符串的节点进行匹配
$ docker node ls -f name=swarm-manager1
# 过滤器相匹配的基础上一个存在的节点role和一个值worker或manager
$ docker node ls -f "role=manager"
```

#### 格式化

格式化选项（`--format`）使用Go模板漂亮地打印节点输出。下面列出了Go模板的有效占位符：

| 占位符           | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| `.ID`            | 节点ID                                                       |
| `.Self`          | 守护进程的节点（`true/false`，`true`表示该节点与当前的docker守护进程相同） |
| `.Hostname`      | 节点主机名                                                   |
| `.Status`        | 节点状态                                                     |
| `.Availability`  | 节点可用性（“活动”，“暂停”或“漏” -- “active”, “pause”, or “drain” ） |
| `.ManagerStatus` | 节点的管理器状态                                             |
| `.TLSStatus`     | 节点的TLS状态（“准备就绪”或“需要轮换”具有由旧CA签署的TLS证书） |
| `.EngineVersion` | 引擎版本                                                     |

使用该`--format`选项时，该`node ls`命令将按照模板声明输出数据，或者在使用该 `table`指令时也包含列标题。

下面的示例使用的模板没有报头，并输出 `ID`，`Hostname`和`TLS Status`通过对所有节点冒号分隔的条目：

```sh
$ docker node ls --format "{{.ID}}: {{.Hostname}} {{.TLSStatus}}"
e216jshn25ckzbvmwlnh5jr3g: swarm-manager1 Ready
35o6tiywb700jesrt3dmllaza: swarm-worker1 Needs Rotation  
$ docker node ls --format "table {{.ID}}: {{.Hostname}} {{.TLSStatus}}"
```

## ps 查看任务

列出Docker 发现的节点上的所有任务。可以使用`-f`或`--filter`标志进行过滤。有关可用过滤器选项的更多信息，请参阅[过滤](https://docs.docker.com/engine/reference/commandline/node_ps/#filtering)部分。

 ### 命令参数选项

---

| 选项，简写      | 默认 | 描述                     |
| --------------- | ---- | ------------------------ |
| `--filter , -f` |      | 根据提供的条件过滤输出   |
| `--format`      |      | 使用Go模板的漂亮打印任务 |
| `--no-resolve`  |      | 不要将ID映射到名称       |
| `--no-trunc`    |      | 不要截断输出             |
| `--quiet , -q`  |      | 只显示任务ID             |

### 示例

---

#### 查看任务

```sh
$ docker node ps swarm-manager1
NAME                                IMAGE        NODE            DESIRED STATE  CURRENT STATE
redis.1.7q92v0nr1hcgts2amcjyqg3pq   redis:3.0.6  swarm-manager1  Running        Running 5 hours
```

#### 过滤

过滤标志（`-f`或`--filter`）格式为“key = value”。如果有多个过滤器，则传递多个标志（例如`--filter "foo=bar" --filter "bif=baz"`）

目前支持的过滤器是：

- [name](https://docs.docker.com/engine/reference/commandline/node_ps/#name)
- [id](https://docs.docker.com/engine/reference/commandline/node_ps/#id)
- [label](https://docs.docker.com/engine/reference/commandline/node_ps/#label)
- [desired-state](https://docs.docker.com/engine/reference/commandline/node_ps/#desired-state)

```sh
# 任务名称的全部或部分匹配
$ docker node ps -f name=redis swarm-manager1
# 过滤器匹配任务的ID
$ docker node ps -f id=bg8c07zzg87di2mufeq51a2qp swarm-manager1
# 标签匹配
$ docker node ps -f "label=usage"
# 该desired-state过滤器可以取值running，shutdown，或accepted。
$ docker node ps -f desired-state=running node1
```

## rm 删除

从管理器节点运行时，从群中删除指定的节点。

### 命令参数选项

---

| 选项，简写     | 默认 | 描述                   |
| -------------- | ---- | ---------------------- |
| `--force , -f` |      | 强制从群中删除一个节点 |

### 示例

---

#### 删除停止节点

```sh
$ docker node rm swarm-node-02
Node swarm-node-02 removed from swarm
```

#### 删除运行节点

从群中删除指定的节点，但只有当节点处于停机状态时才会这样。如果**尝试删除活动节点，将收到错误消息**：

```sh
$ docker node rm swarm-node-03
Error response from daemon: rpc error: code = 9 desc = node swarm-node-03 is not
down and can't be removed

$ docker node rm swarm-node-03 -f
```

#### 强行删除不可访问的节点

如果**失去对工作节点的访问权限或需要将其关闭**，因为它已被破坏或行为不如预期，则可以使用`--force`选项。这可能会导致*暂时错误或中断**，具体取决于节点上正在运行的任务的类型。

```sh
$ docker node rm --force swarm-node-03
Node swarm-node-03 removed from swarm
```

管理员节点**必须先降级到工作节点**（使用`docker node demote`），然后才能将其从群集中删除。

## update 更新

更新有关节点的元数据，例如其可用性，标签或角色。

### 命令参数选项

---

| 选项，简写       | 默认 | 描述                                 |
| ---------------- | ---- | ------------------------------------ |
| `--availability` |      | 节点的可用性（“活动”\|“暂停”\|“漏”） |
| `--label-add`    |      | 添加或更新节点标签（key = value）    |
| `--label-rm`     |      | 删除节点标签（如果存在）             |
| `--role`         |      | 节点的角色（“worker”\|“manager”）    |

### 示例

---

使用节点标签将元数据添加到swarm节点。您可以将节点标签指定为具有空值的键：

```sh
$ docker node update --label-add foo worker1
```

要将多个标签添加到节点，请`--label-add`为每个标签传递标志：

```sh
$ docker node update --label-add foo --label-add bar worker1
```

在[创建服务时](https://docs.docker.com/engine/reference/commandline/service_create/)，可以将节点标签用作约束。约束限制调度程序为服务部署任务的节点。

例如，要添加`type`标签以标识调度程序应该部署消息队列服务任务的节点：

```sh
$ docker node update --label-add type=queue worker1
```

为节点设置的标签`docker node update`仅适用于群内的节点实体。

# docker service 基本命令

```sh
$ docker service -h
Usage:  docker service COMMAND

服务管理

Commands:
  create      # 创建一个新服务
  inspect     # 检查服务查看详细信息
  logs        # 获取服务或任务的日志
  ls          # 服务列表
  ps          # 列出一项或多项服务的任务
  rm          # 删除一项或多项服务
  rollback    # 将更改还原为服务的配置
  scale       # 扩展一个或多个复制服务
  update      # 更新服务
```

## create 创建

按照指定的参数描述创建服务。必须在管理节点上运行此命令。

### 命令参数选项

---

| 选项，简写                     | 默认         | 描述                                                         |
| ------------------------------ | ------------ | ------------------------------------------------------------ |
| `--config`                     |              | [API 1.30+](https://docs.docker.com/engine/api/v1.30/) 指定配置以暴露给服务 |
| `--constraint`                 |              | 约束                                                         |
| `--container-label`            |              | 容器标签                                                     |
| `--credential-spec`            |              | [API 1.29+](https://docs.docker.com/engine/api/v1.29/) 托管服务帐户的凭证规范（仅限Windows） |
| `--detach , -d`                |              | [API 1.29+](https://docs.docker.com/engine/api/v1.29/) 立即退出，而不是等待服务收敛 |
| `--dns`                        |              | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) 设置自定义DNS服务器 |
| `--dns-option`                 |              | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) 设置DNS选项 |
| `--dns-search`                 |              | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) 设置自定义DNS搜索域 |
| `--endpoint-mode`              | `vip`        | 端点模式（vip或dnsrr）                                       |
| `--entrypoint`                 |              | 覆盖图像的默认入口点                                         |
| `--env , -e`                   |              | 设置环境变量                                                 |
| `--env-file`                   |              | 读入环境变量文件                                             |
| `--generic-resource`           |              | 用户定义的资源                                               |
| `--group`                      |              | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) 为容器设置一个或多个补充用户组 |
| `--health-cmd`                 |              | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) 运行以检查运行状况的命令 |
| `--health-interval`            |              | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) 运行检查之间的时间（ms \| s \| m \| h） |
| `--health-retries`             |              | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) 需要报告不健康的连续失败 |
| `--health-start-period`        |              | [API 1.29+](https://docs.docker.com/engine/api/v1.29/) 在重新计数到不稳定（ms \| s \| m \| h）之前，容器要初始化的起始周期 |
| `--health-timeout`             |              | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) 允许一次检查运行的最大时间（ms \| s \| m \| h） |
| `--host`                       |              | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) 设置一个或多个自定义主机到IP映射（主机：IP） |
| `--hostname`                   |              | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) 容器主机名 |
| `--isolation`                  |              | [API 1.35+](https://docs.docker.com/engine/api/v1.35/) 服务容器隔离模式 |
| `--label , -l`                 |              | 服务标签                                                     |
| `--limit-cpu`                  |              | 限制CPU                                                      |
| `--limit-memory`               |              | 限制内存                                                     |
| `--log-driver`                 |              | 记录驱动程序的服务                                           |
| `--log-opt`                    |              | 记录驱动程序选项                                             |
| `--mode`                       | `replicated` | 服务模式（复制或全局）                                       |
| `--mount`                      |              | 将文件系统挂载附加到服务                                     |
| `--name`                       |              | 服务名称                                                     |
| `--network`                    |              | 网络附件                                                     |
| `--no-healthcheck`             |              | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) 禁用任何容器指定的HEALTHCHECK |
| `--no-resolve-image`           |              | [API 1.30+](https://docs.docker.com/engine/api/v1.30/) 不要查询注册表以解析图像摘要和支持的平台 |
| `--placement-pref`             |              | [API 1.28+](https://docs.docker.com/engine/api/v1.28/) 添加展示位置偏好设置 |
| `--publish , -p`               |              | 将端口发布为节点端口                                         |
| `--quiet , -q`                 |              | 抑制进度输出                                                 |
| `--read-only`                  |              | [API 1.28+](https://docs.docker.com/engine/api/v1.28/) 将容器的根文件系统挂载为只读 |
| `--replicas`                   |              | 任务数量                                                     |
| `--reserve-cpu`                |              | 预留CPU                                                      |
| `--reserve-memory`             |              | 保留内存                                                     |
| `--restart-condition`          |              | 满足条件时重新启动（“none”\|“on-failure”\|“any”）（默认为“any”） |
| `--restart-delay`              |              | 重启尝试之间的延迟（ns \| us \| ms \| s \| m \| h）（默认5秒） |
| `--restart-max-attempts`       |              | 放弃前的最大重启次数                                         |
| `--restart-window`             |              | 用于评估重新启动策略的窗口（ns \| us \| ms \| s \| m \| h）  |
| `--rollback-delay`             |              | [API 1.28+](https://docs.docker.com/engine/api/v1.28/) 任务回滚之间的延迟（ns \| us \| ms \| s \| m \| h）（默认为0） |
| `--rollback-failure-action`    |              | [API 1.28+](https://docs.docker.com/engine/api/v1.28/) 回滚失败的操作（“暂停”\|“继续”）（默认“暂停”） |
| `--rollback-max-failure-ratio` |              | [API 1.28+](https://docs.docker.com/engine/api/v1.28/) 在回滚期间容忍的失败率（默认0） |
| `--rollback-monitor`           |              | [API 1.28+](https://docs.docker.com/engine/api/v1.28/) 每个任务回滚之后监视失败的持续时间（ns \| us \| ms \| s \| m \| h）（默认5秒） |
| `--rollback-order`             |              | [API 1.29+](https://docs.docker.com/engine/api/v1.29/) 回滚顺序（“start-first”\|“stop-first”）（默认为“stop-first”） |
| `--rollback-parallelism`       | `1`          | [API 1.28+](https://docs.docker.com/engine/api/v1.28/) 同时回滚的最大任务数量（0一次全部回滚） |
| `--secret`                     |              | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) 指定泄露给服务的秘密 |
| `--stop-grace-period`          |              | 强制杀死一个容器之前等待的时间（ns \| us \| ms \| s \| m \| h）（默认10秒） |
| `--stop-signal`                |              | [API 1.28+](https://docs.docker.com/engine/api/v1.28/) 停止容器的信号 |
| `--tty , -t`                   |              | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) 分配伪TTY |
| `--update-delay`               |              | 更新之间的延迟（ns \| us \| ms \| s \| m \| h）（默认为0）   |
| `--update-failure-action`      |              | 更新失败的操作（“暂停”\|“继续”\|“回滚”）（默认“暂停”）       |
| `--update-max-failure-ratio`   |              | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) 更新期间[允许的](https://docs.docker.com/engine/api/v1.25/)失败率（默认值为0） |
| `--update-monitor`             |              | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) 每次任务更新后监视失败的持续时间（ns \| us \| ms \| s \| m \| h）（默认5秒） |
| `--update-order`               |              | [API 1.29+](https://docs.docker.com/engine/api/v1.29/) 更新顺序（“start-first”\|“stop-first”）（默认为“stop-first”） |
| `--update-parallelism`         | `1`          | 同时更新的最大任务数（0个一次全部更新）                      |
| `--user , -u`                  |              | 用户名或UID（格式：<name \| uid> [：<group \| gid>]）        |
| `--with-registry-auth`         |              | 向注册代理发送注册表认证详细信息                             |
| `--workdir , -w`               |              | 容器内的工作目录                                             |

### 示例

---

#### 创建服务

---

```sh
$ docker service create --name redis redis:3.0.6
dmu1ept4cxcfe8k8lhtux3ro3

$ docker service create --mode global --name redis2 redis:3.0.6
a8q9dasaafudfs8q8w32udass

$ docker service ls
ID            NAME    MODE        REPLICAS  IMAGE
dmu1ept4cxcf  redis   replicated  1/1       redis:3.0.6
a8q9dasaafud  redis2  global      1/1       redis:3.0.6
```

#### 使用私人注册表中的图像创建服务

---

如果图像在需要登录的私人注册表中可用，请在登录后使用该 `--with-registry-auth`标志`docker service create`。如果图像存储在`registry.example.com`哪个私有注册表中，请使用类似以下的命令：

```sh
$ docker login registry.example.com
$ winpty docker login -u xxx registry.cn-hangzhou.aliyuncs.com
$ docker login --username=xxx -p xxxx registry.cn-hangzhou.aliyuncs.com

$ docker service  create \
  --with-registry-auth \
  --name my_service \
  registry.cn-hangzhou.aliyuncs.com/acme/my_image:latest
```

这使用加密的WAL日志将登录令牌从本地客户端传递到部署服务的群集节点。有了这些信息，这些节点就能够登录到注册表并提取图像。

#### 创建5个副本任务的服务

---

使用`--replicas`标志设置复制服务的副本任务数量。以下命令`redis`使用`5`副本任务创建服务：

```sh
$ docker service create --name redis --replicas=5 redis:latest
4cdgfyky7ozwh3htjfw0d12qv
```

以上命令为服务设置了**所需**的任务数量。即使该命令立即返回，但服务的实际伸缩可能需要一些时间。`REPLICAS`列显示服务的**实际**和**期望**数量的副本任务。

在以下示例中，所需的状态是 `5`副本，但当前的`RUNNING`任务数是`3`：

```sh
$ docker service ls
ID            NAME   MODE        REPLICAS  IMAGE
4cdgfyky7ozw  redis  replicated  3/5       redis:3.0.7
```

一旦创建了所有任务并且`RUNNING`**实际的任务数量等于所需的数量**：

```sh
$ docker service ls
ID            NAME   MODE        REPLICAS  IMAGE
4cdgfyky7ozw  redis  replicated  5/5       redis:3.0.7
```

#### 创建私密服务

---

使用`--secret`标志可以让容器访问 [私密](https://docs.docker.com/engine/reference/commandline/secret_create/)。

创建一个指定私密的服务：

```sh
$ docker service create --name redis --secret secret.json redis:latest
4cdgfyky7ozwh3htjfw0d12qv
```

创建一个指定秘密、目标、用户/组ID和模式的服务：

```sh
$ docker service create --name redis \
    --secret source=ssh-key,target=ssh \
    --secret source=app-key,target=app,uid=1000,gid=1001,mode=0400 \
    redis:latest

4cdgfyky7ozwh3htjfw0d12qv
```

授予服务访问多个秘密，使用多个`--secret`标志。

秘密位于`/run/secrets`容器中。如果未指定目标，则秘密的名称将用作容器中的内存文件。如果指定了目标，那将是文件名。在上面的例子中，将创建两个文件：`/run/secrets/ssh`以及 `/run/secrets/app`指定的每个秘密目标。

#### 使用滚动更新策略创建服务

---

```sh
$ docker service create \
  --replicas 10 \
  --name redis \
  --update-delay 10s \
  --update-parallelism 2 \
  redis:latest
```

当运行[服务更新时](https://docs.docker.com/engine/reference/commandline/service_update/)，调度程序一次最多更新2个任务，并在`10s`更新之间更新。有关更多信息，请参阅[滚动更新教程](https://docs.docker.com/engine/swarm/swarm-tutorial/rolling-update/)。

#### 设置环境变量

---

这为服务中的所有任务设置了一个环境变量。例如：

```sh
$ docker service create \
  --name redis_2 \
  --replicas 5 \
  --env MYVAR=foo \
  redis:latest
```

要指定多个环境变量，请指定多个`--env`标志，每个标志都有一个单独的键 - 值对。

```sh
$ docker service create \
  --name redis_2 \
  --replicas 5 \
  --env MYVAR=foo \
  --env MYVAR2=bar \
  redis:latest
```

查看环境变量

```sh
$ docker service inspect redis_2 --format "{{json .Spec.TaskTemplate.ContainerSpec.Env}}"
["MYVAR=foo","MYVAR2=bar"]
```

#### 创建具有特定主机名的服务

---

此选项将docker服务容器主机名设置为特定的字符串。例如：

```sh
$ docker service create --name redis --hostname myredis redis:latest
```

#### 在服务上设置元数据

---

标签是`key=value`将元数据应用于服务的一对。用两个标签标记服务：

```sh
$ docker service create \
  --name redis_2 \
  --label com.example.foo="bar"
  --label bar=baz \
  redis:latest
```

#### 添加挂载，卷或内存文件系统

---

Docker支持三种不同的挂载方式，它们允许容器在主机操作系统或内存文件系统上读取或写入文件或目录。这些类型是**数据卷**（通常简称为卷），**绑定挂载**和**tmpfs**。

**绑定挂载**使可被挂载在容器内的主机上的文件或目录。绑定挂载可以是**只读的或读写**的。例如，容器可能通过主机的绑定挂载来**共享主机的DNS信息**，`/etc/resolv.conf`或者容器可能会将日志写入其主机的`/var/log/myContainerLogs`目录。如果使用绑定挂载并且主机和容器具有不同的权限，访问控制或其他此类详细信息，则会遇到**可移植性**问题。

**卷是一种机制**，用于将容器所需的持久性数据与用于创建容器的映像和主机分离。命名卷由Docker创建和管理，即使当前没有容器正在使用它，命名卷仍然存在。命名卷中的数据可以在容器和主机之间共享，也可以在多个容器之间共享。Docker使用*卷驱动*来创建，管理和安装卷。您可以使用Docker命令备份或恢复卷。

**tmpfs**在容器内安装tmpfs以获取易失性数据。

考虑一下你的图像启动一个轻量级web服务器的情况。您可以将该图像用作基础图像，复制网站的HTML文件并将其打包到另一个图像中。每次您的网站更改时，您都需要更新新映像并重新部署为您的网站提供服务的所有容器。更好的解决方案是将网站存储在每个Web服务器容器启动时附加的命名卷中。要更新网站，只需更新指定的卷。

下表介绍了适用于服务中绑定安装和命名卷的选项：

| 选项                                | 是否必须             | 描述                                                         |
| ----------------------------------- | -------------------- | ------------------------------------------------------------ |
| **types**                           |                      | mount的类型可以是`volume`，`bind`或`tmpfs`。如果没有指定类型，则默认为`volume`。<br/>`volume`：将[托管卷](https://docs.docker.com/engine/reference/commandline/volume_create/) 装入容器。<br/>`bind`：将主机上的目录或文件绑定到容器中。<br/>`tmpfs`：在容器中安装一个tmpfs |
| **src** or **source**               | `type=bind`<br/>必须 | `type = volume`：`src`是指定卷名称的可选方式（例如，`src = my-volume`）。如果指定的**卷不存在，则会自动创建**。如果未指定`src，`则会为该卷指定一个**随机名称**，该名称在**主机上保证是唯一**的，但可能**不是群集范围内唯一**的。随机命名的卷与其容器具有相同的生命周期，并在**容器被销毁时销毁**（这是在` service update `时，或者扩展或重新平衡服务时）<br/>`type = bind`：`src`是必需的，并指定要绑定挂载的文件或目录的绝对路径（例如，`src = / path / on / host /`）。如果文件或目录不存在，则会产生错误。<br/>`type = tmpfs`：`src`不受支持。 |
| **dst** /**destination**/**target** | 是                   | 装入容器内的路径，例如`/some/path/in/container/`。如果路径**不存在**于容器的文件系统中，则引擎会在挂载卷或绑定挂载之前在指定位置**创建**一个目录。 |
| **readonly** or **ro**              |                      | 除非在安装挂载或卷时提供`只读`选项，否则引擎将挂接和卷默认设置为`读写`。<br/> `true`或`1`或没有值：只读。<br/>`false`或`0`：读写。 |
| **consistency** **一致性**          |                      | 挂载的一致性要求<br/>`default`：相当于`consistent`。<br/>`consistent`：完全一致。容器运行时和主机始终保持相同的安装视图。<br/>`cached`：主机的装载视图是权威的。在主机上进行的更新在容器内可见之前可能会有延迟。<br/>`delegated`：容器运行时的挂载视图是权威的。在容器中进行的更新在主机上可见之前可能会有延迟。 |

##### **挂载传播**

挂载传播是指在给定的绑定挂载或命名卷中创建的**挂载**是否可以**传播到该挂载的副本**。考虑一个挂载点`/mnt`，它也被挂载`/tmp`。该状态设置控制是否启用挂载`/tmp/a`也可用`/mnt/a`。每个传播设置都有一个**递归对应点**。在递归的情况下，考虑它`/tmp/a`也被挂载为`/foo`。传播设置控制是否`/mnt/a`和/或`/tmp/a`将存在。

`bind-propagation`选项默认`rprivate`为绑定挂载和卷挂载，并且**只能为绑定挂载**进行配置。换句话说，**命名卷不支持绑定传播**。

- **shared**：原始挂载的子安装会**暴露给副本**挂载，并且副挂载的子挂载也会传播到原始挂载。
- **slave**：类似于共享挂载，但仅限于**一个方向**。如果原始挂载存在一个子挂载，则副本挂载可以看到它。但是，如果副本挂载公开了子挂载，则原始挂载无法看到它。
- **private**：挂载是私有的。其中的子挂载不会暴露给副本挂载，并且副挂载的子挂载不会暴露给原始挂载。
- **rshared**：与共享相同，但传播也扩展到嵌套在任何原始或副本挂载点内的挂载点。
- **rslave**：与之相同`slave`，但传播也扩展到嵌套在任何原始或副本挂载点内的挂载点。
- **rprivate**：默认。与之相同`private`，这意味着原始或副本挂载点内的任何位置都不会以任何方向传播。

有关绑定传播的更多信息，请参阅[共享子树](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)的 [Linux内核文档](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)。

##### **命名卷的选项**

以下选项只能用于命名卷（`type=volume`）：

| 选项              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| **volume-driver** | 用于卷的卷驱动程序**插件的名称**。如果该卷不存在，则默认为 **local**，以使用本地卷驱动程序创建卷。 |
| **volume-label**  | 创建时应用于卷的一个或多个自定义元数据（“标签”）。<br/>例如， `volume-label = mylabel = hello-world，my-other-label = hello-mars`。<br/>有关标签的更多信息，请参阅 [应用自定义元数据](https://docs.docker.com/engine/userguide/labels-custom-metadata/)。 |
| **volume-nocopy** | 默认情况下，如果您将空卷附加到容器，并且文件或目录已经存在于容器中的安装路径（`dst`）中，则引擎会将这些文件和目录**复制到卷中**，以便主机访问它们。设置`volume-nocopy`以**禁止**将文件从容器的文件系统复制到卷并挂载空卷。 <br/>值是可选的：  <br/> `true` or `1`： 如果您不提供值，则为默认值。禁用复制。<br/> `false` or `0：` 启用复制 |
| **volume-opt**    | 特定于给定卷驱动程序的**选项，将在创建卷时传递给驱动程序**。选项以逗号分隔的键/值对列表形式提供，例如， `volume-opt = some-option = some-value，volume-opt = some-other-option = some-other-value`。<br/>有关给定驱动程序的可用选项，请参阅该驱动程序的文档。 |

##### **TMPFS的选项**

以下选项只能用于`tmpfs mounts`（`type=tmpfs`）;

| 选项           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| **tmpfs-size** | tmpfs的大小，以字节为单位。Linux中默认无限制。               |
| **tmpfs-mode** | tmpfs的八进制文件模式。（例如`“700”`或`“0700”）`。在Linux中默认为`“1777”`。 |

##### **`--MOUNT`和`--VOLUME`的区别**

`--mount`标志支持由支持的大多数选项`-v` 或`--volume`标志`docker run`，有一些重要的例外情况：

- 该`--mount`标志允许您指定音量驱动器和音量驱动程序选项*每卷*，而无需预先创建的卷。相反， `docker run`您可以使用该`--volume-driver`标志指定由所有卷共享的单个卷驱动程序。
- 该`--mount`标志允许您在卷创建之前指定卷的自定义元数据（“标签”）。
- 在使用`--mount`时`type=bind`，主机路径必须引用主机上的*现有* 路径。路径将不会为您创建，如果路径不存在，服务将失败并显示错误。
- 该`--mount`标志不允许您重新标记用于标记的卷`Z`或`z`标志`selinux`。

# 集群的应用



经理、管理者、管理员、管理器-管理节点

工作人员、工作者——工作节点

群集-集群

群体-集群

您-你

标志-选项

映像、图像、图片-镜像

秘密-私密