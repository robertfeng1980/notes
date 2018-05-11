# docker swarm 集群的应用

上面介绍过集群的创建和应用，这里详细介绍集群的原理和流程，以及需要注意的问题。

# 在群集模式下运行Docker引擎

当首次安装并开始使用Docker Engine时，群集模式默认处于禁用状态。当启用群集模式时，将使用通过`docker service`命令管理的服务概念。 

当在本地计算机上以群集模式运行引擎时，可以根据自己创建的映像或其他可用映像来创建和测试服务。在生产环境中，群集模式提供具有**群集管理功能**的**容错平台**，以保持服务正常运行。 

当运行命令**创建群集**时，Docker引擎开始以**群集模式运行**。运行[`docker swarm init`](https://docs.docker.com/engine/reference/commandline/swarm_init/) 以在**当前节点**上**创建单节点集群**。引擎的工作流程如下：

- 将当前节点切换到群集模式。
- 创建一个名为`default`的集群。
- 指定**当前节点**作为该群的**领导者管理者节点**。
- 用机器**主机名**命名节点。
- 配置管理器在**端口2377**上监听活动网络接口。
- 将**当前节点设置为`Active`可用性**，这意味着它可以从调度程序**接收任务**。
- 为参与群体的引擎**启动**一个**内部分布式数据存储**，以维护群集及其上运行的所有服务的一致**视图**。
- 默认情况下，为群体生成一个**自签名的根CA**。
- 默认情况下，为worker和manager节点**生成令牌**以加入群集。
- 创建一个**覆盖网络`ingress`**，网络命名为发布群体外部的服务端口。

输出用于`docker swarm init`提供将新工作节点加入群集时使用的连接命令：

```sh
$ docker swarm init
Swarm initialized: current node (dxn1zf6l61qsb1josjja83ngz) is now a manager.
To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

## 配置通知地址

管理器节点使用**通知地址**允许集群中的**其他节点访问Swarmkit API并覆盖网络**。集群中的**其他节点**必须能够在其**广播**地址上**访问管理器节点**。

如果**未指定**通知地址，则Docker将**检查**系统是否具有**单个IP地址**。如果存在单个IP地址，默认情况下Docker使用侦听端口`2377`的IP地址 。如果系统有**多个IP**地址，则必须**指定确切**的`--advertise-addr`以启用管理员间通信和覆盖网络：

```sh
$ docker swarm init --advertise-addr <MANAGER-IP>
```

如果**其他节点**到达**第一个管理器节点**的地址与管理员认为的**地址不同**，那么还必须指定`--advertise-addr`。例如，在跨越不同区域的云设置中，主机具有用于在该区域内访问的**内部地址**以及用于从该区域外访问的**外部地址**。在这种情况下，使用`--advertise-addr`指定外部地址，以便节点可以将信息传播到随后连接到节点的其他节点。

## 查看加入命令或更新加入令牌

节点需要一个**秘密的令牌**来加入群。工作节点的标记与管理器节点的标记不同。节点在加入群时只使用**连接**令牌。在节点已经加入群体之后**旋转连接令牌**不会影响节点的群集**成员资格**。令牌轮转可确保任何**尝试加入**群集的**新节点**都不能使用**旧令牌**。

要检索包含工作节点的连接令牌的连接命令，请运行：

```sh
$ docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377

This node joined a swarm as a worker.
```

要查看管理器节点的连接命令和令牌，请运行：

```sh
$ docker swarm join-token manager

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-59egwe8qangbzbqb3ryawxzk3jn97ifahlsrw01yar60pmkr90-bdjfnkcflhooyafetgjod97sz \
    192.168.99.100:2377
```

通过`--quiet`标志只打印令牌：

```sh
$ docker swarm join-token --quiet worker

SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c
```

**加入令牌时要小心，因为它们是加入群集所必需的秘钥**。特别是将一个检查秘密到版本控制是一个不好的做法，因为它允许任何有权访问应用**程序源代码**的人员向群体添加新节点。**管理令牌特别敏感**，因为它们允许新的管理节点加入并获得**整个群体的控制权**。

建议在以下情况下**轮换联合令牌**：

- 如果令牌偶然签入版本控制系统，**群聊或意外打印到日志**中。
- 如果怀疑某个节点已**受到攻击**。
- 如果你想保证**没有新的节点可以加入**群。

此外，对包括群集加入令牌在内的任何秘密实施**定期轮换**计划是最佳做法。我们建议您至少**每6个月**轮换一次令牌。

运行`swarm join-token --rotate`使**旧的令牌失效并生成新的令牌**。指定是否要旋转`worker`或`manager` 节点的标记：

```sh
# docker swarm join-token  --rotate manager
$ docker swarm join-token  --rotate worker
To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-2kscvs0zuymrsc9t0ocyy1rdns9dhaodvpl639j2bqx55uptag-ebmn5u927reawo27s3azntd44 \
    192.168.99.100:2377
```

# 将节点加入集群

当第一次创建swarm时，将Docker Engine（引擎）开启swarm模式。要充分利用集群模式，可以向集群添加节点：

- **添加工作节点会增加容量**。将服务部署到群集时，引擎会调度可用节点上的任务，无论它们是工作节点还是管理节点。当将工作节点添加到集群中时，可以增加集群的**规模**以处理任务，而不会影响管理节点的**一致性**。
- **管理节点增加了容错能力**。Manager节点为集群执行**编排和群集管理**功能。在管理节点中，**单个管理领导节点**执行编排任务。如果一个管理节点**出现故障**，其余的管理者节点会**选举一个新的领导者**并**恢复**协调和**维护**群体状态。默认情况下，**管理节点也运行任务**。

在将节点添加到群集之前，必须在主机上安装Docker Engine 1.12或更高版本。

Docker引擎根据提供给`docker swarm join`命令的**连接令牌**加入群集。该节点仅在**连接时**使用该令牌。如果**随后**旋转该标记，则**不会影响现有**的群集节点。请参考以[群集模式运行Docker Engine](https://docs.docker.com/engine/swarm/swarm-mode/#view-the-join-command-or-update-a-swarm-join-token)。

## 作为工作节点加入

要查看包含工作节点的连接令牌的连接命令，请在管理节点上运行以下命令：

```sh
$ docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377
```

在`worker`节点上的输出运行命令加入群集：

```sh
$ docker swarm join \
  --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
  192.168.99.100:2377

This node joined a swarm as a worker.
```

`docker swarm join`命令执行以下操作：

- 将当前节点上的Docker引擎**切换到群集**模式。
- 向管理节点**申请TLS证书**。
- 用机器**主机名命名节点**
- 基于群集**令牌**将当前节点**加入**管理节点监听地址上的群集。
- 将**当前节点设置为`Active`可用性**，这意味着它可以从调度程序**接收任务**。
- 将`ingress`覆盖网络**扩展**到**当前节点**。

## 作为管理节点加入


当运行`docker swarm join`并传递管理器令牌时，Docker引擎将**切换到群集模式**，与工作节点相同。管理节点也参与Raft共识。新的节点应该是`Reachable`，但现有的经理仍然是群体`Leader`。

Docker为每个集群推荐**三个或五个**管理节点来实现**高可用性**。由于群模式管理节点使用Raft共享数据，因此**必须有奇数个管理器**。只要**超过一半**的管理节点的法定数量**可用**，群集可以继续运行。

有关集群管理员和管理集群的更多详细信息，请参阅 [管理和维护一群Docker引擎](https://docs.docker.com/engine/swarm/admin_guide/)。

要查看包含**管理节点**的连接令牌的连接命令，请在管理器节点上运行以下命令：

```sh
$ docker swarm join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-61ztec5kyafptydic6jfc1i33t37flcl4nuipzcusor96k7kby-5vy9t8u35tuqm7vh67lrz9xp6 \
    192.168.99.100:2377
```

在**准**管理节点的输出的运行命令以加入群集：

```sh
$ docker swarm join \
  --token SWMTKN-1-61ztec5kyafptydic6jfc1i33t37flcl4nuipzcusor96k7kby-5vy9t8u35tuqm7vh67lrz9xp6 \
  192.168.99.100:2377

This node joined a swarm as a manager.
```

# 管理群中的节点

节点的管理作为群体管理生命周期的一部分，需要掌握：列出群中的节点、检查单个节点、更新节点、离开集群。

## 列出节点
`docker node ls`从管理节点查看群集中的节点列表：

```sh
$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
4ukl6wl714uljl43m51zji80f *   manager1            Ready               Active              Leader              18.03.1-ce
rp87bhtw4qmtg9i3k4o53b4xi     worker1             Ready               Active                                  18.03.1-ce
occde76c7oiv93zhds0x8ij5u     worker2             Ready               Active                                  18.03.1-ce
```

`AVAILABILITY`列显示调度程序是否可以将任务分配给节点：

- `Active` 意味着调度程序**可以**将任务分配给节点。
- `Pause` 意味着调度程序**不会**将新任务分配给节点，但**现有任务仍在运行**。
- `Drain`意味着调度程序**不会**将新任务分配给节点。调度程序**关闭所有现有任务**并在可用节点上调度它们。

`MANAGER STATUS`列显示节点参与Raft共识：

- `没有值`表示不参与集群管理的**工作节点**。
- `Leader` 意味着节点是为群体进行所有群集管理和编排决策的主要**管理节点**。
- `Reachable`意味着该节点是**参与Raft共识**法定人数的**管理节点**。如果**领导者节点变得不可用，该节点有资格被选为新领导者**。
- `Unavailable`意味着节点是一个**不能与其他管理沟通**的管理。如果管理节点变得不可用，则应该将新管理节点加入到群集中，或者将**工作节点提升为管理节点**。

有关群体管理的更多信息，请参阅[群体管理指南](https://docs.docker.com/engine/swarm/admin_guide/)。

## 检查单个节点

可以在管理节点上运行`docker node inspect <NODE-ID>`以查看单个节点的详细信息。输出默认为JSON格式，但可以传递该`--pretty`标记以便以可读格式打印结果。例如：

```sh
$ docker service inspect --pretty helloworld

ID:             qkexf88msvohybd6pico0t4g0
Name:           helloworld
Service Mode:   Replicated
 Replicas:      1
Placement:
UpdateConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
RollbackConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Rollback order:    stop-first
ContainerSpec:
 Image:         alpine:latest@sha256:7df6db5aa61ae9480f52f0b3a06a140ab98d427f86d8d5de0bedab9b8df6b1c0
 Args:          ping baidu.com
Resources:
Endpoint Mode:  vip
```

## 更新节点

可以修改节点属性，如下所示：

- [更改节点可用性](https://docs.docker.com/engine/swarm/manage-nodes/#change-node-availability)
- [添加或删除标签元数据](https://docs.docker.com/engine/swarm/manage-nodes/#add-or-remove-label-metadata)
- [更改节点角色](https://docs.docker.com/engine/swarm/manage-nodes/#promote-or-demote-a-node)

### 更改节点可用性
---
更改节点可用性：

- `drain`  耗尽**管理器节点**，以便**仅**执行**群集管理**任务并且**不**可用于**任务分配**。
- `drain`  排空一个**节点**，这样你就可以把它**拿下来进行维护**。
- `pause`  暂停一个节点，以便它**不**能接**收新的任务**。
- `active` 恢复**不可用或暂停**的节点可用状态。

例如，要将管理节点更改为`Drain`可用性：

```sh
$ docker node update --availability drain manager-1
node-1
```

### 添加或删除标签元数据
---
节点标签提供了一种灵活的**组织节点方法**。也可以在服务**约束**中使用节点标签。创建服务时应用约束，以**限制**调度程序为服务分配任务的节点。

`docker node update --label-add`在管理节点上运行以将标签元数据添加到节点。`--label-add`标志支持`<key>`或`<key>=<value>` 。为要添加的每个节点标签传递一次`--label-add`标志：

```sh
$ docker node update --label-add foo --label-add bar=baz node-1
node-1
```

使用docker节点更新设置的标签仅适用于**集群内的节点**。不要将它们与[dockerd的docker](https://docs.docker.com/engine/userguide/labels-custom-metadata/#daemon-labels)守护进程标签 [混淆](https://docs.docker.com/engine/userguide/labels-custom-metadata/#daemon-labels)。因此，可以使用节点标签将**关键任务限制为满足特定要求的节点**。例如，仅在需要运行特殊工作负载的机器上进行调度，例如符合[PCI-SS合规性的机器](https://www.pcisecuritystandards.org/)。受影响的工作节点无法损拖累这类特殊工作负载，因为它被约束，且无法更改节点标签。

然而，**引擎标签**仍然很有用，因为一些不影响容器安全编排的功能可能更好地分散方式设置。例如，引擎可以有一个标签来表明它具有某种类型的磁盘设备，这可能与安全性无关。这些标签更容易被swarm协调器“信任”。

### 升级或降级节点
---
可以将工作节点提升为经理角色。**当管理节点变得不可用或者想让管理节点进行维护**时，这非常有用。同样，可以将管理节点降级为辅助角色。

> **注意**：无论提升或降级节点的理由如何，都必须始终维护群体中的管理节点的法定人数。

要提升一个节点或一组节点，请在管理器节点运行`docker node promote`：

```sh
$ docker node promote node-3 node-2
Node node-3 promoted to a manager in the swarm.
Node node-2 promoted to a manager in the swarm.
```

要降级一个节点或一组节点，请从管理器节点运行`docker node demote`：

```sh
$ docker node demote node-3 node-2

Manager node-3 demoted in the swarm.
Manager node-2 demoted in the swarm.
```

`docker node promote`和`docker node demote`是为了方便的命令 `docker node update --role manager`和`docker node update --role worker` 。

## 在群集节点上安装插件

> **仅限Edge**：此选项仅在Docker CE Edge版本中可用。请参阅[Docker CE Edge](https://docs.docker.com/edge/)。

如果swarm服务依赖于一个或多个 [插件](https://docs.docker.com/engine/extend/plugin_api/)，则这些插件需要在可能部署服务的每个节点上可用。可以在每个节点上**手动安装插件或编写安装脚本**。在Docker 17.07及更高版本中，也可以使用Docker API以类似于**全局服务的方式部署插件**，只需指定一个`PluginSpec`而不是 `ContainerSpec`。

> **注意**：目前没有办法使用Docker CLI或Docker Compose将插件部署到swarm。另外，从私有存储库安装插件是不可能的。

[`PluginSpec`](https://docs.docker.com/engine/extend/plugin_api/#json-specification) 是由插件开发人员定义的。要将插件添加到所有Docker节点，请使用[`service/create`](https://docs.docker.com/engine/api/v1.31/#operation/ServiceCreate)API，并传递在`TaskTemplate`中定义的`PluginSpec` JSON 。

## 离开集群

在节点上运行命令`docker swarm leave`将其从群集中删除。

```sh
$ docker swarm leave
Node left the swarm.
```

当一个节点离开群集时，Docker引擎**停止以群集模式**运行。Orchestrator不再将任务安排到节点。

如果节点是**管理节点**，则会收到有关维护**法定人数**的警告。要覆盖警告，请传递`--force`标志。如果**最后一个管理器节点离开**群集，则群集变得**不可用**，要求您采取灾难恢复措施。

有关维护仲裁和灾难恢复的信息，请参阅 [Swarm管理指南](https://docs.docker.com/engine/swarm/admin_guide/)。

节点离开群集后，可以在管理器节点上运行命令`docker node rm`以从节点列表中删除该节点。例如：

```sh
$ docker node rm node-2
```

# 将服务部署到群集

Swarm服务使用**声明性**模型，这意味着您可以定义所需的**服务状态**，并依靠Docker来维护此状态。状态包括诸如（但不限于）的信息：

- 服务容器应该运行的映像名称和标记
- 有多少个容器参与服务
- 是否有任何端口暴露给群体外的客户
- 当Docker启动时服务是否应该自动启动
- 服务重新启动时发生的特定行为（例如是否使用滚动重启）
- 服务可以运行的节点的特性（例如资源约束和布局偏好）

## 创建一个服务

要创建**没有额外配置的单副本服务**，只需提供映像名称即可。命令启动一个带有**随机生成名称**并且**没有发布端口**的Nginx服务。

```sh
$ docker service create nginx
```

该服务安排在可用节点上。要确认服务已成功创建并启动，请使用以下`docker service ls`命令：

```sh
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                                                                                             PORTS
a3iixnklxuem        quizzical_lamarr    replicated          1/1                 docker.io/library/nginx@sha256:41ad9967ea448d7c2b203c699b429abe1ed5af331cd92533900c6d77490e0268
```

创建的服务并不总是立即运行。如果服务的图像不可用，如果没有节点满足您为服务配置的要求或其他原因，服务可能处于**挂起状态**。

要为服务提供名称，请使用以下`--name`标志：

```sh
$ docker service create --name my_web nginx
```

就像使用独立容器一样，可以通过在映像名称后面添加它来指定服务容器应该**运行的命令**。此示例启动一个名为`helloworld`使用`alpine`图像并运行以下命令的服务`ping docker.com`：

```sh
$ docker service create --name helloworld alpine ping docker.com
```

也可以指定要使用的服务的图像标签。这个例子修改了前一个使用`alpine:3.6`标签：

```sh
$ docker service create --name helloworld alpine:3.6 ping docker.com
```

### 使用私人注册表中的图像创建服务
---
如果图像在需要登录的私人注册表中可用，请在登录后使用 `--with-registry-auth`标志。如果图像存储在`registry.example.com`私有注册表中，请使用类似以下的命令：

```sh
$ docker login registry.example.com

$ docker service  create \
  --with-registry-auth \
  --name my_service \
  registry.example.com/acme/my_image:latest
```

这使用加密的WAL日志将**登录令牌**从本地客户端**传递到部署服务的群集节点**。有了这些信息，这些节点就能够登录到注册表并提取图像。

## 更新服务

可以使用`docker service update`命令更改现有服务 。当**更新服务时**，Docker会**停止**容器并使用新配置**重启**它们。

由于Nginx是一个Web服务，如果将端口80发布到群集外的客户端，它会更好。可以在创建服务时使用`-p`或`--publish`标志来指定。**更新**现有服务时，标志是`--publish-add`。还有一个`--publish-rm`标志可以**删除**以前发布的端口。

假设`my_web`来自上一节的服务仍然存在，请使用以下命令将更新为发布端口80。

```sh
$ docker service update --publish-add 80 my_web
```

要验证它是否有效，请使用`docker service ls`：

```sh
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                                                                                             PORTS
4nhxl7oxw5vz        my_web              replicated          1/1                 docker.io/library/nginx@sha256:41ad9967ea448d7c2b203c699b429abe1ed5af331cd92533900c6d77490e0268   *:0->80/tcp
```

可以更新有关现有服务的几乎每个配置详细信息，包括其运行的图像名称和标记。请参阅 [创建后更新服务的图像](https://docs.docker.com/engine/swarm/services/#update-a-services-image-after-creation)。

## 删除服务

要删除服务，请使用`docker service remove`命令。可以通过ID或名称来删除服务，如`docker service ls` 命令的输出中所示。以下命令将删除该`my_web`服务。

```sh
$ docker service rm my_web
$ docker service rm 4nhxl7oxw5vz
$ docker service rm 4nhxl7oxw5vz my_web2
```

## 服务配置细节

以下各节提供有关服务配置的详细信息。在几乎所有可以在服务创建时定义配置的情况下，也可以用类似的方式更新现有服务的配置。

### 配置运行时环境
---
可以在**容器中为运行时环境**配置以下选项：

- 使用`--env`标志的**环境变量**
- 使用`--workdir`标志的容器内的**工作目录**
- 使用`--user`标志的**用户名或UID**

以下服务的容器的环境变量`$MYVAR` 设置为`myvalue`，在`/tmp/`目录运行，并以`my_user`用户身份运行 。

```sh
$ docker service create --name helloworld \
  --env MYVAR=myvalue \
  --workdir /tmp \
  --user my_user \
  alpine ping docker.com
```

### 更新已有服务
---
更新现有服务运行的命令，可以使用`--args`标志。下面的示例更新一个已调用的现有服务`helloworld`，以便它运行命令`ping docker.com`而不是之前运行的任何命令：

```sh
$ docker service update --args "ping docker.com" helloworld
```

### 指定服务使用的映像版本
---
当**创建**服务时**未指定**要使用的**图像版本**的任何详细信息时，该服务将**使用`latest`标签**的版本。可以**强制**服务以几种不同的方式**使用特定**版本的图像，具体取决于您想要的结果。

图像版本可以用几种不同的方式：

- 如果指定了一个标签，那么管理器（或Docker客户端，如果使用 [内容信任](https://docs.docker.com/engine/swarm/services/#image_resolution_with_trust)）将该**标签解析为摘要**。当在工作节点上接收到创建容器任务的请求时，工作节点只会看到**摘要，而不是标签**。

  ```sh
  $ docker service create --name="myservice" ubuntu:16.04
  ```

  一些标签代表迭代的版本，例如`ubuntu:16.04`。像这样的标签几乎总是会随着时间的推移逐渐成为稳定的版本， **建议尽可能使用这种固定版本的标签**。

  其他类型的标签（如`latest`或`nightly`）可能经常会解析为新的版本，具体取决于图像作者更新标签的频率。**建议不要使用经常更新的标签**来运行服务，以防止**使用不同映像版本**的不同服务副本任务。

- 如果**完全不指定版本**，则按照惯例，图像的`latest`标记将被解析为摘要。创建服务任务时，工作人员使用此摘要中的图像。因此，以下两个命令是相同的：

  ```sh
  $ docker service create --name="myservice" ubuntu
  $ docker service create --name="myservice" ubuntu:latest
  ```

- 如果**直接指定版本和摘要**，则在创建服务任务时始终**使用**图像的**确切**版本。

  ```sh
  $ docker service create \
      --name="myservice" \
      ubuntu:16.04@sha256:35bc48a1ca97c3971611dc4662d08d131869daa692acb281c7e9e052924e38b1
  ```

当创建服务时，图像的标记将被解析为标记在创建服务时指向的**特定版本摘要**。服务的工作节点永远使用**特定版本**的摘要，除非服务被更新。如果使用经常更改的标签（例如`latest`），则此功能尤其重要，因为它可**确保**所有服务任务使用**相同版本的图像**。

> **注意**：如果启用了[内容信任](https://docs.docker.com/engine/security/trust/content_trust/)，客户端实际上会在联系swarm管理器之前将图像的标签解析为摘要，以验证图像是否已签名。因此，如果使用内容信任，swarm管理器会收到预先解决的请求。在这种情况下，如果客户端无法将图像解析为摘要，则请求失败。

如果**管理器无法将标签解析为摘要，则每个工作节点负责将标签解析为摘要**，并且**不同节点可以使用不同版本的图像**。如果发生这种情况，会输出下面的警告，用占位符代替真实信息。

```sh
unable to pin image <IMAGE-NAME> to digest: <REASON>
```

要**查看图像的当前摘要**，请执行命令 `docker inspect <IMAGE>:<TAG>`并查找`RepoDigests`行。以下是`ubuntu:latest`此内容写入时的当前摘要。为了清晰起见，输出被截断。

```sh
$ docker inspect ubuntu:latest
"RepoDigests": [
    "ubuntu@sha256:35bc48a1ca97c3971611dc4662d08d131869daa692acb281c7e9e052924e38b1"
],
```

创建服务之后，除非明确`docker service update`使用`--image`标志运行，否则其**映像不会更新** 。其他更新操作（如扩展服务，添加或删除网络或卷，重命名服务或任何其他类型的更新操作）不会更新服务的映像。

### 更新服务的图像
---
每个标签代表一个摘要，类似于Git哈希。一些标签，如 `latest`更新通常指向一个新的摘要。比如`ubuntu:16.04`，代表一个已发布的软件版本，并且预计不会更新以经常指向新的摘要。在Docker 1.13及更高版本中，当创建服务时，它将被限制为使用图像的**特定摘要创建任务**，直到**使用**`service update` `--image`标志更新服务为止。如果使用较**旧版本的Docker Engine，则必须删除并重新创建**服务以更新其映像。

当`service update`使用`--image`标志运行时，swarm管理器会查询Docker Hub或您的私人Docker注册中心以获取**标签记当前指向的摘要**并**更新服务任务**使用新摘要。

> **注意**：如果使用[内容信任](https://docs.docker.com/engine/swarm/services/#image_resolution_with_trust)，Docker客户端解析图像，swarm管理器接收图像和摘要，而不是标签。

通常，**管理节点可以将标签解析为新的摘要和服务**更新，**重新部署每个任务以使用新图像**。如果管理器无法解决标签或发生其他问题，将会发生以下情况：

#### 如果管理器成功解析标签

如果swarm manager可以将image标签解析为摘要，它会**指示工作节点重新部署任务并使用新摘要中的图像**。

- 如果工作人员在摘要中**缓存**了图像，则会使用缓存的图像。
- 如果不是，它会尝试从Docker Hub或私有注册表中**提取**图像。
  - 如果**成功**，则使用新映像**部署相关任务**。
  - 如果工作节点**无法拉取**图像，则服务**无法**在工作人员节点上**部署**。Docker**再次尝试**部署任务，可能**在不同**的工作节点上。

#### 如果管理器无法解析标签

如果swarm manager无法将图像解析为摘要，则全部不会丢失：

- 管理员指示工作节点**使用该标签**处的图像**重新**部署任务。
- 如果工作人员**拥有**解析为标记的**本地缓存图像**，则它会**使用该图像**。
- 如果工作人员**没有解析**为标签的本地高速缓存映像，则工作人员会尝试连接到Docker Hub或私有注册表以在该标签处**拉取图像**。
  - 如果这**成功**了，工作人员将**使用该图像**。
  - 如果**失败**，该任务将**无法部署**，并且管理员**再次尝试**部署该任务，可能位于**不同**的工作节点上。

## 发布端口

在创建群集服务时，可以通过两种方式将服务的端口发布到群集外的主机：

- [可以依靠路由网格](https://docs.docker.com/engine/swarm/services/#publish-a%20services-ports-using-the-routing-mesh)。当发布服务端口时，无论在节点上运行的服务是否有任务，群集都可以**在每个节点上的目标端口上访问该服务**。这并不复杂，是许多类型服务都是这种方式。
- [可以直接在](https://docs.docker.com/engine/swarm/services/#publish-a-services-ports-directly-on-the-swarm-node) 运行服务[的群集节点上发布服务任务的端口](https://docs.docker.com/engine/swarm/services/#publish-a-services-ports-directly-on-the-swarm-node)。此功能在Docker 1.13和更高版本中可用。这**绕过了路由网格**，并提供了最大的灵活性，包括**开发自己的路由框架**的能力。但是，有责任跟踪每个任务的运行位置，并将请求路由到任务，并在各个节点之间进行负载平衡。

### 使用路由网格发布服务的端口
---
要从外部向群发布服务的端口，请使用 `--publish <PUBLISHED-PORT>:<SERVICE-PORT>`标志。群集使服务可以在**每个群集节点**的发布端口**上**访问。如果外部主机连接到任何群集节点上的该端口，则路由网格会将其**路由到任务**。外部主机不需要知道**服务任务的IP地址**或内部使用**端口**就可以与服务交互。当用户或进程连接到服务时，任何运行服务任务的工作节点都可能会响应。有关群集服务网络的更多详细信息，请参阅 [管理群集服务网络](https://docs.docker.com/engine/swarm/networking/)。

#### 示例：在10个节点群上运行三任务Nginx服务

假设有一个10节点的群集，并且部署了一个在10节点群集上运行三个任务的Nginx服务：

```sh
$ docker service create \
	--name my_web \
	--replicas 3 \
	--publish published=8080,target=80 \
nginx
```

三个任务在最多三个节点上运行。不需要知道哪些节点正在运行任务; 在10个节点中的**任何**节点上连接到8080端口都可以将连接到三个`nginx`任务之一。你可以使用测试`curl`。以下示例假定这`localhost`是群集节点之一。如果情况并非如此，或者`localhost`无法解析到主机上的IP地址，请使用主机的IP地址或可解析的主机名称。

HTML输出被截断：

```sh
$ curl localhost:8080

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...truncated...
</html>
```

后续连接可能会**路由**到同一个群集节点或不同的节点。

### 直接在集群节点上发布服务的端口
---
如果需要根据**应用程序状态**做出路由决策，或者需要**完全控制将请求路由到服务任务**的流程，则使用路由网格可能不是应用程序的正确选择。要直接在**正在运行的节点上**发布服务端口，请使用标 志 `--publish` 选项`mode=host`。

> **注意：**如果使用`mode = host`直接在群集节点上发布服务的端口，并设置`published = <PORT>`，则会创建隐式限制，只能在**给定群集节点上为服务运行一个任务**。可以通过指定发布而不使用端口定义来解决此问题，这会导致Docker为每个任务**分配一个随机**端口。
>
> 另外，如果使用`mode = host`，并且在`docker service create`上不使用`--mode = global`标志，则**很难知道哪些节点正在运行**服务以将工作路由到它们。

#### 示例：`nginx`在每个群集节点上运行Web服务

[nginx](https://hub.docker.com/_/nginx/)是一个开源的反向代理，负载均衡器，HTTP缓存和一个Web服务器。如果使用路由网格将nginx作为服务运行，则连接到任何swarm节点上的nginx端口将显示（有效）运行该服务**的随机群集节点**的网页。

以下示例在群集中的每个节点上运行nginx作为服务，并在每个群集节点上本地公开nginx端口。

```sh
$ docker service create \
  --mode global \
  --publish mode=host,target=80,published=8080 \
  --name=nginx \
  nginx:latest
```

可以在每个群集节点的端口8080上访问nginx服务器。如果向群集添加节点，则会启动一个nginx任务。不能在任何绑定到端口8080的群集节点上启动另一个服务或容器。

> **注意**：这是一个极端的例子。为多层服务创建应用程序层路由框架非常复杂，超出了本主题的范围。

## 将服务连接到覆盖网络

可以使用覆盖网络连接群中的一个或多个服务。

首先，使用`docker network create` 带有`--driver overlay`标志的命令在**管理器节点**上创建覆盖网络。

```sh
$ docker network create --driver overlay my-network
```

在群集模式下创建覆盖网络后，**所有管理节点都可以访问网络**。可以创建新服务并传递该`--network`标志以将**服务附加到覆盖网络**：

```sh
$ docker service create \
  --replicas 3 \
  --network my-network \
  --name my-web \
  nginx
```

集群扩展`my-network`到运行服务的每个节点。

还可以使用`--network-add`标志**将现有服务连接到覆盖网络** 。

```sh
$ docker service update --network-add my-network my-web
```

要**断开**正在运行的服务与网络的连接，请使用`--network-rm`标志。

```sh
$ docker service update --network-rm my-network my-web
```

有关覆盖网络和服务发现的更多信息，请参阅 [将服务附加到覆盖网络](https://docs.docker.com/engine/swarm/networking/)和 [Docker群集模式覆盖网络安全模型](https://docs.docker.com/engine/userguide/networking/overlay-security-model/)。

## 授予对秘密的服务访问权限


要创建一个访问Docker管理的机密的服务，请使用`--secret` 标志。有关更多信息，请参阅 [管理Docker服务的敏感字符串（机密）](https://docs.docker.com/engine/swarm/secrets/)

## 自定义服务的隔离模式


Docker 17.12 CE和更高版本允许指定群集服务的隔离模式。**设置仅适用于Windows主机，Linux主机将忽略此设置。**隔离模式可以是以下之一：

- `default`：使用由`-exec-opt`标志或`exec-opts`阵列所配置的默认隔离模式为Docker主机配置`daemon.json`。如果守护进程未指定隔离技术，`process`则它是Windows Server `hyperv`的默认设置，并且是Windows 10的默认（且唯一）选择。

- `process`：将服务任务作为主机上的单独进程运行。

  > **注意**：`process`隔离模式仅在Windows Server上受支持。Windows 10仅支持`hyperv`隔离模式。

- `hyperv`：将服务任务作为独立的`hyperv`任务运行。这增加了开销但提供了更多的隔离。

可以在使用该`--isolation`标志创建或更新新服务时指定隔离模式。

## 控制服务分布


Swarm服务为提供了几种不同的方式来**控制不同节点上服务的规模和位置**。

- 可以指定服务是否需要运行**特定数量的副本**，还是应该在每个工作节点上**全局运行**。请参阅 [复制或全局服务](https://docs.docker.com/engine/swarm/services/#replicated-or-global-services)。

- 可以配置服务的 [CPU或内存要求](https://docs.docker.com/engine/swarm/services/#reserve-memory-or-cpus-for-a-service)，并且服务**仅在满足**这些要求的节点上运行。

- [分布约束](https://docs.docker.com/engine/swarm/services/#placement-constraints)使可以将服务配置为**仅在具有特定（任意）元数据**集的节点上运行，并且如果适当的节点**不存在**，则会*导致部署失败**。例如，可以指定服务只应在任意标签`pci_compliant`设置为的节点上运行 `true`。

- [“分布位置”偏好设置](https://docs.docker.com/engine/swarm/services/#placement-preferences)允许每个节点应用一系列值的**任意标签**，并使用**算法**将服务的任务分散到这些节点上。目前，唯一支持的算法是`spread`尝试将它们**均匀**地分布。例如，如果`rack`使用值为1-10的标签为每个节点添加标签，则指定一个位置偏好设置`rack`，然后`rack`在采用其他**放置约束**后，服务任务尽可能均匀地分布在**具有标签**的所有节点上，放置偏好以及其他节点**特定**的限制。

  与约束不同，分布选项是**尽力而为**的，并且如果**没有节点能够满足偏好**，则服务**不会失败**部署。如果指定服务的分布选项，那么当群集管理决定哪些节点应该运行服务任务时，与该选项匹配的节点**排名较高**。其他因素（如服务的高可用性）还会考虑将哪些节点安排为运行服务任务。例如，如果有`N`个节点具有标签（以及其他一些节点），并且服务配置为运行`N + 1`个副本，则会在尚未拥有该服务的节点上调度`+1`有一个，不管该节点是否有`rack`标签。

### 副本服务或全局服务
---
Swarm模式有两种类型的服务：副本和全局。对于副本服务，指定swarm管理器调度到可用节点的**副本任务数**。对于全局服务，调度程序在**每个可用节点上放置一个任务**，以满足服务的 [分布约束](https://docs.docker.com/engine/swarm/services/#placement-constraints)和 [资源需求](https://docs.docker.com/engine/swarm/services/#reserve-cpu-or-memory-for-a-service)。

可以使用`--mode`标志来控制服务的类型。如果未指定模式，则服务默认为`replicated`。对于**副本服务**，可以指定要使用`--replicas`标志开始的**副本任务数**。例如，要使用3个副本任务启动复制的nginx服务：

```sh
$ docker service create \
  --name my_web \
  --replicas 3 \
  nginx
```

要在每个可用节点上启动**全局服务**，请传递`--mode global`给 `docker service create`。每当**新节点变为可用**时，调度程序就将全局服务的任务**放置在新节点**上。例如，要开始在群中**每个节点**上运行`alpine`的服务：

```sh
$ docker service create \
  --name myservice \
  --mode global \
  alpine top
```

通过服务约束，可以在调度程序将服务部署到节点之前设置节点的条件。可以根据**节点属性**和**元数据**或**引擎元数据**将约束应用于服务。

### 为服务预留内存或CPU
---
要为服务保留给定数量的**内存或CPU**数量，请使用 `--reserve-memory`或`--reserve-cpu`标志。如果**没有**可用的节点能够满足要求（例如，如果请求4个CPU并且群中没有节点具有4个CPU），则服务保持**挂起**状态，**直到有合适**的节点可用于运行其任务。

### 内存异常（OOME）
---
如果服务尝试使用比swarm节点**更多的内存**，则可能会遇到**内存异常**（OOME），并且容器或Docker守护进程可能会被**内核OOM杀手所杀**。要防止发生这种情况，请确保应用程序在具有**足够内存**的主机上运行，并且请参阅 [了解耗尽内存的风险](https://docs.docker.com/engine/admin/resource_constraints/#understand-the-risks-of-running-out-of-memory)。

Swarm服务允许使用资源约束，分布选项和标签来确保将服务部署到适当的群集节点。

### 分布约束
---
**使用布局约束来控制服务可以分配给的节点**。在以下示例中，服务仅在[标签](https://docs.docker.com/engine/swarm/engine/swarm/manage-nodes/#add-or-remove-label-metadata) `region`设置为的节点上运行 `east`。如果**没有适当标签**的节点可用，则**部署失败**。`--constraint`标志使用相等运算符（`==`或`!=`）。对于**副本服务，有可能所有服务在同一节点上运行，或者每个节点只运行一个副本，或者某些节点不运行任何副本**。对于**全局服务，服务在满足布局约束和任何[资源需求的](https://docs.docker.com/engine/swarm/services/#reserve-cpu-or-memory-for-a-service)每个节点上**运行 。

```sh
$ docker service create \
  --name my-nginx \
  --replicas 5 \
  --constraint region==east \
  nginx
```

也可以`constraint`在`docker-compose.yml` 文件中使用服务级别。

如果指定了多个分布位置约束，则服务**只会部署到满足**它们的节点上。以下示例限制服务在所有`region`设置为`east`并且`type`未设置为的节点上运行`devel`：

```sh
$ docker service create \
  --name my-nginx \
  --global \
  --constraint region==east \
  --constraint type!=devel \
  nginx
```

还可以将布局约束与布局偏好和CPU/内存约束结合使用。小心不要使用不可能实现的设置。

### 分布偏好设置
---
尽管[分布约束](https://docs.docker.com/engine/swarm/services/#placement-constraints)限制了服务可以运行的节点，但**分布偏好**尝试以算法的方式将服务放置在**适当偏好的节点**上（目前只能**均匀分布**）。例如，如果每个节点分配一个`rack`标签，则可以设置一个放置选项，以便按照`rack`值通过标签均匀分布服务。这样，如果丢失了`rack`，该服务仍在其他`rack`上的节点上运行。

分布偏好设置没有严格执行。如果没有节点具有选项中指定的标签，则会像部署首选项那样部署该服务。

> 注意：**全局服务会忽略分布位置偏好设置**。

以下示例设置了一个选项，以根据`datacenter`标签的值在节点之间分布部署。如果有一些节点`datacenter=us-east`和其他节点有 `datacenter=us-west`，则服务在两组节点间**尽可能均匀**地部署。

```sh
$ docker service create \
  --replicas 9 \
  --name redis_2 \
  --placement-pref 'spread=node.labels.datacenter' \
  redis:3.0.6
```

> **缺少标签或空标签**<br/>缺少用于传播的标签的节点仍然接收任务分配。作为一个组，这些节点接收任何与由特定标签值标识的其他组等比例的任务。从某种意义上说，一个缺失的标签与附加了一个空值的标签相同。如果服务**只**应在标签用于扩展偏好的节点上运行，则偏好应与约束组合。

可以指定**多个分布偏好设置**，并按照遇到的顺序处理它们。以下示例使用多个分布位置选项设置服务。任务首先在各个`datacenter `上传播，然后在`rack`上传播（如各个标签所示）：

```sh
$ docker service create \
  --replicas 9 \
  --name redis_2 \
  --placement-pref 'spread=node.labels.datacenter' \
  --placement-pref 'spread=node.labels.rack' \
  redis:3.0.6
```

还可以将**分布选项与分布约束或CPU/内存约束**结合使用。小心不要使用不可能实现的设置。

此图说明了分布展示位置偏好的工作原理

![放置偏好示例](https://docs.docker.com/engine/swarm/images/placement_prefs.png)

在`docker service update`更新服务时，`--placement-pref-add` 在所有现有展示位置偏好设置之后追加新的展示位置偏好设置。 `--placement-pref-rm`删除与参数匹配的现有展示位置偏好设置。

## 配置服务的更新行为

在创建服务时，可以指定**滚动更新**行为，以便在运行时群集应如何将更改应用于`docker service update`服务。也可以将这些标志指定为更新的一部分，作为`docker service update`参数 。

`--update-delay`标志配置**更新服务任务或多组任务之间的时间延迟**。可以将时间描述为`T`，秒数`Ts`，分钟数`Tm`或小时数的组合`Th`。因此`10m30s`表示延迟10分30秒。

默认情况下，调度程序**一次更新1个**任务。可以通过 `--update-parallelism`标志来配置调度程序**同时**更新的**最大**服务任务数。

当对单个任务的更新返回状态时`RUNNING`，调度程序通过**继续**执行另一个任务来继续更新，直到更新**所有**任务。如果在任务更新期间的任何时间`FAILED`，调度程序会暂停更新。可以使用`docker service create`或`docker service update`的 `--update-failure-action` 标志来控制行为。

在下面的示例服务中，调度程序一次**最多应用2个副本**。当更新的任务返回`RUNNING`或者时`FAILED`，调度程序在**停止*下一个任务更新之前等待10秒钟：

```sh
$ docker service create \
  --replicas 10 \
  --name my_web \
  --update-delay 10s \
  --update-parallelism 2 \
  --update-failure-action continue \
  alpine
```

`--update-max-failure-ratio`标志控制在更新之前更新的整个过程中可能失败的部分任务可能失败。例如，使用`--update-max-failure-ratio 0.1 --update-failure-action`暂停，**10％的任务更新失败后，更新将暂停**。

如果任务未启动，或者在`--update-monitor`标志指定的**监视时间段内停止**运行，则认为单个任务**更新失败**。`--update-monitor`的默认值为`30秒`，这意味着任务在其开始后的**前30秒内**失败，将计入服务**更新失败阈值**，并且**在此之后的失败将不计入**。

## 回滚服务

如果`docker service update`更新版本的服务没有按预期运行，可以使用`--rollback`标志手动回滚到服务的先前版本。这会将服务恢复到最近`docker service update`命令之前的配置 。

其他选项可以结合使用`--rollback`。 例如， `--update-delay 0s`在任务之间没有延迟地执行回滚：

```sh
$ docker service update \
  --rollback \
  --update-delay 0s
  my_web
```

在Docker 17.04及更高版本中，如果服务更新未能部署，可以将服务配置为**自动回滚**。与新的自动回滚功能相关，在Docker 17.04及更高版本中，如果守护程序运行Docker 17.04或更高版本，**手动回滚**将在服务器端而不是客户端进行处理。允许**手动启动的回滚**来遵守新的回滚参数。客户端是版本感知的，所以它仍然使用旧的守护进程的方法。

最后，在Docker 17.04及更高版本中，`--rollback`不能与其他标志一起使用`docker service update`。

## 更新失败，自动回滚

可以通过以下方式来配置服务：如果对服务的**更新**导致**重新部署**失败，则服务可以**自动回滚**到以前的配置。这有助于保护服务**可用性**。可以在创建或更新服务时设置以下一个或多个标志。如果您未设置值，则使用默认值。

| 选项                           | 默认    | 描述                                                         |
| ------------------------------ | ------- | ------------------------------------------------------------ |
| `--rollback-delay`             | `0s`    | 在**回滚下一个任务**之前回滚任务之后要**等待的时间**。`0`在第一个回滚任务部署完成后**立即回滚**第二个任务的方法值。 |
| `--rollback-failure-action`    | `pause` | 当任务无法回滚时，无论是要`pause`还是`continue`试图**回滚其他**任务。 |
| `--rollback-max-failure-ratio` | `0`     | 在回滚期间**容忍的故障**率，指定为介于0和1之间的浮点数。例如，给定5个任务，故障率`.2`会容忍一个任务无法回滚。值`0`没有故障被容忍，而值`1`装置的任何数量的故障容忍。 |
| `--rollback-monitor`           | `5s`    | 每个任务回滚之后的**持续时间监视失败**。如果任务在**此时间段过去之前停止**，则认为**回滚失败**。 |
| `--rollback-parallelism`       | `1`     | **并行回滚的最大任务数**。默认情况下，一次回滚一个任务。一个值将`0`导致所有任务并行回滚。 |

以下示例将配置`redis`服务`docker service update`在部署失败时自动回滚。**两个任务**可以**并行**回滚。任务在回滚后**监视20秒**，以确保它们不会退出，并且**最大失败率为20％**是可以接受的。默认值用于`--rollback-delay`和`--rollback-failure-action`。

```sh
$ docker service create --name=my_redis \
                        --replicas=5 \
                        --rollback-parallelism=2 \
                        --rollback-monitor=20s \
                        --rollback-max-failure-ratio=.2 \
                        redis:latest
```

## 服务访问`卷或挂载`

为了获得**最佳性能和可移植性**，应避免将**重要数据直接写入容器的可写层**，而应使用**数据卷或绑定挂载**。这一原则也适用于服务。

可以为群体中的服务创建两种类型的挂载，`volume`挂载或 `bind`挂载。无论使用哪种类型的挂载，在创建服务时使用标志`--mount`进行配置 ，或者 在更新现有服务时使用`--mount-add`或 `--mount-rm`标志进行配置。如果未指定类型，则默认值为`volume`。

### 数据卷
---
数据卷`volume`是**独立于容器**而存在的存储。群集服务下的数据卷的**生命周期与容器下的相似**。卷超过任务和服务，因此必须**单独管理**它们的删除。卷可以在**部署服务之前**创建，或者如果在某个特定主机上不存在任何特定主机时，它们将根据服务上的卷规范**自动创建**。

要将现有数据卷用于服务，请使用以下`--mount`标志：

```sh
$ docker service create \
  --mount src=<VOLUME-NAME>,dst=<CONTAINER-PATH> \
  --name myservice \
  <IMAGE>
```

如果`<VOLUME-NAME>`任务安排到特定主机时，具有相同的卷不存在，则会创建一个。默认的`volume`驱动程序是`local`。要按需创建模式下使用不同的卷驱动程序，请使用以下`--mount`标志**指定驱动程序**及其选项：

```sh
$ docker service create \
  --mount type=volume,src=<VOLUME-NAME>,dst=<CONTAINER-PATH>,volume-driver=<DRIVER>,volume-opt=<KEY0>=<VALUE0>,volume-opt=<KEY1>=<VALUE1>
  --name myservice \
  <IMAGE>
```

### 绑定挂载
---
绑定挂载是调度程序为任务部署容器的主机的文件系统路径。Docker将路径安装到容器中。文件系统路径必须在swarm为任务初始化容器之前存在。

以下示例显示了绑定挂载语法：

- 要挂载读写绑定：

  ```sh
  $ docker service create \
    --mount type=bind,src=<HOST-PATH>,dst=<CONTAINER-PATH> \
    --name myservice \
    <IMAGE>
  ```

- 要挂载一个只读绑定：

  ```sh
  $ docker service create \
    --mount type=bind,src=<HOST-PATH>,dst=<CONTAINER-PATH>,readonly \
    --name myservice \
    <IMAGE>
  ```

> **重要提示**：绑定挂载可能很有用，但它们也可能导致问题。在大多数情况下，建议构建应用程序，以便不需要从主机安装路径。主要风险包括以下几点：
>
> - 如果将主机路径绑定到服务的容器中，则**路径必须存在于每个群集节点上**。Docker群模式调度程序可以在满足资源可用性要求的任何机器上调度容器，并满足指定的所有约束和分布偏好。
> - 如果Docker群模式调度程序变得不健康或无法访问，Docker群模式调度程序可能会随时**重新安排**正在运行的服务容器。
> - 主机绑定挂载是**不可移植**的。当使用绑定挂载时，不保证应用程序在开发中的运行方式与生产中的相同。

## 使用模板创建服务

可以使用`service create`Go的[文本/模板](http://golang.org/pkg/text/template/) 软件包提供的语法来为某些标志使用[模板](http://golang.org/pkg/text/template/)。

支持以下标志：

- `--hostname`
- `--mount`
- `--env`

Go模板的有效占位符是：

| 占位符            | 描述     |
| ----------------- | -------- |
| `.Service.ID`     | 服务ID   |
| `.Service.Name`   | 服务名称 |
| `.Service.Labels` | 服务标签 |
| `.Node.ID`        | 节点ID   |
| `.Task.Name`      | 任务名称 |
| `.Task.Slot`      | 任务槽   |

### 模板示例
---
本示例根据服务的名称和容器运行节点的ID来设置创建的容器的模板：

```sh
$ docker service create --name hosttempl \
                        --hostname="{{.Node.ID}}-{{.Service.Name}}"\
                         busybox top
```

要查看使用模板的结果，请使用`docker service ps`和 `docker inspect`命令。

```sh
$ docker service ps va8ew30grofhjoychbr6iot8c
ID            NAME         IMAGE                                                                                   NODE          DESIRED STATE  CURRENT STATE               ERROR  PORTS
wo41w8hg8qan  hosttempl.1  busybox:latest@sha256:29f5d56d12684887bdfa50dcd29fc31eea4aaf4ad3bec43daf19026a7ce69912  2e7a8a9c4da2  Running        Running about a minute ago
$ docker inspect --format="{{.Config.Hostname}}" hosttempl.1.wo41w8hg8qanxwjwsg4kxpprj
```

# docker config 基本命令

```sh
$ docker config -h
Usage:  docker config COMMAND

管理 docker 配置

Commands:
  create      # 从文件或STDIN创建配置
  inspect     # 显示一个或多个配置的详细信息
  ls          # 配置列表
  rm          # 删除一个或多个配置
```
## create 创建

从文件或STDIN创建配置

### 命令参数选项

---

| 选项，简写          | 默认 | 描述     |
| ------------------- | ---- | -------- |
| `--label , -l`      |      | 配置标签 |
| `--template-driver` |      | 模板驱动 |

### 示例

---

```sh
# docker config create [OPTIONS] CONFIG file|-

# 创建一个my-config 的配置文件
$ echo "This is a config" | docker config create my-config -
```

## inspect 查看

显示一个或多个配置的详细信息

 ### 命令参数选项

---

| 选项，简写      | 默认 | 描述                       |
| --------------- | ---- | -------------------------- |
| `--format , -f` |      | 使用给定的Go模板格式化输出 |
| `--pretty`      |      | 以人性化的格式打印信息     |

### 示例

---

```sh
$ docker config inspect my-config
[
    {
        "ID": "te2r2q67yipn0o5tesavytn3n",
        "Version": {
            "Index": 2971
        },
        "CreatedAt": "2018-05-10T09:26:46.485305805Z",
        "UpdatedAt": "2018-05-10T09:26:46.485305805Z",
        "Spec": {
            "Name": "my-config",
            "Labels": {},
            "Data": "VGhpcyBpcyBhIGNvbmZpZwo="
        }
    }
]
```

## ls 列表

查看配置文件列表

```sh
$ docker config ls
ID                          NAME                CREATED             UPDATED
te2r2q67yipn0o5tesavytn3n   my-config           30 seconds ago      30 seconds ago
```

## rm 删除

删除一个或多个配置

 ```sh
$ docker config rm my-config
 ```

# 使用 config 存储数据

Docker 17.06引入了swarm服务配置，它允许在服务映像外部或运行容器中存储**非敏感信息**，例如**配置文件**。这允许保持图像尽可能**通用**，而无需将**配置文件绑定到容器或使用环境变量**。

Config以类似于[秘密的](https://docs.docker.com/engine/swarm/secrets/)方式运行，不同之处在于它们没有在`rest`时加密，并且不使用RAM和磁盘直接安装到**容器**的文件系统中。**随时可以从服务添加或删除**配置，并且服务可以**共享配置**。甚至可以将配置与环境变量或标签结合使用，以获得最大的灵活性。配置值可以是通用字符串或二进制内容（最大可达`500 kb`）。

> **注意**：Docker**配置仅适用于群集**服务，而**不适用于独立容器**。要使用此功能，请考虑将容器作为1级服务运行。

**Windows支持**

Docker 17.06和更高版本包括对Windows容器配置的支持。在实现中存在差异的地方，它们在下面的例子中被调用。牢记以下显着差异：

- 自定义目标的配置文件不直接绑定到Windows容器中，因为Windows不支持非目录文件绑定挂载。相反，容器的配置都被装入 `C:\ProgramData\Docker\internal\configs`容器内的（应用程序不应该依赖的实现细节）。符号链接用于指向容器内配置的所需目标。默认目标是`C:\ProgramData\Docker\configs`。
- 创建使用Windows容器的服务时，配置不支持指定UID，GID和模式的选项。配置目前只能由具有`system`容器访问权限的管理员和用户访问。

## 如何管理 config

当配置添加到swarm中时，Docker会通过**相互TLS连接**将**配置发送到swarm管理器**。配置存储在**加密的Raft日志**中。整个Raft日志被**复制**到其他管理器中，确保配置的**高可用性**保证与其他群管理数据相同。

当授予新创建或正在运行的服务对配置的访问权时，**配置将作为文件装载**到容器中。容器中安装点在Linux容器中的位置默认为`/<config-name>`。在Windows容器中，配置都被装入`C:\ProgramData\Docker\configs`，**软链接被创建到所需的位置**，默认为 `C:\<config-name>`。

可以使用**数字ID或用户或组**的名称来设置**所有权（`uid`和`gid`）或配置**。还可以指定**文件权限（`mode`）**。对于Windows容器，这些设置将被忽略。

- 如果未设置，则配置由**用户拥有**，并且运行容器命令（通常`root`）以及该用户的默认组（通常也是`root`）。
- 如果未设置，则配置具有**全局可读的权限**（模式`0444`），除非在容器中设置了`umask` ，在这种情况下，该模式受该`umask`值影响。

可以**随时更新*服务以授予其访问其他配置或**撤销**对给定配置的**访问权限**。

如果节点是群管理器，或者它**正在运行已被授权**访问配置的服务任务，则节点只能访问配置。当容器**任务停止**运行时，共享给它的配置将从该**容器的内存中文件系统卸载**，并从**节点的内存中清除**。

如果节点在运行可访问配置的任务容器时**失去与群集的连接**，则**任务容器仍可访问其配置**，但在节点重新连接到群集之前**无法接收更新**。<br/>可以**随时添加或检查**单个配置，或列出所有配置。**无法删除正在运行的服务正在使用的配置**。请参阅[旋转配置，](https://docs.docker.com/engine/swarm/configs/#example-rotate-a-config)以便在**不中断正在运行**的服务的情况下**移除**配置。

要更容易地**更新或回滚**配置，请考虑在配置名称中**添加版本号或日期**。通过控制给定容器内配置的**挂载点**，这变得更容易。

要更新堆栈，请更改compose文件，然后重新运行`docker stack deploy -c <new-compose-file> <stack-name>`。如果在该文件中使用新的配置，服务将开始使用它们。请记住，**配置是不可变的**，所以无法更改现有服务的文件。相反，**创建一个新配置**以使用不同的文件。<br/>可以运行`docker stack rm`以**停止应用程序并取下堆栈**。这将删除`docker stack deploy`使用相同堆栈名称创建的任何**配置**。这将删除**所有**配置，包括那些未被服务引用的配置以及在`docker service update --config-rm`之后剩下的配置。

## 实例演示

### 开始使用 config

---

这个简单的例子显示了配置如何在几个命令中工作。

1. 添加一个配置到Docker。`docker config create`命令读取标准输入，因为最后一个参数表示要读取配置文件的文件被设置为`-`。

   ```sh
   $ echo "This is a config" | docker config create my-config -
   ```

2. 创建一个`redis`服务并授予它对配置的访问权限。默认情况下，容器可以访问配置`/my-config`，但可以使用该`target`选项自定义容器上的文件名。

   ```sh
   $ docker service create --name redis --config my-config redis:alpine
   ```

3. 验证任务是否正在运行，没有问题`docker service ps`。如果一切正常，输出结果如下所示：

   ```sh
   $ docker service ps redis
   ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
   xdhqto5td9jk        redis.1             redis:latest        manager1            Running             Running 55 seconds ago 
   ```

4. `docker ps`获取`redis`服务任务容器的标识，以便可以使用`docker container exec`它连接到容器并读取config数据文件的内容，该数据文件默认为全部可读，并且具有与config的名称相同的名称。下面的第一条命令说明了如何找到容器ID，第二个和第三个命令使用shell完成来自动执行此操作。

   ```sh
   $ docker ps --filter name=redis -q
   5cb1c2348a59
   
   # ssh 连接到manager1
   $ docker-machine.exe ssh manager1
   $ docker container exec $(docker ps --filter name=redis -q) ls -l /my-config
   -r--r--r--    1 root     root            12 Jun  5 20:49 my-config                                                     
   $ docker container exec $(docker ps --filter name=redis -q) cat /my-config
   This is a config
   ```

5. 尝试删除配置。删除失败，因为`redis`服务**正在运行**并可以访问配置。

   ```sh
   $ docker config ls
   ID                          NAME                CREATED             UPDATED
   fzwcfuqjkvo5foqu7ts7ls578   hello               31 minutes ago      31 minutes ago
   
   $ docker config rm my-config
   Error response from daemon: rpc error: code = 3 desc = config 'my-config' is
   in use by the following service: redis
   ```

6. `redis`通过更新服务，从正在运行的服务中删除对配置的访问。

   ```sh
   $ docker service update --config-rm my-config redis
   ```

7. 重复步骤3和4，验证该服务不再有权访问配置。容器ID不同，因为该 `service update`命令重新部署服务。

   ```sh
   $ docker container exec -it $(docker ps --filter name=redis -q) cat /my-config
   cat: can't open '/my-config': No such file or directory
   ```

8. 停止并删除服务，并从Docker中删除配置。

   ```sh
   $ docker service rm redis
   $ docker config rm my-config
   ```

### 在 Windows 中使用 config

---

这是一个非常简单的示例，它显示了如何使用Microsoft Windows Server 2016上的Docker 17.06 EE或Windows上的Docker 17.06 CE上运行的Microsoft IIS服务的配置。它将网页存储在配置中。

此示例假定您已安装PowerShell。

1. 将以下内容保存到一个新文件中`index.html`。

   ```html
   <html>
     <head><title>Hello Docker</title></head>
     <body>
       <p>Hello Docker! You have deployed a HTML page.</p>
     </body>
   </html>
   ```

2. 如果还没有加入集群，请初始化或加入群集。

   ```sh
   $ docker swarm init
   ```

3. 将`index.html`文件保存为名为`homepage`的群集配置。

   ```sh
   $ docker config create homepage index.html
   ```

4. 创建一个IIS服务并授予它对`homepage`配置的访问权限。

   ```sh
   $ docker service create \
       --name my-iis \
       --publish published=8000,target=8000 \
       --config src=homepage,target="\inetpub\wwwroot\index.html" \
       microsoft/iis:nanoserver
   ```

5. 访问IIS服务`http://localhost:8000/`。它应该从第一步开始提供HTML内容。

   ```sh
   $ docker service ps my-iis
   $ docker-machine ip manager1
   
   $ curl http://$(docker-machine ip manager1):8080
   ```

6. 删除服务和配置。

   ```sh
   $ docker service rm my-iis
   $ docker config rm homepage
   ```

### 使用配置与Nginx服务

---

这个例子分为两部分。 [第一部分](https://docs.docker.com/engine/swarm/configs/#generate-the-site-certificate)是关于生成站点证书，并不直接涉及Docker配置，但是它建立[了第二部分](https://docs.docker.com/engine/swarm/configs/#configure-the-nginx-container)的基础，在存储和使用站点证书作为一系列秘密，并将Nginx配置作为配置使用。该示例显示如何在配置上设置选项，例如容器中的目标位置和文件权限（`mode`）。

#### 生成站点证书

为您的站点生成根CA和TLS证书和密钥。对于生产站点，您可能希望使用服务`Let’s Encrypt`来生成TLS证书和密钥，但此示例使用命令行工具。这一步有点复杂，但仅仅是一个设置步骤，以便您可以将某些内容存储为Docker秘密。如果你想跳过这些子步骤，您可以[使用我们的加密](https://letsencrypt.org/getting-started/)生成网站密钥和证书，命名文件`site.key`和 `site.crt`，然后跳到 [配置Nginx的容器](https://docs.docker.com/engine/swarm/configs/#configure-the-nginx-container)。

1. 生成一个根密钥。

   ```sh
   $ openssl genrsa -out "root-ca.key" 4096
   ```

2. 使用根密钥生成CSR。

   ```sh
   $ openssl req \
             -new -key "root-ca.key" \
             -out "root-ca.csr" \
             -sha256
             #-subj '/C=CN/ST=SC/L=GZ/O=org/OU=unit/CN=docker/emailAddress=docker@docker.io'
   ```

3. 配置根CA。编辑一个名为的新文件`root-ca.cnf`并将以下内容粘贴到其中。这限制了根CA只签署叶证书而不是中间CA。

   ```properties
   [root_ca]
   basicConstraints = critical,CA:TRUE,pathlen:1
   keyUsage = critical, nonRepudiation, cRLSign, keyCertSign
   subjectKeyIdentifier=hash
   ```

4. 签署证书。

   ```sh
   $ openssl x509 -req -days 3650 -in "root-ca.csr" \
                  -signkey "root-ca.key" -sha256 -out "root-ca.crt" \
                  -extfile "root-ca.cnf" -extensions \
                  root_ca
   ```

5. 生成站点密钥。

   ```sh
   $ openssl genrsa -out "site.key" 4096
   ```

6. 生成站点证书并使用站点密钥对其进行签名。

   ```sh
   $ openssl req -new -key "site.key" -out "site.csr" -sha256
   ```

7. 配置站点证书。编辑一个名为的新文件`site.cnf`并将以下内容粘贴到其中。这限制了站点证书，因此它只能用于对服务器进行身份验证，并且不能用于签名证书。

   ```sh
   [server]
   authorityKeyIdentifier=keyid,issuer
   basicConstraints = critical,CA:FALSE
   extendedKeyUsage=serverAuth
   keyUsage = critical, digitalSignature, keyEncipherment
   subjectAltName = DNS:localhost, IP:192.168.99.106
   subjectKeyIdentifier=hash
   ```

8. 签署网站证书。

   ```sh
   $ openssl x509 -req -days 750 -in "site.csr" -sha256 \
       -CA "root-ca.crt" -CAkey "root-ca.key" -CAcreateserial \
       -out "site.crt" -extfile "site.cnf" -extensions server
   ```

9. 在`site.csr`和`site.cnf`文件不需要由Nginx的服务，但你需要他们，如果你想生成一个新的站点证书。保护`root-ca.key`文件。

#### 配置NGINX容器

1. 生成一个非常基本的Nginx配置，通过HTTPS提供静态文件。TLS证书和密钥作为Docker机密存储，以便它们可以轻松旋转。

   在当前目录中，`site.conf`使用以下内容创建一个新文件：

   ```nginx
   server {
       listen                443 ssl;
       server_name           192.168.99.106;
       ssl_certificate       /run/secrets/site.crt;
       ssl_certificate_key   /run/secrets/site.key;
   
       location / {
           root   /usr/share/nginx/html;
           index  index.html index.htm;
       }
   }
   ```

2. 创建两个秘密，代表密钥和证书。只要小于500 KB，您就可以将任何文件存储为秘密文件。这使您可以将密钥和证书与使用它们的服务分离。在这些示例中，秘密名称和文件名是相同的。

   ```sh
   $ docker secret create site.key site.key
   $ docker secret create site.crt site.crt
   ```

3. 将`site.conf`文件保存在Docker配置中。第一个参数是配置的名称，第二个参数是要从中读取的文件。

   ```sh
   $ docker config create site.conf site.conf
   ```

   列出配置：

   ```sh
   $ docker config ls
   ID                          NAME                CREATED             UPDATED
   4ory233120ccg7biwvy11gl5z   site.conf           4 seconds ago       4 seconds ago
   ```

4. 创建一个运行Nginx并可以访问两个秘密和配置的服务。将模式设置为`0440`使文件只能由其所有者和该所有者的组读取，而不是所有人。

   ```sh
   $ docker service create \
        --name nginx \
        --secret site.key \
        --secret site.crt \
        --config source=site.conf,target=/etc/nginx/conf.d/site.conf,mode=0440 \
        --publish published=3000,target=443 \
        nginx:latest \
        sh -c "exec nginx -g 'daemon off;'"
   ```

   在正在运行的容器中，现在存在以下三个文件：

   - `/run/secrets/site.key`
   - `/run/secrets/site.crt`
   - `/etc/nginx/conf.d/site.conf`

5. 验证Nginx服务正在运行。

   ```sh
   $ docker service ls
   ID            NAME   MODE        REPLICAS  IMAGE
   zeskcec62q24  nginx  replicated  1/1       nginx:latest
   
   $ docker service ps nginx
   NAME                  IMAGE         NODE  DESIRED STATE  CURRENT STATE          ERROR  PORTS
   nginx.1.9ls3yo9ugcls  nginx:latest  moby  Running        Running 3 minutes ago
   ```

6. 验证服务是否可操作：您可以访问Nginx服务器，并且正在使用正确的TLS证书。

   ```sh
   $ curl --cacert root-ca.crt https:// 192.168.99.106:3000
   
   $ openssl s_client -connect  192.168.99.106:3000 -CAfile root-ca.crt
   ```

7. 除非你打算继续下一个例子，否则在运行这个例子之后，通过删除`nginx`服务和存储的秘密和配置来清理。

   ```sh
   $ docker service rm nginx
   $ docker secret rm site.crt site.key
   $ docker config rm site.conf
   ```

现在配置了一个Nginx服务，其配置与其映像分离。您可以使用完全相同的映像运行多个站点，但可以单独配置，而无需构建自定义映像。

#### 旋转配置

要旋转配置，你首先保存一个与当前正在使用的名称不同的新配置。然后重新部署服务，删除旧配置并在容器中的相同安装点添加新配置。此示例通过旋转`site.conf` 配置文件构建在前一个示例上。

1. 在`site.conf`本地编辑文件。添加`index.php`到该`index`行，并保存该文件。

   ```nginx
   server {
       listen                443 ssl;
       server_name           localhost;
       ssl_certificate       /run/secrets/site.crt;
       ssl_certificate_key   /run/secrets/site.key;
   
       location / {
           root   /usr/share/nginx/html;
           index  index.html index.htm index.php;
       }
   }
   ```

2. 使用新的`site.conf`叫做的新建一个Docker配置`site-v2.conf`。

   ```sh
   $ docker config create site-v2.conf site.conf
   ```

3. 更新`nginx`服务以使用新配置而不是旧配置。

   ```sh
   $ docker service update \
     --config-rm site.conf \
     --config-add source=site-v2.conf,target=/etc/nginx/conf.d/site.conf,mode=0440 \
     nginx
   ```

4. 验证`nginx`服务是否完全重新部署，使用 `docker service ls nginx`。当它是，你可以删除旧的`site.conf` 配置。

   ```sh
   $ docker config rm site.conf
   ```

5. 为了清理，你可以删除`nginx`服务，以及秘密和配置。

   ```sh
   $ docker service rm nginx
   $ docker secret rm site.crt site.key
   $ docker config rm site-v2.conf
   ```

现在已经更新了`nginx`服务的配置，而无需重新构建其映像。

# docker secret 基本命令

```sh
$ docker secret -h
Usage:  docker secret COMMAND

管理敏感数据

Commands:
  create      # 创建秘密
  inspect     # 显示秘密
  ls          # 秘密列表
  rm          # 删除秘密
```

## create 创建

使用标准输入或秘密内容的文件创建一个秘密。

### 命令参数选项

---

| 选项，简写          | 默认 | 描述                                                         |
| ------------------- | ---- | ------------------------------------------------------------ |
| `--driver , -d`     |      | [API 1.37+](https://docs.docker.com/engine/api/v1.37/) 秘密驱动程序 |
| `--label , -l`      |      | 秘密标签                                                     |
| `--template-driver` |      | 模板驱动                                                     |

 ### 示例

---

#### 创建秘密

```sh
$ echo <secret> | docker secret create my_secret -
onakdyv307se2tl7nl20anokv

$ docker secret ls
ID                          NAME                CREATED             UPDATED
onakdyv307se2tl7nl20anokv   my_secret           6 seconds ago       6 seconds ago
```

#### 用文件创建秘密

```sh
$ docker secret create my_secret ./secret.json
dg426haahpi5ezmkkj5kyl3sn

$ docker secret ls
ID                          NAME                CREATED             UPDATED
dg426haahpi5ezmkkj5kyl3sn   my_secret           7 seconds ago       7 seconds ago
```

#### 用标签创建秘密

```sh
$ docker secret create --label env=dev \
                       --label rev=20170324 \
                       my_secret ./secret.json
eo7jnzguqgtpdah3cm5srfb97

$ docker secret inspect my_secret
```

## inspect 查看

查看指定的秘密。该命令必须在管理器节点上运行。默认情况下，这会将所有结果呈现在JSON数组中。如果指定了格式，则将为每个结果执行给定的模板。Go的[文本/模板](http://golang.org/pkg/text/template/)包描述了格式的所有细节。

```sh
$ docker secret inspect my_secret3
[
    {
        "ID": "xpo8llaj0j6wvqgmt6kmenbl6",
        "Version": {
            "Index": 19132
        },
        "CreatedAt": "2018-05-11T09:51:17.127938107Z",
        "UpdatedAt": "2018-05-11T09:51:17.127938107Z",
        "Spec": {
            "Name": "my_secret3",
            "Labels": {
                "env": "dev",
                "rev": "20170324"
            }
        }
    }
]
```

## ls 列表

### 过滤

---

过滤标志（`-f`或`--filter`）格式是一`key=value`对。如果有多个过滤器，则传递多个标志（例如`--filter "foo=bar" --filter "bif=baz"`）

目前支持的过滤器是：

- id（秘密ID）
- label（`label=<key>`或`label=<key>=<value>`）
- name（名字）

```sh
# 匹配一个秘密的id
$ docker secret ls -f "id=6697bflskwj1998km1gnnjr38"
# 带有project标签的秘密与其值进行匹配
$ docker secret ls --filter label=project
# 匹配project具有project=test值的标签
$ docker service ls --filter label=project=test
# 包含前缀的名称相匹配test
$ docker secret ls --filter name=test_secret
```

### 格式化输出

---

格式化选项（`--format`）可以使用Go模板打印秘密输出。下面列出了Go模板的有效占位符：

| 占位符       | 描述                                                        |
| ------------ | ----------------------------------------------------------- |
| `.ID`        | 秘密ID                                                      |
| `.Name`      | 秘密名称                                                    |
| `.CreatedAt` | 秘密被创建的时间                                            |
| `.UpdatedAt` | 秘密更新的时间                                              |
| `.Labels`    | 分配给秘密的所有标签                                        |
| `.Label`     | 这个秘密的特定标签的价值。例如`{{.Label "secret.ssh.key"}}` |

当使用该`--format`选项时，`secret ls`命令将完全按照模板声明输出数据，或者在使用该 `table`指令时也会包含列标题。

以下示例使用不带标题的模板，并输出 由冒号分隔的所有图像`ID`和`Name`条目：

```sh
$ docker secret ls --format "{{.ID}}: {{.Name}}"
```

要以表格格式列出所有秘密名称和创建日期，可以使用：  

```sh
$ docker secret ls --format "table {{.ID}}\t{{.Name}}\t{{.CreatedAt}}"
```

## rm 删除

从群中删除指定的秘密。该命令必须在管理器节点上运行。

```sh
$ docker secret rm secret.json
```

# 管理机密敏感数据

就Docker Swarm服务而言，**机密**是一组数据，如密码，SSH私钥，SSL证书或另一部分数据，这些数据**不应通过网络传输**或在Dockerfile或应用程序中**未加密存储源代码**。在Docker 1.13及更高版本中，可以使用Docker **机密**来集中管理这些数据，并将其安全地传输给需要访问的那些容器。在运输过程中**密码被加密**，并在Docker群中休息。给定的**秘密**只能被那些被**授予明确访问权限**的服务访问，并且只有在这些服务任务**正在运行**时才能访问。

可以使用秘密来管理容器在运行时需要的任何敏感数据，但不想将它存储在图像或源代码管理中，下面的情况适合：

- 用户名和密码
- TLS证书和密钥
- SSH密钥
- 其他重要数据，如数据库或内部服务器的名称
- 通用字符串或二进制内容（最大为500 kb）

> **注意**：Docker机密**仅适用于群集**服务，而**不适用于独立容器**。要使用此功能，请考虑**将容器作为服务**运行。有状态的容器通常可以在不更改容器代码的情况下以1个实例运行。

另一个使用秘密的用例是在容器和一组证书之间提供一个**抽象层**。考虑一个场景，可以为应用程序分别开发，测试和生产环境。这些环境中的每一个都可以拥有不同的凭证，并以相同的秘密名称存储在开发，测试和生产群中。容器只需要知道在所有三种环境中运行的秘密的名称。

还可以使用机密来管理**非敏感数据**，例如配置文件。但是，Docker 17.06和更高版本支持使用[configs](https://docs.docker.com/engine/swarm/configs/) 来存储非敏感数据。配置直接安装到容器的**文件系统**中，而不使用RAM磁盘。

## 如何管理机密

当为swarm添加秘密时，Docker会通过相互**TLS连接**将密钥发送给swarm管理器。秘密存储在**加密的Raft日志**中。**整个Raft日志被复制**到其他管理节点中，确保与其他群管理数据相同的高可用性保证。

> **警告**：Raft数据在Docker 1.13和更高版本中被**加密**。如果任何Swarm管理者运行早期版本，并且其中一位经理成为群组的管理者，则秘密将在节点的Raft日志中**未加密存储**。在添加任何秘密之前，**将所有管理器节点更新到Docker 1.13或更高版本**，以**防止将秘密写入纯文本**的Raft日志。

当授予新创建或正在运行的服务对秘密的访问权限时，解密的秘密将被装入容器中的**内存中文件**系统。容器中安装点的位置默认为 `/run/secrets/<secret_name>`在Linux容器或 `C:\ProgramData\Docker\secrets`Windows容器中。可以在Docker 17.06和更高版本中指定自定义位置。

可以更新服务，授权其访问其他机密或**随时撤销对指定机密的访问权限**。

如果节点是群管理器或者它正在运行已**被授权访问秘密的服务**任务，那么节点**只能访问（加密的）秘密**。当容器任务停止运行时，共享给它的*解密秘密**将从该容器的内存中文件系统**卸载**，并从节点的内存**刷新*。<br/>如果节点在运行可访问秘密的任务容器时失去与群集的连接，则任务容器仍可访问其秘密，但在节点重新连接群集之前无法接收更新。

可以随时**添加或检查**个人秘密，或列出所有秘密。**无法删除正在运行**的服务正在使用的秘密。请参阅[旋转秘密](https://docs.docker.com/engine/swarm/secrets/#example-rotate-a-secret)以在不中断正在运行的服务的情况下移除秘密。<br/>要更容易地更新或回滚机密，请考虑在秘密名称中添加**版本号或日期**。通过控制给定容器内**秘密的安装点**的能力使这变得更容易。

## 实例演示

### 开始秘密

---

这个简单的例子展示了秘密如何在几个命令中工作。对于真实世界的例子，继续到 [中级例子：使用Nginx服务的秘密](https://docs.docker.com/engine/swarm/secrets/#intermediate-example-use-secrets-with-a-nginx-service)。

1. 给Docker添加一个秘密。`docker secret create`命令读取标准输入，因为最后一个参数表示要读取密钥的文件设置为`-`。

   ```sh
   $ printf "This is a secret" | docker secret create my_secret_data -
   ```

2. 创建一个`redis`服务并授予它访问秘密的权限。默认情况下，容器可以访问秘密`/run/secrets/<secret_name>`，但可以使用`target`选项自定义容器上的文件名。

   ```sh
   $ docker service  create --name redis --secret my_secret_data redis:alpine
   ```

3. 验证任务是否正在运行，没有问题`docker service ps`。如果一切正常，输出结果如下所示：

   ```sh
   $ docker service ps redis
   ID            NAME     IMAGE         NODE              DESIRED STATE  CURRENT STATE          ERROR  PORTS
   bkna6bpn8r1a  redis.1  redis:alpine  ip-172-31-46-109  Running        Running 8 seconds ago  
   ```

   如果出现错误，并且任务失败并反复重新启动，则会看到如下所示的内容：

   ```sh
   $ docker service ps redis
   NAME                      IMAGE         NODE  DESIRED STATE  CURRENT STATE          ERROR                      PORTS
   redis.1.siftice35gla      redis:alpine  moby  Running        Running 4 seconds ago                             
    \_ redis.1.whum5b7gu13e  redis:alpine  moby  Shutdown       Failed 20 seconds ago      "task: non-zero exit (1)"  
    \_ redis.1.2s6yorvd9zow  redis:alpine  moby  Shutdown       Failed 56 seconds ago      "task: non-zero exit (1)"  
    \_ redis.1.ulfzrcyaf6pg  redis:alpine  moby  Shutdown       Failed about a minute ago  "task: non-zero exit (1)"  
    \_ redis.1.wrny5v4xyps6  redis:alpine  moby  Shutdown       Failed 2 minutes ago       "task: non-zero exit (1)"
   ```

4. 获取`redis`使用的服务任务容器的ID `docker ps`，以便您可以使用`docker container exec`连接到容器并读取秘密数据文件的内容，该内容默认为全部可读，并且与秘密的名称相同。下面的第一条命令说明了如何找到容器ID，第二个和第三个命令使用shell完成来自动执行此操作。

   ```sh
   $ docker-machine ssh manager1
   
   $ docker ps --filter name=redis -q
   5cb1c2348a59
   
   $ docker container exec $(docker ps --filter name=redis -q) ls -l /run/secrets
   total 4
   -r--r--r--    1 root     root            17 Dec 13 22:48 my_secret_data
   
   $ docker container exec $(docker ps --filter name=redis -q) cat /run/secrets/my_secret_data
   This is a secret
   ```

5. 如果提交容器，秘密**不可**用。

   ```sh
   $ docker commit $(docker ps --filter name=redis -q) committed_redis
   
   $ docker run --rm -it committed_redis cat /run/secrets/my_secret_data
   cat: can't open '/run/secrets/my_secret_data': No such file or directory
   ```

6. 尝试删除秘密。删除失败，因为该`redis`服务正在运行并可以访问该秘密。

   ```sh
   $ docker secret ls
   ID                          NAME                CREATED             UPDATED
   wwwrxza8sxy025bas86593fqs   my_secret_data      4 hours ago         4 hours ago
   
   $ docker secret rm my_secret_data
   Error response from daemon: rpc error: code = 3 desc = secret
   'my_secret_data' is in use by the following service: redis
   ```

7. `redis`通过更新服务，从正在运行的服务中移除对秘密的访问权限。

   ```
   $ docker service update --secret-rm my_secret_data redis
   ```

8. 重复步骤3和4，验证该服务不再有权访问该秘密。容器ID不同，因为该 `service update`命令重新部署服务。

   ```sh
   $ docker container exec -it $(docker ps --filter name=redis -q) cat /run/secrets/my_secret_data
   
   cat: can't open '/run/secrets/my_secret_data': No such file or directory
   ```

9. 停止并删除服务，并从Docker中删除密钥。

   ```sh
   $ docker service rm redis
   $ docker secret rm my_secret_data
   ```

### 使用WordPress服务的秘密

---

使用自定义root密码创建单节点MySQL服务，将凭证添加为秘密，并创建使用这些凭证连接到MySQL的单节点WordPress服务。在 [下面的例子](https://docs.docker.com/engine/swarm/secrets/#example-rotate-a-secret)建立在这一个，并告诉如何旋转MySQL的密码和更新服务，使WordPress的服务仍然可以连接到MySQL。

此示例说明了一些使用Docker机密的技巧，以**避免将敏感凭证保存在映像中或直接在命令行上**传递。

> **注意**：为简单起见，此示例使用单引擎群，并使用单节点MySQL服务，因为单个MySQL服务器实例无法通过简单地使用复制服务来扩展，并且设置MySQL群集超出了本示例的范围。<br/>另外，更改MySQL密码并不像更改磁盘上的文件那么简单。必须使用查询或`mysqladmin`命令来更改MySQL中的密码。

1. 为MySQL生成一个**随机的字母**数字密码，并`mysql_password`使用该`docker secret create` 命令将其作为Docker机密存储。要使密码更短或更长，请调整`openssl`命令的最后一个参数。这只是创建相对随机密码的一种方式。如果选择可以使用其他命令来生成密码。

   > **注意**：**创建秘密后，无法更新它。只能删除并重新创建它**，并且无法删除服务正在使用的秘密。但是，可以使用**授予或撤销正在运行的服务**对秘密的访问权限`docker service update`。如果需要更新密码的功能，请考虑在密码名称中添加一个版本组件，以便稍后添加新版本，更新服务以使用它，然后删除旧版本。

   最后一个参数设置为`-`，表示输入是从标准输入读取的。

   ```sh
   $ openssl rand -base64 20 | docker secret create mysql_password -
   l1vinzevzhj4goakjap5ya409
   ```

   返回的值不是密码，而是密码的ID。在本教程的其余部分中，ID输出被省略。

   为MySQL `root`用户生成第二个秘密。这个秘密不会与稍后创建的WordPress服务共享。它只需要引导`mysql`服务。

   ```sh
   $ openssl rand -base64 20 | docker secret create mysql_root_password -
   ```

   列出由Docker管理的秘密`docker secret ls`：

   ```sh
   $ docker secret ls
   
   ID                          NAME                  CREATED             UPDATED
   l1vinzevzhj4goakjap5ya409   mysql_password        41 seconds ago      41 seconds ago
   yvsczlx9votfw3l0nz5rlidig   mysql_root_password   12 seconds ago      12 seconds ago
   ```

   秘密存储在丛集的加密的Raft日志中。

2. 创建用于MySQL和WordPress服务之间通信的用户定义覆盖网络。不需要将MySQL服务公开给任何外部主机或容器。

   ```sh
   $ docker network create -d overlay mysql_private
   ```

3. 创建MySQL服务。MySQL服务具有以下特征：

   - 由于该比例设置为`1`，所以只有一个MySQL任务运行。负载均衡MySQL作为练习留给读者，不仅涉及扩展服务。

   - 只能由`mysql_private`网络上的其他容器访问。

   - 使用卷`mydata`存储MySQL数据，以便在重新启动`mysql`服务时保持不变。

   - 秘密都安装在and `tmpfs`处的文件系统中 。它们从不作为环境变量公开，如果命令运行，它们也不会承诺映像。 秘密是非特权WordPress容器连接到MySQL所使用的秘密。`/run/secrets/mysql_password``/run/secrets/mysql_root_password``docker commit``mysql_password`

   - 设置环境变量`MYSQL_PASSWORD_FILE`并 `MYSQL_ROOT_PASSWORD_FILE`指向文件`/run/secrets/mysql_password`和`/run/secrets/mysql_root_password`。`mysql`第一次初始化系统数据库时，映像会从这些文件中读取密码字符串。之后，密码存储在MySQL系统数据库本身中。

   - 设置环境变量`MYSQL_USER`和`MYSQL_DATABASE`。被调用的新数据库`wordpress`在容器启动时被创建，并且`wordpress`用户仅对该数据库具有完全权限。该用户不能创建或删除数据库或更改MySQL配置。

     ```
     $ docker service create \
          --name mysql \
          --replicas 1 \
          --network mysql_private \
          --mount type=volume,source=mydata,destination=/var/lib/mysql \
          --secret source=mysql_root_password,target=mysql_root_password \
          --secret source=mysql_password,target=mysql_password \
          -e MYSQL_ROOT_PASSWORD_FILE="/run/secrets/mysql_root_password" \
          -e MYSQL_PASSWORD_FILE="/run/secrets/mysql_password" \
          -e MYSQL_USER="wordpress" \
          -e MYSQL_DATABASE="wordpress" \
          mysql:latest
     ```

4. 验证`mysql`容器是否正在使用该`docker service ls`命令运行。

   ```
   $ docker service ls
   
   ID            NAME   MODE        REPLICAS  IMAGE
   wvnh0siktqr3  mysql  replicated  1/1       mysql:latest
   ```

   此时，您可以实际撤消`mysql`服务对密码`mysql_password`和`mysql_root_password`秘密的访问权限， 因为密码已保存在MySQL系统数据库中。现在不要这样做，因为我们稍后使用它们来方便旋转MySQL密码。

5. 现在MySQL已经建立，创建一个连接到MySQL服务的WordPress服务。WordPress服务具有以下特点：

   - 由于该比例设置为`1`，所以只有一个WordPress任务运行。由于将WordPress会话数据存储在容器文件系统上的限制，因此负载均衡WordPress作为练习留给读者。
   - 在主机的端口30000上显示WordPress，以便您可以从外部主机访问它。如果您没有在主机的端口80上运行Web服务器，则可以公开端口80。
   - 连接到`mysql_private`网络，以便它可以与`mysql`容器通信 ，并在所有群集节点上发布端口80到端口30000。
   - 有权访问`mysql_password`密码，但在容器中指定不同的目标文件名称。WordPress容器使用安装点`/run/secrets/wp_db_password`。还通过将模式设置为，指定秘密不是群组或世界可读的 `0400`。
   - 将环境变量设置`WORDPRESS_DB_PASSWORD_FILE`为安装密钥的文件路径。WordPress服务从该文件读取MySQL密码字符串并将其添加到`wp-config.php` 配置文件中。
   - 使用用户名`wordpress`和密码连接到MySQL容器，`/run/secrets/wp_db_password`并创建`wordpress` 数据库（如果它尚不存在）。
   - 将其数据（如主题和插件）存储在一个名为的卷中，`wpdata` 以便在重新启动服务时保持这些文件。

   ```
   $ docker service create \
        --name wordpress \
        --replicas 1 \
        --network mysql_private \
        --publish published=30000,target=80 \
        --mount type=volume,source=wpdata,destination=/var/www/html \
        --secret source=mysql_password,target=wp_db_password,mode=0400 \
        -e WORDPRESS_DB_USER="wordpress" \
        -e WORDPRESS_DB_PASSWORD_FILE="/run/secrets/wp_db_password" \
        -e WORDPRESS_DB_HOST="mysql:3306" \
        -e WORDPRESS_DB_NAME="wordpress" \
        wordpress:latest
   ```

6. 验证服务正在使用`docker service ls`和 `docker service ps`命令运行。

   ```
   $ docker service ls
   
   ID            NAME       MODE        REPLICAS  IMAGE
   wvnh0siktqr3  mysql      replicated  1/1       mysql:latest
   nzt5xzae4n62  wordpress  replicated  1/1       wordpress:latest
   ```

   ```
   $ docker service ps wordpress
   
   ID            NAME         IMAGE             NODE  DESIRED STATE  CURRENT STATE           ERROR  PORTS
   aukx6hgs9gwc  wordpress.1  wordpress:latest  moby  Running        Running 52 seconds ago   
   ```

   此时，您实际上可以撤消WordPress服务对`mysql_password`秘密的访问权限，因为WordPress已将秘密复制到其配置文件中`wp-config.php`。现在不要这样做，因为我们稍后会使用它来方便旋转MySQL密码。

7. `http://localhost:30000/`使用基于Web的向导访问任何swarm节点并设置WordPress。所有这些设置都存储在MySQL `wordpress`数据库中。WordPress自动为您的WordPress用户生成密码，这与WordPress用于访问MySQL的密码完全不同。安全地存储此密码，例如在密码管理器中。[旋转秘密](https://docs.docker.com/engine/swarm/secrets/#example-rotate-a-secret)后，您需要它登录WordPress 。

   继续撰写一篇或两篇博文，并安装一个WordPress插件或主题，以验证WordPress是否完全正常运行，并且在服务重新启动时保存其状态。

8. 如果您打算继续下一个示例，演示如何旋转MySQL根密码，请不要清除任何服务或秘密。















如果还没有连接到`manager1`，请打开一个终端并将ssh连入运行管理节点`manager1`的机器中。

管理者、管理员、管理器-管理节点

工作人员、工作者——工作节点

管理-管理

集群-集群

群体-集群

你-你

选项-选项

镜像、镜像、镜像-镜像

秘密-私密

网格--网络

其

偏好-偏好