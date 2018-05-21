# docker compose 服务编排



# compose 用例参考
使用Compose堆栈文件来实现多容器应用程序，服务定义和集群模式。文件结构参考：

```yaml
version: "3"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]

  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - 5000:80
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure

  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - 5001:80
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```



# compose 服务编排配置详解

`Compose`文件是一个定义[服务](https://docs.docker.com/compose/compose-file/#service-configuration-reference)、 [网络](https://docs.docker.com/compose/compose-file/#network-configuration-reference)和 [卷](https://docs.docker.com/compose/compose-file/#volume-configuration-reference)的[YAML](http://yaml.org/)文件 。`Compose`文件的默认路径是。`./docker-compose.yml`

> **提示**：可以对此文件使用 `.yml`或`.yaml`扩展名都支持。

服务定义包含应用于为该服务启动的每个容器的配置，就像命令行参数传递给 `docker container create`一样。同样，网络和卷的定义类似于 `docker network create`和`docker volume create`。

正如`docker container create`在`Dockerfile`指定选项，如`CMD`， `EXPOSE`，`VOLUME`，`ENV`，注意在默认情况下——你不需要再次指定它们`docker-compose.yml`。

您可以使用Bash类`${VARIABLE}`语法在配置值中使用环境变量 - 有关完整详细信息，请参阅 [变量替换](https://docs.docker.com/compose/compose-file/#variable-substitution)。

下面介绍服务定义所支持的所有配置选项的列表。需要创建好文件，文件目录结构如下：

```shell
D:\DOCKER\COMPOSE_SAMPLE
│  docker-compose.yml
│
└─sample
        app.py
        Dockerfile
        Dockerfile-alternate
        requirements.txt
```

`docker-compose.yml` 将是下面演示需要修改的文件，`sample` 目录存放应用程序的文件。



## `build` 构建

在构建时应用的配置选项。

`build`可以指定为构建上下文的路径，**`build`后面的字符串将指向构建程序的真实路径**，下面的代码build指向的路径是 当前`compose`文件所在目录下的`sample `目录。

```yaml
version: '3'
services:
  webapp:
    build: ./sample # 当前docker-compose.yml文件所在目录下的 sample目录
```

在 `docker-compose.yml`文件所在目录中执行命令 `docker-compose build`构建程序
```shell
$ docker-compose build
Building webapp
.....
```

或者，用`context`指定应用的路径，并且可以使用`dockerfile` 指定`Dockerfile` 文件名称和`args` 指定当前构建的参数数据，作为环境变量使用：

```yaml
version: '3'
services:
  webapp:
    build:
      context: ./sample  # 当前docker-compose.yml文件所在目录下的 sample目录
      dockerfile: Dockerfile-alternate # 备用的 dockerfile 文件名称
      args: # 构建参数
        buildno: 1 
```

如果指定`image` 和 `build`，那么文件中指定的`image: webapp:first`将是构建镜像名称：

```yaml
  webapp:
    image: webapp:first # 镜像名称
    build:
      context: ./sample     
```

随后可以运行命令`docker-compose config`检查 `docker-compose.yml`配置是否正确。

**这将产生一个名为`webapp`和`tag`标记为`first`的镜像，该镜像由目录`./sample`内容构建。**

> **注意**：当 使用（版本3）Compose文件[在集群模式下部署编排堆栈服务时，](https://docs.docker.com/engine/reference/commandline/stack_deploy/)忽略此选项 。`docker stack`命令仅接受预先构建的镜像。



### `context` 上下文

---

可以是`Dockerfile`所在的目录的路径，也可以是`Git`仓库的`url`。

当`context`的值是相对路径时，它被解释为相对于`docker-compose.yam`文件的位置。这个目录也是推送到Docker守护进程的构建上下文。

```yaml
build:
  context: ./sample
```



### `dockerfile` 

---

备用`Dockerfile`。`compose`命令使用替代文件来构建，还必须指定构建路径 `build` 的值。

```yacas
 build:
      context: ./sample
      dockerfile: Dockerfile-alternate
```

在 `sample` 目录中必须有文件名称为 `Dockerfile-alternate`的`dockerfile`



### `args` 参数

---

添加构建参数，这些参数是仅在构建过程中可访问的环境变量。

首先，在`Dockerfile-alternate`中指定参数：

```dockerfile
ARG buildno
ARG author

RUN echo "Build number: $buildno"
RUN echo "Build author: $author"
```

然后指定`build`键下的参数。可以传递一个映射或一个列表：

```yaml
build:
  context: .
  args:
    buildno: 1
    author: hoojo
    
### 或者如下写法
build:
  context: .
  args:
    - buildno: 1
    - author: hoojo    
```

重新构建后会看到输出信息中出现我们定义输出的内容

```shell
$ docker-compose build

Step 6/9 : RUN echo "Build number: ${buildno}"
 ---> Running in 1611c23cbdb6
Build number: 1
Removing intermediate container 1611c23cbdb6
 ---> e2ea0d5585da
Step 7/9 : RUN echo "Build author: $author"
 ---> Running in fc12e0ee505b
Build author: hoojo
```

指定构建参数时可以省略该值，在这种情况下，构建时的值是运行环境中的值。

```yaml
args:
  - buildno
  - author
```

> **注**：YAML布尔值（`true`，`false`，`yes`，`no`，`on`，`off`）必须用引号括起来，以便解析器将它们解释为字符串。



### `cache_from` 缓存

---

引擎用于缓存解析的镜像列表。**在版本：3.2 可用**

```yaml
build:
  context: .
  cache_from:
    - redis:latest
    - hoojo/web_app:3.14
```



### `lables` 标签

---

使用[Docker标签](https://docs.docker.com/engine/userguide/labels-custom-metadata/)将元数据添加到生成的镜像中。可以使用数组或字典。建议使用反向DNS标记来防止标签与其他软件使用的标签冲突。**注意：**这个选项在v3.3中可用

```shell
build:
  context: .
  labels:
    com.example.description: "Accounting webapp"
    com.example.department: "Finance"
    com.example.label-with-empty-value: ""

build:
  context: .
  labels:
    - "com.example.description=Accounting webapp"
    - "com.example.department=Finance"
    - "com.example.label-with-empty-value"
```



### `shm_size` 分区大小

---

设置`/dev/shm`容器的分区大小。指定为表示字节数的整数值或表示[字节值](https://docs.docker.com/compose/compose-file/#specifying-byte-values)的字符串。**在3.5版本中可用**

```yaml
build:
  context: .
  shm_size: '2gb'


build:
  context: .
  shm_size: 10000000
```



### `target` 目标

---

根据构建阶段在`Dockerfile`中定义指定目标配置。**在3.4版本可用**

```yaml
  build:
    context: .
    target: prod # 在dockerfile中的prod 构建阶段
```



## `cap_add，cap_drop` 添加删除容器

添加或删除容器功能。请参阅`man 7 capabilities`完整列表。

```yaml
cap_add:
  - ALL

cap_drop:
  - NET_ADMIN
  - SYS_ADMIN
```

> **注意**：[在](https://docs.docker.com/engine/reference/commandline/stack_deploy/) 使用（版本3）Compose文件的[群集模式下部署堆栈](https://docs.docker.com/engine/reference/commandline/stack_deploy/)时，会忽略这些选项 。



## command 命令

启动后执行的命令，会覆盖默认命令。

```yaml
command: bundle exec thin -p 3000
```

该命令也可以是一个列表，方式类似于 [dockerfile](https://docs.docker.com/engine/reference/builder/#cmd)：

```yaml
command: ["bundle", "exec", "thin", "-p", "3000"]
```

在`docker-compose.yml`文件中加入配置`    command: echo "hello!"` ，执行启动后发现输出日志

```shell
$ docker-compose up
Creating network "composesample_default" with the default driver
Creating composesample_webapp_1 ... done
Attaching to composesample_webapp_1
webapp_1  | hello!
composesample_webapp_1 exited with code 0
```



## configs 配置

使用每项服务`configs` 配置为每个服务授予对配置的访问权限。支持两种不同的语法变体。

> **注意**：配置必须已经存在或 [在`configs`](https://docs.docker.com/compose/compose-file/#configs-configuration-reference) 此堆栈文件[的顶层](https://docs.docker.com/compose/compose-file/#configs-configuration-reference)[配置](https://docs.docker.com/compose/compose-file/#configs-configuration-reference)中[定义](https://docs.docker.com/compose/compose-file/#configs-configuration-reference)，否则堆栈部署失败。



### 短语法

短的语法体只能指定配置名称。这会授予容器对配置的访问权限并将其装载到容器中的`/<config_name>`处。源名称和目标装入点都设置为配置名称。

以下示例使用简短语法将`redis`服务访问权限授予`my_config`和`my_other_config`configs。值`my_config`被设置为文件的内容`./my_config.txt`，并被 `my_other_config`定义为外部资源，这意味着它已经在Docker中定义，可以通过运行该`docker config create` 命令或通过另一个堆栈部署。如果外部配置不存在，则堆叠部署失败并出现`config not found`错误。

> **注**：`config`定义仅在3.3版及更高版本的compose文件格式中受支持。

```yaml
version: "3.3"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - my_config
      - my_other_config
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```

### 长语法

长的语法提供了更多的粒度，以便在服务的任务容器中如何配置。

- `source`：它在Docker中存在的配置名称。
- `target`：要在服务的任务容器中装载的文件的路径和名称。如果未指定，则默认为`/<source>`。
- `uid`以及`gid`：在服务的任务容器中拥有安装的配置文件的数字UID或GID。`0`如果未指定，则默认为在Linux上。Windows不支持。
- `mode`：在服务的任务容器中安装的文件的权限，以八进制表示法。例如，`0444` 代表世界可读的。默认是`0444`。配置文件无法写入，因为它们安装在临时文件系统中，所以如果设置了可写位，它将被忽略。可执行位可以被设置。如果您不熟悉UNIX文件权限模式，则可能会发现此 [权限计算器](http://permissions-calculator.org/) 很有用。

以下示例在容器中将`my_config`的名称设置为`redis_config`，将模式设置为`0440`（可读组），并将用户和组设置为`103`. `redis`服务无法访问`my_other_config`配置。

```yaml
version: "3.5"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - source: my_config
        target: /redis_config
        uid: '103'
        gid: '103'
        mode: 0440
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```

您可以授予多个配置的服务访问权限，并且您可以混合使用长短语法。定义配置并不意味着授予服务访问权限。



## cgroup_parent 上级组

为容器指定一个可选的父cgroup。

```yaml
cgroup_parent: m-executor-abcd
```

> **注意**：当 使用（版本3）Compose文件[在群集模式下部署堆栈时，](https://docs.docker.com/engine/reference/commandline/stack_deploy/)忽略此选项 。



## container_name 容器名称

指定一个自定义容器名称，而不是生成的默认名称。

```yaml
version: '3.5'
services:
  webapp:
    container_name: webapp_first
    image: webapp:first
```

由于Docker容器名称必须是唯一的，因此如果指定了自定义名称，则无法将服务扩展到1个容器之外。试图这样做会导致错误。

> **注意**：当 使用（版本3）Compose文件[在群集模式下部署堆栈时，](https://docs.docker.com/engine/reference/commandline/stack_deploy/)忽略此选项 。

```shell
$ docker-compose.exe ps
    Name         Command     State    Ports
-------------------------------------------
webapp_first   echo hello!   Exit 0
```



## credential_spec 凭据规范

> **注意：**该选项已添加到v3.3中

为托管服务帐户配置凭据规范。此选项仅用于使用Windows容器的服务。在`credential_spec`必须在格式`file://<filename>`或`registry://<value-name>`。

使用时`file:`，引用的文件必须存在于`CredentialSpecs` docker数据目录的子目录中，该目录默认为`C:\ProgramData\Docker\` 在Windows上。以下示例从名为的文件加载凭证规范`C:\ProgramData\Docker\CredentialSpecs\my-credential-spec.json`：

```yaml
credential_spec:
  file: my-credential-spec.json
```

使用时`registry:`，将从守护进程主机上的Windows注册表中读取凭据规范。具有给定名称的注册表值必须位于：

```
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\CredentialSpecs
```

以下示例从`my-credential-spec` 注册表中指定的值加载凭证规范：

```yaml
credential_spec:
  registry: my-credential-spec
```



## deploy 部署


> **仅限版本3。**

指定与服务的部署和运行相关的配置。这只在部署到使用`docker stack`部署的`swarm`时才会生效，并且被`docker-compose up`和`docker-compose run`忽略。

```yaml
version: '3'
services:
  redis:
    image: redis:alpine
    deploy:
      replicas: 6
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
```

### endpoint_mode 端点模式
---
为连接到群的外部客户端指定服务发现方法。

> **仅版本3.3。**

- `endpoint_mode: vip` - Docker为该服务分配一个虚拟IP（VIP），作为客户端到达网络服务的“前端”。Docker在客户端和可用的工作节点之间为服务路由请求，而客户端不知道有多少节点参与服务或其IP地址或端口。（这是默认设置。）
- `endpoint_mode: dnsrr` - DNS轮询（DNSRR）服务发现不使用单个虚拟IP。Docker为服务设置DNS条目，使得服务名称的DNS查询返回一个IP地址列表，并且客户端直接连接到其中的一个。如果您想使用自己的负载平衡器，或者混合Windows和Linux应用程序，则DNS轮询功能非常有用。

```yaml
version: "3.3"

services:
  wordpress:
    image: wordpress
    ports:
      - 8080:80
    networks:
      - overlay
    deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: vip

  mysql:
    image: mysql
    volumes:
       - db-data:/var/lib/mysql/data
    networks:
       - overlay
    deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: dnsrr

volumes:
  db-data:

networks:
  overlay:
```

这些选项`endpoint_mode`也可用作集群模式CLI命令[docker service create中的](https://docs.docker.com/engine/reference/commandline/service_create/)选项 。有关所有swarm相关`docker`命令的快速列表，请参阅[Swarm模式CLI命令](https://docs.docker.com/engine/swarm/#swarm-mode-key-concepts-and-tutorial)。

要了解有关集群模式下的服务发现和网络的更多信息，请参阅 在群集模式主题中[配置服务发现](https://docs.docker.com/engine/swarm/networking/#configure-service-discovery)。

### labels 标签
---
指定服务的标签。这些标签*仅*在服务上设置，*而不*在服务的任何容器上设置。

```yaml
version: "3"
services:
  web:
    image: web
    deploy:
      labels:
        com.example.description: "This label will appear on the web service"
```

要改为在容器上设置标签，请在以下位置使用`labels`，而并非在`deploy` 键下面：

```yaml
version: "3"
services:
  web:
    image: web
    labels:
      com.example.description: "This label will appear on all containers for the web service"
```

### mode 模式
---
全局唯一模式：`global`，还是副本多实例模式：`replicated`，若是副本模式就需要指定副本数量`replicas`。要么`global`（每个群集节点只有一个容器）或`replicated`（指定数量的容器）。默认是`replicated`。（要了解更多信息，请参阅[swarm](https://docs.docker.com/engine/swarm/)主题 中的[复制和全局服务](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/#replicated-and-global-services)。）

```yaml
version: '3'
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    deploy:
      mode: global
```

### placement 分布位置
---
指定约束和偏好的位置。请参阅docker服务创建文档以获取语法和可用类型的[约束](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint)和[首选项](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-placement-preferences-placement-pref)的完整说明。

```yaml
version: '3'
services:
  db:
    image: postgres
    deploy:
      placement:
        constraints:
          - node.role == manager
          - engine.labels.operatingsystem == ubuntu 14.04
        preferences:
          - spread: node.labels.zone
```

### replicas 副本
---
如果服务是`replicated`（这是默认设置），请指定在任何给定时间应该运行的容器数量。

```yaml
version: '3'
services:
  worker:
    image: dockersamples/examplevotingapp_worker   
    deploy:
      mode: replicated
      replicas: 6
```

### resources 资源
---
配置限制资源。

> **注意**：这取代了[旧的资源约束选项](https://docs.docker.com/compose/compose-file/compose-file-v2/#cpu-and-other-resources)在compose非集群模式文件之前版本3（ `cpu_shares`，`cpu_quota`，`cpuset`，`mem_limit`，`memswap_limit`，`mem_swappiness`）如在[升级版本2.x到3.x](https://docs.docker.com/compose/compose-file/compose-versioning/#upgrading)。

这些都是单一的值，类似于[docker服务创建的](https://docs.docker.com/engine/reference/commandline/service_create/)对应服务。

在这个通用示例中，`redis`服务限制使用不超过`50M`的内存和`0.50`（50％）可用处理时间（CPU），并且 保留`20M`了内存和`0.25`CPU时间（如同始终可用的那样）。

```yaml
version: '3'
services:
  redis:
    image: redis:alpine
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 50M
        reservations: # 保留资源，避免程序内存溢出
          cpus: '0.25'
          memory: 20M
```

下面的主题描述了设置群集中服务或容器资源约束的可用选项。

> 寻找在非群模式容器上设置资源的选项？
>
> 这里描述的选项特定于 `deploy`密钥和群集模式。如果要为非集群部署设置资源约束，请使用 [Compose文件格式版本2 CPU，内存和其他资源选项](https://docs.docker.com/compose/compose-file/compose-file-v2/#cpu-and-other-resources)。如果您还有其他问题，请参阅关于GitHub问题[docker /compose/4513](https://github.com/docker/compose/issues/4513)的讨论。



#### Out Of Memory Exceptions 内存异常（OOME）

如果服务或容器尝试使用比系统可用的内存更多的内存，则可能会遇到内存异常（`OOME`），并且容器或Docker守护程序可能会被内核OOM错误所杀。要防止发生这种情况，请确保应用程序在具有足够内存的主机上运行，并且请参阅[了解耗尽内存的风险](https://docs.docker.com/engine/admin/resource_constraints/#understand-the-risks-of-running-out-of-memory)。

### restart_policy 重启策略
---
配置是否以及如何在退出时重新启动容器。取代 [`restart`](https://docs.docker.com/compose/compose-file/compose-file-v2/#orig-resources)。

- `condition`：其中之一`none`，`on-failure`或`any`，默认：`any`。
- `delay`：在重启尝试之间等待多长时间，指定为 [持续时间](https://docs.docker.com/compose/compose-file/#specifying-durations)（默认值：0）。
- `max_attempts`：在放弃之前尝试重新启动容器多少次（默认：从不放弃）。如果重新启动在配置中没有成功`window`，则此尝试不计入配置`max_attempts`值。例如，如果`max_attempts`设置为“2”，并且第一次尝试重新启动失败，则可能会尝试重新启动两次以上。
- `window`：在决定重新启动是否成功之前等待多久，指定为[持续时间](https://docs.docker.com/compose/compose-file/#specifying-durations)（默认值：立即决定）。

```
version: "3"
services:
  redis:
    image: redis:alpine
    deploy:
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
```

### update_config 更新配置
---
配置如何更新服务，用于配置滚动更新。

- `parallelism`：一次更新的容器数量。
- `delay`：在更新一组容器之间等待的时间。。
- `failure_action`：如果更新失败该怎么办。其中`continue`，`rollback`或`pause` ，默认：`pause`。
- `monitor`：每次任务更新后，监控是否失败的时间`(ns|us|ms|s|m|h)`（默认为0）。
- `max_failure_ratio`：在更新期间容忍的失败率。
- `order`：更新期间的操作顺序。其中一个`stop-first`（旧的任务在开始新任务之前停止），或`start-first`（新的任务首先启动，并且正在运行的任务短暂重叠）（默认`stop-first`）**注意**：仅在v3.4和更高版本中受支持。

> **注意**：`order`仅支持v3.4及更高版本的撰写文件格式。

```yaml
version: '3.4'
services:
  vote:
    image: dockersamples/examplevotingapp_vote:before
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
        order: stop-first
```

## 不支持 `DOCKER STACK DEPLOY`

下面的子选项（支持`docker compose up`和`docker compose run`）是**不支持**的`docker stack deploy`或`deploy`关键字的。

- [build](https://docs.docker.com/compose/compose-file/#build)
- [cgroup_parent](https://docs.docker.com/compose/compose-file/#cgroup_parent)
- [container_name](https://docs.docker.com/compose/compose-file/#container_name)
- [devices](https://docs.docker.com/compose/compose-file/#devices)
- [tmpfs](https://docs.docker.com/compose/compose-file/#tmpfs)
- [external_links](https://docs.docker.com/compose/compose-file/#external_links)
- [links](https://docs.docker.com/compose/compose-file/#links)
- [network_mode](https://docs.docker.com/compose/compose-file/#network_mode)
- [restart](https://docs.docker.com/compose/compose-file/#restart)
- [security_opt](https://docs.docker.com/compose/compose-file/#security_opt)
- [stop_signal](https://docs.docker.com/compose/compose-file/#stop_signal)
- [sysctls](https://docs.docker.com/compose/compose-file/#sysctls)
- [userns_mode](https://docs.docker.com/compose/compose-file/#userns_mode)

> **提示**：请参阅关于[如何为服务，群集和docker-stack.yml文件配置卷](https://docs.docker.com/compose/compose-file/#volumes-for-services-swarms-and-stack-files)的部分。支持卷*，*但要与群集和服务一起使用，它们必须配置为命名卷或与限制为可访问必需卷的节点的服务相关联

## devices 设备

设备映射列表。使用方式和`--device`Docker客户端创建选项相同的格式。

```yaml
devices:
  - "/dev/ttyUSB0:/dev/ttyUSB0"
```

> **注意**：当使用（版本3）Compose文件[在群集模式下部署堆栈](https://docs.docker.com/engine/reference/commandline/stack_deploy/)时，[将](https://docs.docker.com/engine/reference/commandline/stack_deploy/)忽略此选项 。

## depends_on 依赖

服务之间的快速依赖关系，服务依赖关系导致以下行为：

- `docker-compose up`以**依赖的顺序**启动服务。在下面的例子中，`db`和`redis`在之前启动`web`。
- `docker-compose up SERVICE`自动包含`SERVICE`依赖关系。在下面的例子中，`docker-compose up web`创建并启动`db`和`redis`。

简单的例子：

```yaml
version: '3'
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

> 使用`depends_on`注意事项：
>
> - 在启动`web`之前，`depends_on`不会等待`db`和`redis`“准备就绪” - 只有在它们启动之前。如果需要等待服务准备就绪，请参阅[控制启动顺序](https://docs.docker.com/compose/startup-order/)以了解有关此问题的更多信息以及解决此问题的策略。
> - 第3版不再支持`condition`的形式`depends_on`。
> - 使用版本3编排文件[在群集模式下部署堆栈](https://docs.docker.com/engine/reference/commandline/stack_deploy/)`depends_on`时，该选项将被忽略 。

## dns

自定义DNS服务器。可以是单个值或列表。

```yaml
dns: 8.8.8.8
dns:
  - 8.8.8.8
  - 9.9.9.9
```

## dns_search

自定义DNS搜索域。可以是单个值或列表。

```yaml
dns_search: example.com
dns_search:
  - dc1.example.com
  - dc2.example.com
```

## tmpfs 临时文件系统

在容器中装入一个**临时文件系统**。可以是单个值或列表。

```yaml
tmpfs: /run
tmpfs:
  - /run
  - /tmp
```

> **注意**：[在](https://docs.docker.com/engine/reference/commandline/stack_deploy/) 使用（版本3-3.5）compose文件的[群集模式下部署堆栈时，](https://docs.docker.com/engine/reference/commandline/stack_deploy/)忽略此选项 。

在**容器中**装入一个临时文件系统。`size`参数以字节为单位指定`tmpfs`挂载的大小，无限制默认。

```yaml
 - type: tmpfs
     target: /app
     tmpfs:
       size: 1000
```

## entrypoint 入口点

覆盖默认入口点。

```yaml
entrypoint: /code/entrypoint.sh
```

入口点也可以是一个列表，方式类似于 [dockerfile](https://docs.docker.com/engine/reference/builder/#entrypoint)：

```yaml
entrypoint:
    - php
    - -d
    - zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20100525/xdebug.so
    - -d
    - memory_limit=-1
    - vendor/bin/phpunit
```

> **注意**：`entrypoint`使用 Dockerfile 的`ENTRYPOINT`指令设置**覆盖服务映像上**设置的任何默认入口点，并**清除映像**上的任何默认命令。这意味着如果Dockerfile中有 `CMD` 指令，则会**忽略**它。

## env_file 环境变量文件

从文件添加环境变量，可以是单个值或列表。<br/>如果已经指定了Compose文件`docker-compose -f FILE`，则路径 `env_file`相对于compose文件所在的目录。

`environment`部分中 声明的环境变量**会覆盖**`env_file` 的值， 即使这些值为空或未定义，也是如此。

```yaml
env_file: .env

env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```

compose 配置期望`env`文件中的每行都是`VAR=VAL`格式。以开头的行`#`被视为注释并被忽略，空行也被忽略。

```properties
# Set Rails/Rack environment
RACK_ENV=development
```

> **注意**：如果服务指定了[构建](https://docs.docker.com/compose/compose-file/#build)选项，则在构建期间，环境文件中定义的变量*不会*自动显示。使用[args](https://docs.docker.com/compose/compose-file/#args)子选项`build`来定义构建时环境变量。

**该值按`VAL`原样使用，根本不作任何修改**。例如，如果值被引号包围（如通常是shell变量的情况），则引用将包含在传递给Compose的值中。

请记住，**列表中文件的顺序在确定分配给多次显示的变量的值时非常重要**。对于文件`a.env`中指定的相同变量并在文件`b.env`中分配了不同的值，如果`b.env`列在下面（之后），则表示`b.env`中的值。例如，在`docker_compose.yml`中给出以下声明：

```yaml
services:
  some-service:
    env_file:
      - a.env
      - b.env
```

以下文件：

```yaml
# a.env
VAR=1
```

和

```yaml
# b.env
VAR=hello
```

`$VAR`是`hello`。

## environment 环境变量

添加环境变量，可以使用**数组或字典**。任何boolean值：`True、False、Yes`，需要用**引号括起来以确保它们不被YML解析器转换为True或False**。

只有一个环境变量会在计算机上运行时解析为它们的值，这对于主机特定的值可能会有所帮助。

```yaml
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:

environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```

> **注意**：如果服务指定了[build](https://docs.docker.com/compose/compose-file/#build)选项，`environment`则在构建过程中定义的变量**不会**自动显示。使用[args](https://docs.docker.com/compose/compose-file/#args)子选项`build`来定义构建时环境变量。

## expose 暴露

**公开端口**而不将它们发布到主机，它们只能**被链接服务访问**，**只能指定内部端口**。

```yaml
expose:
 - "3000"
 - "8000"
```

## external_links 外部链接

链接到在此之外`docker-compose.yml`甚至在Compose之外的容器，特别是对于提供共享或公共服务的容器。 在指定容器名称和链接别名（）时`external_links`遵循与旧选项类似的语义。`links``CONTAINER:ALIAS`

```
external_links:
 - redis_1
 - project_db_1:mysql
 - project_db_1:postgresql
```

> **笔记：**
>
> 如果您使用的是[第2版或更高版本的文件格式](https://docs.docker.com/compose/compose-file/compose-versioning/#version-2)，则外部创建的容器必须至少连接到与链接到它们的服务相同的网络。[链接](https://docs.docker.com/compose/compose-file/compose-file-v2#links)是遗留选项。我们建议使用[网络](https://docs.docker.com/compose/compose-file/#networks)。
>
> 使用（版本3）Compose文件[在群集模式下部署堆栈](https://docs.docker.com/engine/reference/commandline/stack_deploy/)时忽略此选项。