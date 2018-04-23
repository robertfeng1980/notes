# docker 上开发应用

[TOC]

# 概述

## 在Docker上开发新的应用程序

介绍以下内容的应用：

- 学习[从Dockerfile构建图像](https://docs.docker.com/get-started/part2/)
- 使用[多阶段构建](https://docs.docker.com/engine/userguide/eng-image/multistage-build/)来保持图像精简
- 使用[卷](https://docs.docker.com/engine/admin/volumes/volumes/)管理应用程序数据并[绑定挂载](https://docs.docker.com/engine/admin/volumes/bind-mounts/)
- [应用扩展](https://docs.docker.com/get-started/part3/)为群集服务
- 使用compose文件[定义应用程序堆栈](https://docs.docker.com/get-started/part5/)
- 一般应用程序开发最佳实践

## 了解有关Docker语言特定的应用程序开发

- [Docker for Java开发人员](https://github.com/docker/labs/tree/master/developer-tools/java/)实验室
- [将一个node.js应用程序移植到Docker](https://github.com/docker/labs/tree/master/developer-tools/nodejs/porting)

## 使用SDK或API进行高级开发

通过编写`Dockerfiles`或compose文件以及使用`Docker CLI`来开发应用程序，则可以通过使用`Docker Engine for Go`或`Python`或直接使用`HTTP API`将其提升到下一个级别。



# docker 开发事项

## 保持镜像小体积


在启动容器或服务时，**小镜像可以更快速地从网络拉向内存并加载到内存中**。有几条经验法则可以保持较小的镜像尺寸：

- 从适当的**基础镜像开始**。例如，如果需要`JDK`，请考虑将镜像使用官方`openjdk`作为基础镜像，而不是从通用`ubuntu`映像开始，并`openjdk`作为`Dockerfile`的一部分进行安装。

- [使用多阶段构建](https://docs.docker.com/engine/userguide/eng-image/multistage-build/)。例如，可以使用该`maven`镜像构建Java应用程序，然后重置为`tomcat`镜像并将`Java`构件复制到正确的位置以部署应用程序，所有这些都位于相同的`Dockerfile`中。这意味着最终镜像不包含构建所引入的所有库和依赖项，但仅包含运行它们所需的组件和环境。

  - 如果需要使用不包含多级构建的Docker版本，请尽量减少`RUN` `Dockerfile` 中单独命令的数量，以减少镜像中的图层数量。**可以通过将多个命令整合到`RUN`一行中并使用shell的机制将它们组合在一起来完成此操作**。考虑以下两个片段，第一个在镜像中创建两个图层，而第二个创建一个图层。

    ```
    RUN apt-get -y update
    RUN apt-get install -y python
    ```

    ```
    RUN apt-get -y update && apt-get install -y python
    ```

- 如果**有多个共同的镜像**，请考虑使用**共享组件**创建自己的 [基本镜像](https://docs.docker.com/engine/userguide/eng-image/baseimages/)，并在其上创建独特的镜像。Docker只需要加载一次**通用层**，然后**缓存**。这意味着衍生镜像更有效地使用Docker主机上的内存并更快地加载。

- 要保持生产镜像精简但允许调试，请考虑使用**生产镜像作为调试镜像的基本镜像**。可以在生产镜像上添加其他测试或调试工具。

- 在构建镜像时，始终使用有用的**标签对其进行标记**，这些标签将编码版本信息，预期目标（`prod`或`test`），稳定性或在不同环境中部署应用程序时有用的其他信息编码。不要依赖自动创建的`latest`标签。

## 持久保存应用数据

- **避免**使用[存储驱动程序](https://docs.docker.com/engine/userguide/storagedriver/)将应用程序数据存储在容器的可写层中 。这会增加容器的大小，并且从`I/O`角度看，效率会低于使用卷或绑定安装的效率。
- 相反，**使用卷存储数据**。
- 在开发过程中适合使用[绑定挂载的](https://docs.docker.com/engine/admin/volumes/bind-mounts/)一种情况，可能需要**挂载源目录**或刚刚构建到容器中的**二进制文件**。对于生产改用**卷**，将其安装到与在开发过程中**安装绑定挂载程序相同的位置**。
- 对于生产而言，**使用[secrets](https://docs.docker.com/engine/swarm/secrets/)来存储服务使用的敏感私密的应用程序数据**，并将[配置](https://docs.docker.com/engine/swarm/configs/) 用于非敏感数据（如配置文件）。如果当前使用独立容器，请考虑迁移以使用单一副本服务，以便可以利用这些仅限于服务的功能。

## 使用swarm服务

- 在可能的情况下，**使用swarm服务进行扩展的能力**来设计应用程序。
- 即使只需运行应用程序的**单个实例**，swarm服务也可以提供**比独立容器更多的优势**。服务的配置是声明式的，Docker一直在努力使期望的和实际的状态保持同步。
- 网络和卷可以与swarm服务连接和断开连接，并且Docker可以以**不中断的方式重新部署各个服务容器**。需要手动停止，移除和**重新创建**独立容器以适应配置更改。
- 一些功能（如存储 [secrets](https://docs.docker.com/engine/swarm/secrets/)和[config](https://docs.docker.com/engine/swarm/configs/)功能）仅适用于服务而不是独立容器。这些功能允许保持镜像尽可能通用，并避免将**敏感私密数据存储在Docker镜像或容器本身**内。
- 让我们使用`docker stack deploy`为处理任何镜像，而不是使用 `docker pull`。这样，**您的部署不会尝试从已关闭或故障的节点拉取。而且，当新节点添加到群中时，会自动拉取镜像**。

**在集群服务的节点之间共享数据存在限制**。如果您将[Docker用于AWS](https://docs.docker.com/docker-for-aws/persistent-data-volumes/)或 [Docker for Azure](https://docs.docker.com/develop/docker-for-azure/persistent-data-volumes/)，则可以使用`Cloudstor`插件在群集服务节点之间共享数据。您还可以将应用程序数据写入支持同时更新的单独数据库。

## 使用`CI/CD`进行测试和部署

- 当检查对源代码管理的更改或创建拉取请求时，请使用 [Docker Cloud](https://docs.docker.com/docker-cloud/builds/automated-build/)或其他`CI / CD`管道自动构建并标记Docker映像并对其进行测试。`Docker Cloud`也可以将测试过的应用直接部署到生产环境中。
- 通过要求开发，测试和安全团队在将镜像部署到生产环境中之前对其进行**签名**，可以更进一步了解[Docker EE](https://docs.docker.com/ee/)。通过这种方式，可以确保在将映像部署到生产环境之前，它已经通过了开发，质量和安全团队的测试和签名。

## 开发和生产环境的差异

| 开发                                       | 生产                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| 使用绑定挂载使您的容器可以访问您的源代码。 | 使用卷来存储容器数据。                                       |
| 使用Docker for Mac或Docker for Windows。   | 如果可能的话，使用Docker EE，通过[用户映射](https://docs.docker.com/engine/security/userns-remap/)将Docker进程与主进程隔离开来。 |
| 不要担心时间不同步。                       | 始终在Docker主机上和每个容器进程中运行`NTP`客户端，并将它们全部同步到同一个`NTP`服务器。如果使用swarm服务，还要确保每个Docker节点将其时钟与容器同步到同一时间源。 |

---
下面将介绍`dockerfile`的编写和应用，以及如何创建一个基础镜像。随后该如何多阶段构建镜像来降低镜像尺寸，加快镜像加载速度。最终怎么管理我们的镜像程序。


# 编写`Dockerfile`

Docker可以通过从`Dockerfile`文本文件中读取指令自动构建镜像 ，以便构建给定镜像。`Dockerfile`是一个文本文档，其中包含用户可以在命令行上调用以组装图像的所有命令。`Dockerfile`遵循特定的格式并使用特定的指令集。可以在[Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)页面上了解基础知识 。

本文档介绍了由`Docker Inc.`和Docker社区推荐的用于**构建高效镜像的最佳实践和方法**。要查看许多实践和建议，请查看Dockerfile for [buildpack-deps](https://github.com/docker-library/buildpack-deps/blob/master/jessie/Dockerfile)。

> **注意**：有关此处提及的任何Dockerfile命令的更详细说明，请访问[Dockerfile参考](https://docs.docker.com/engine/reference/builder/)页面。



## 准则和建议


### 容器应该是敏捷的

由`dockerfile`定义的镜像生成的容器应尽可能短暂。通过“短暂的”，我们的意思是它可以被停止并被销毁，并且一个新的建立和配置到位，并具有绝对最小的设置和配置。您可能需要查看**12因子**应用程序方法的[流程](https://12factor.net/processes)部分，以了解以这种无状态方式运行容器的动机。

### 构建上下文

当发出一个`docker build`命令时，当前的工作目录被称为*构建上下文*。默认情况下，假定`Dockerfile`位于此处，但可以使用文件选项（`-f`）指定不同的位置。无论`Dockerfile`文件实际在哪里，当前目录中的所有文件和目录的递归内容都将作为构建上下文发送到Docker守护进程。

> **构建上下文示例**
>
> 为构建上下文创建一个目录，并`cd`进入文件目录。写入一个名为“hello”文本文件，内容是`hello`，并创建一个Dockerfile运行`cat`查看文件内容。从构建上下文（`.`）中构建图像：
>
> ```shell
> $ mkdir sample && cd sample
>
> $ echo "hello" > hello
> $ echo -e "FROM busybox\nCOPY /hello /\nRUN cat /hello" > Dockerfile
> $ docker build -t hello_app:v1 .
> ```
>
> 现在移动`Dockerfile`和`hello`到不同的目录，并建立了图像的第二个版本（不依赖于缓存中的最后一个版本）。使用`-f`指向Dockerfile并指定构建上下文的目录：
>
> ```shell
> $ mkdir -p dockerfiles codes
> $ mv Dockerfile dockerfiles && mv hello codes
> $ docker build --no-cache -t hello_app:v2 -f dockerfiles/Dockerfile codes
> ```

包含不需要构建图像的文件会**导致更大的构建上下文和更大的图像大小**。这可以**增加构建时间，拉取和推送图像的时间以及容器的运行时间大小**。要查看构建环境有多大，请在构建系统时查找这样的消息`Dockerfile`：

```shell
Sending build context to Docker daemon  2.607kB
```

### 使用.dockerignore文件

要**排除与构建无关的文件，而不重构源代码库**，请使用`.dockerignore`文件。该文件支持与`.gitignore`文件类似的排除模式。创建一个有关的信息，请参阅[.dockerignore文件](https://docs.docker.com/engine/reference/builder/#dockerignore-file)。除了使用`.dockerignore`文件外，请查看以下关于[多阶段构建的信息](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#use-multi-stage-builds)。

### 使用多阶段构建

如果使用Docker 17.05或更高版本，则可以使用 [多阶段构建](https://docs.docker.com/develop/develop-images/multistage-build/)来大幅**缩减最终镜像的大小**，而无需在构建过程中跳过环节以减少中间层数量或删除中间文件。

图像仅由最后阶段构建，**大多数时间都可以同时构建缓存和图像层**。

构建阶段可能包含多个图层，从较低频率更改为较常更改频率，例如：

- 安装构建应用程序所需的工具
- 安装或更新库依赖关系
- 生成你的应用程序

应用程序的Dockerfile可能如下所示：

```dockerfile
FROM golang:1.9.2-alpine3.6 AS build

# 安装构建项目所需的工具
# 需要运行 `docker build --no-cache .` 更新这些依赖关系
RUN apk add --no-cache git
RUN go get github.com/golang/dep/cmd/dep

# Gopkg.toml and Gopkg.lock 列出项目依赖关系
# 这些图层仅在更新gopkg文件时重新构建
COPY Gopkg.lock Gopkg.toml /go/src/project/
WORKDIR /go/src/project/
# 安装库依赖项
RUN dep ensure -vendor-only

# 复制所有项目并构建它
# 当项目目录中的文件发生更改时，该层将被重建
COPY . /go/src/project/
RUN go build -o /bin/project

# 最终这导致了单层图像
FROM scratch
COPY --from=build /bin/project /bin/project
ENTRYPOINT ["/bin/project"]
CMD ["--help"]
```

### 避免安装不必要的软件包

为了减少复杂性、依赖性、文件大小和构建时间，应该避免安装额外的或不必要的软件包，只是因为它们可能“很好”。例如，不需要在数据库镜像中包含文本编辑器或第三方编辑工具。

### 每个容器应该只有一个问题

应用程序**解耦为多个容器使得横向扩展和重用容器**变得更容易。例如，Web应用程序编排服务可能由三个独立的容器组成，每个容器都有其独立的镜像，以分离的方式管理Web应用程序、数据库和内存缓存。

可能听说应该有“每个容器一个进程”。虽然这个口头禅的意图很好，但并不一定每个容器只有一个操作系统进程。除了现在可以[使用init进程创建](https://docs.docker.com/engine/reference/run/#specifying-an-init-process)容器之外，一些程序可能会自行产生其他进程。例如，[Celery](http://www.celeryproject.org/)可以派生多个工作进程，或者[Apache](https://httpd.apache.org/)可能会为每个请求创建一个进程。虽然“每个容器一个进程”往往是一个很好的经验法则，但它并不是一条硬性规定。尽你最大的努力使容器**保持干净和模块化**。

**如果容器相互依赖，则可以使用[Docker容器网络](https://docs.docker.com/engine/userguide/networking/) 来确保这些容器可以通信**。

### 最小化镜像图层数

在Docker 17.05之前，甚至更多的是在Docker 1.10之前，重要的是**尽量减少镜像中的图层数量**。以下改进缓解了这种需求：

- 在docker 1.10和更高，只有`RUN`，`COPY`和`ADD`指令创建图层。其他动作指令会创建临时中间镜像，不再直接增加构建的大小。
- Docker 17.05和更高版本添加了对[多阶段构建的](https://docs.docker.com/develop/develop-images/multistage-build/)支持 ，这允许只将需要的**构件复制到最终镜像中**。允许在**中间构建阶段包含工具和调试信息，而不增加最终图像的大小**。

### 对多行参数进行排序

通过**按字母数字排序多行参数**来简化后面的更改。这有助于**避免软件包重复**并使列表更容易更新。这也使得PR更**容易阅读**和评论。在**反斜杠（`\`）之前添加一个空格**也有帮助。

以下是[`buildpack-deps`图片中](https://github.com/docker-library/buildpack-deps)的一个示例：

```dockerfile
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion
```

### 建立缓存

在构建镜像的过程中，`Dockerfile`按照指定顺序执行每个指令。在检查每条指令时，**Docker会在其缓存中查找可重用的现有镜像**，而不是创建新的（重复）镜像。如果根本不想使用缓存，则可以使用命令中的`--no-cache=true`选项进行`docker build`。

但是，如果确实让Docker使用了它的缓存，那么了解它何时可以并且无法找到匹配的镜像是非常重要的。Docker遵循的基本规则如下：

- 从已经在缓存中的父镜像开始，**将下一条指令与从该基本镜像派生的所有子镜像进行比较**，以查看是否使用完全相同的指令构建。否则缓存失效。
- 在大多数情况下，简单地比较`Dockerfile`一个**子镜像中的指令**就足够了。但是，某些指令动作需要更多的**检查和解释**。
- 对于`ADD`和`COPY`，将检查镜像中文件的内容，并为每个文件计算校验`checksum `。这些校验`checksum `不考虑文件的最后修改时间和最后访问时间。在缓存查找过程中，**将校验`checksum `与现有镜像中的校验`checksum `进行比较。如果文件中有任何内容已更改，如内容和元数据，则缓存将失效**。
- 除了`ADD`和`COPY`命令之外，缓存检查**不会查看容器中**的文件以确定缓存匹配。例如，在处理`RUN apt-get -y update`命令时，**不检查容器中更新的文件以确定是否存在缓存命中**。在这种情况下，只是**命令字符串本身**用于查找匹配。

一旦缓存失效，所有后续的`Dockerfile`命令将**生成新图像**，并且不使用缓存。

## 用法


当编辑好`dockerfile`后，[`docker build`](https://docs.docker.com/engine/reference/commandline/build/)命令从 `Dockerfile`和*上下文*构建镜像。构建的上下文是指定位置`PATH`或文件集中的文件`URL`。`PATH`是你本地文件系统上的一个目录。`URL`是一个Git存储库位置。

上下文是递归处理的。因此， `PATH`包含任何子目录，`URL`包含存储库及其子模块。示例显示了一个使用当前目录作为上下文的构建命令：

```shell
$ docker build .
Sending build context to Docker daemon  6.51 MB
...
```

**构建由Docker守护进程运行，而不是通过CLI运行**。构建过程的第一件事是将整个上下文（递归地）发送到守护进程。在大多数情况下，**最好以空目录作为上下文开始，并将`Dockerfile`保存在该目录中**。仅添加构建Dockerfile所需的文件。

> **警告**：不要用你的根目录下，`/`作为`PATH`因为它会导致生成到硬盘驱动器的全部内容传输到docker守护进程。

要在构建上下文中使用文件`Dockerfile`引用指令中指定的文件，例如`COPY`指令。要增加构建的性能，请通过将`.dockerignore`文件添加到上下文目录来排除文件和目录。有关如何[创建`.dockerignore` 文件的信息，](https://docs.docker.com/engine/reference/builder/#dockerignore-file)请参阅此页面上的文档。

传统上，它`Dockerfile`位于上下文的根部。可以使用该`-f`选项`docker build`指向文件系统中任何位置的Dockerfile。

```shell
$ docker build -f /path/to/a/Dockerfile .
```

如果构建成功，可以指定一个仓库和标签来保存新镜像：

```shell
$ docker build -t shykes/myapp .
$ docker build -tag shykes/myapp:last .
```

要在构建之后将镜像标记到多个仓库中，请在`-t`运行此`build`命令时添加多个参数：

```shell
$ docker build -t shykes/myapp:1.0.2 -t shykes/myapp:latest .
```

在Docker守护进程运行`Dockerfile`其中的指令之前，它执行初步的验证`Dockerfile`并在语法不正确时返回错误：

```shell
$ docker build -t test/myapp .
Sending build context to Docker daemon 2.048 kB
Error response from daemon: Unknown instruction: RUNCMD
```

Docker守护进程会`Dockerfile`逐个运行指令，在必要时将每条指令的结果提交给新映像，最终输出新映像的标识。Docker守护进程将自动清理您发送的上下文。

请注意，每条指令都是独立运行的，并会创建一个新图像 - 所以`RUN cd /tmp`不会对下一条指令产生任何影响。

只要有可能，Docker将重新使用中间映像（缓存），以`docker build`显着加速该过程。这由`Using cache`控制台输出中的消息指示。（有关更多信息，请参阅最佳做法指南中的 “ [构建缓存”部分](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#build-cache)`Dockerfile`）：

```
$ docker build -t svendowideit/ambassador .
Sending build context to Docker daemon 15.36 kB
Step 1/4 : FROM alpine:3.2
 ---> 31f630c65071
Step 2/4 : MAINTAINER SvenDowideit@home.org.au
 ---> Using cache
 ---> 2a1c91448f5f
Step 3/4 : RUN apk update &&      apk add socat &&        rm -r /var/cache/
 ---> Using cache
 ---> 21ed6e7fbb73
Step 4/4 : CMD env | grep _TCP= | (sed 's/.*_PORT_\([0-9]*\)_TCP=tcp:\/\/\(.*\):\(.*\)/socat -t 100000000 TCP4-LISTEN:\1,fork,reuseaddr TCP4:\2:\3 \&/' && echo wait) | sh
 ---> Using cache
 ---> 7ea8aef582cc
Successfully built 7ea8aef582cc
```

构建缓存仅用于具有本地父链的图像。这意味着这些图像是由以前的版本创建的，或者是整个图像链加载的`docker load`。如果您希望使用特定映像的构建缓存，则可以使用`--cache-from`选项指定它。指定的图像`--cache-from`不需要有父链，可以从其他注册表中提取。

当你完成你的构建时，你已经准备好考虑[*将存储库推送到它的注册表*](https://docs.docker.com/engine/tutorials/dockerrepos/#/contributing-to-docker-hub)。

## Dockerfile 指令




### FROM

[用于FROM指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#from)

只要有可能，请使用当前的官方存储库作为基础镜像。我们推荐使用[Alpine图像](https://hub.docker.com/_/alpine/) 因为它的控制非常严格，并且保持最低限度（目前低于`5 MB`），同时仍然是完整的发行版。

### LABEL

[了解对象标签](https://docs.docker.com/config/labels-custom-metadata/)

可以将标签添加到镜像中，以帮助按项目组织镜像、记录许可信息、帮助自动化或出于其他原因。添加`LABEL`以一个或多个键值对开头的行。以下示例显示了不同的格式。

> **注意**：如果你的字符串包含空格，那么它必须被引号包围或者**空格必须被转义**。如果你的字符串包含内部引号字符（`"`），也可以将它们转义。

```dockerfile
# Set one or more individual labels
LABEL com.example.version="0.0.1-beta"
LABEL vendor="ACME Incorporated"
LABEL com.example.release-date="2015-02-12"
LABEL com.example.version.is-production=""
```

一个`LABEL`以有多个标签。在`Docker 1.10`之前，建议将所有标签合并为一条`LABEL`指令，以防止创建额外的图层。现在这不再是必要的，但组合标签仍然受支持。

```dockerfile
# Set multiple labels on one line
LABEL com.example.version="0.0.1-beta" com.example.release-date="2015-02-12"
```

以上内容也可以写成：

```dockerfile
# Set multiple labels at once, using line-continuation characters to break long lines
LABEL vendor=ACME\ Incorporated \
      com.example.is-beta= \
      com.example.is-production="" \
      com.example.version="0.0.1-beta" \
      com.example.release-date="2015-02-12"
```

有关可接受的标签键和值的信息，请参阅[了解对象标签](https://docs.docker.com/config/labels-custom-metadata/)。有关查询标签的信息，请参阅[管理对象上的标签中](https://docs.docker.com/config/labels-custom-metadata/#managing-labels-on-objects)与过滤相关的项目。另请参阅 Dockerfile参考中的[LABEL](https://docs.docker.com/engine/reference/builder/#label)。

### RUN

[运行指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#run)

为了使`Dockerfile`可读性更强，易于理解并且可维护，可以`RUN`在**多行用反斜杠分隔长分或复杂语句**

#### apt-get

---

可能最常见的用例是应用程序`RUN apt-get`。该 `RUN apt-get`命令，因为它安装了软件包，有几个需要注意的问题。

应该避免`RUN apt-get upgrade`或者`dist-upgrade`，尽可能多的来自父镜像的“基本”软件包无法在[非特权容器](https://docs.docker.com/engine/reference/run/#security-configuration)内升级 。如果父镜像中包含的软件包已过时，则应联系其维护人员。如果知道有特定的软件包`foo`需要更新，请使用`apt-get install -y foo`自动更新。

在相同的指令中结合`RUN apt-get update`使用。例如：`RUN apt-get install `

```dockerfile
    RUN apt-get update && apt-get install -y \
        package-bar \
        package-baz \
        package-foo
```

`apt-get update`在`RUN`声明中**单独使用会导致缓存**问题，并导致后续`apt-get install`指令失败。例如，假设你有一个Dockerfile：

```dockerfile
    FROM ubuntu:14.04
    RUN apt-get update
    RUN apt-get install -y curl
```

构建完镜像后，所有图层都在Docker缓存中。假设稍后通过`apt-get install`添加额外的软件包进行修改：

```dockerfile
    FROM ubuntu:14.04
    RUN apt-get update
    RUN apt-get install -y curl nginx
```

Docker将初始和修改后的指令视为相同，并重用先前步骤中的缓存。结果`apt-get update`是*不*执行，因为构建使用缓存版本。由于`apt-get update`未运行，构建可能会获得`curl`和`nginx` 包的过时版本。

**使用 `RUN apt-get update && apt-get install -y`可确保的Dockerfile安装最新的软件包版本**，而无需进一步编码或手动干预。这种技术被称为**缓存破坏**。也可以通过指定软件包版本来实现**缓存清除**。这被称为**版本固定**，例如：

```dockerfile
    RUN apt-get update && apt-get install -y \
        package-bar \
        package-baz \
        package-foo=1.3.*
```

**版本固定强制构建查找特定版本，而不管缓存中的内容**。该技术还可以减少由于所需软件包的意外更改而导致的故障。

以下是一个格式良好的`RUN`动作指令描述，可以说明所有`apt-get` 建议。

```dockerfile
RUN apt-get update && apt-get install -y \
    aufs-tools \
    automake \
    build-essential \
    curl \
    dpkg-sig \
    libcap-dev \
    libsqlite3-dev \
    mercurial \
    reprepro \
    ruby1.9.1 \
    ruby1.9.1-dev \
    s3cmd=1.1.* \
 && rm -rf /var/lib/apt/lists/*
```

该`s3cmd`指令指定的版本`1.1.*`。如果以前使用旧版本的镜像，指定新映像会导致缓存失败`apt-get update`并确保安装新版本。在**每行上列出软件包还可以防止软件包重复**中的错误。

另外，通过删除`/var/lib/apt/lists` 缩小镜像大小来清理`apt cache`时，因为`apt cache`并不存储在一个图层中。由于 `RUN`语句开头`apt-get update`，所以包缓存总是在`apt-get install`之前刷新。

> **注意**：官方的Debian和Ubuntu镜像会[自动运行`apt-get clean`](https://github.com/moby/moby/blob/03e2923e42446dbb830c654d0eec323a0b4ef02a/contrib/mkimage/debootstrap#L82-L105)，所以不需要显式调用。



#### pipe

某些`RUN `命令取决于将一个命令的输出传送到另一个命令的能力，使用管道字符`（|）`分隔，如下例所示：

```dockerfile
RUN wget -O - https://some.site | wc -l > /number
```

Docker使用`/bin/sh -c`解释器执行这些命令，解释器只评估管道中最后一个操作的退出代码以确定是否成功。在上面的示例中，只要`wc -l`命令成功，即使`wget`命令失败，该构建步骤也会成功并生成新图像。

如果由于管道中任何阶段的错误而导致命令失败，请预先`set -o pipefail &&`确保意外错误可防止构建无意中成功。例如：

```dockerfile
RUN set -o pipefail && wget -O - https://some.site | wc -l > /number
```

> **注意**：并非所有`shell`都支持该`-o pipefail`选项。在这种情况下（例如`dash`shell，这是基于`Debian`的镜像上的默认`shell`），考虑使用`run`的`exec`形式来显式选择一个支持`pipefail`选项的`shell`。例如：

```dockerfile
RUN ["/bin/bash", "-c", "set -o pipefail && wget -O - https://some.site | wc -l > /number"]
```

### CMD

[CMD指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#cmd)

该`CMD`指令应该用于运行镜像像中包含的软件以及任何参数。`CMD`应该几乎总是以形式使用`CMD [“executable”, “param1”, “param2”…]`。因此，如果镜像是用于服务的，比如`Apache`和`Rails`，那么你可以运行类似的东西 `CMD ["apache2","-DFOREGROUND"]`。这种形式的指令被推荐用于任何基于服务的图像。

在其他大多数情况下，`CMD`应该给出一个交互式`shell`，比如`bash`，`python`和`perl`。例如，`CMD ["perl", "-de0"]`，`CMD ["python"]`，或 `CMD [“php”, “-a”]`。使用这种形式意味着当你执行类似的东西时`docker run -it python`，你会被放入一个可用的`shell`中，随时可以使用。 `CMD`应该很少的方式使用`CMD [“param”, “param”]`会同[`ENTRYPOINT`](https://docs.docker.com/engine/reference/builder/#entrypoint)，除非你和你预期的用户已经非常熟悉如何`ENTRYPOINT` 工作的。

### EXPOSE

[EXPOSE指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#expose)

该`EXPOSE`指令指示容器侦听连接的端口。因此，应该为应用程序使用通用的传统端口。例如，包含`Apache Web`服务器的镜像将使用`EXPOSE 80`，而包含`MongoDB`的图像将使用`EXPOSE 27017`等等。

对于外部访问，用户可以执行`docker run`一个标志，指示如何将指定端口映射到他们选择的端口。对于容器链接，Docker为从收件人容器返回到源路径（即，`MYSQL_PORT_3306_TCP`）提供环境变量。

### ENV

[Dock指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#env)

为了使新软件更容易运行，可以使用`ENV`更新容器安装软件的 `PATH`环境变量。例如，`ENV PATH /usr/local/nginx/bin:$PATH`确保`CMD [“nginx”]` 正常工作。

`ENV`指令对于提供特定于你希望容器化的服务所需的环境变量也很有用，例如`Postgres's`  `PGDATA`。

最后，`ENV`还可以用于设置常用版本号，以便版本变更中更容易维护，如以下示例所示：

```dockerfile
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```

类似于程序中的常量变量（与硬编码值不同），这种方法可以更改单个`ENV`指令，自动地处理容器中软件的版本。

**每个`ENV`行创建一个新的图层**，就像`RUN`命令一样。这意味着，即使**在未来的图层中取消设置环境变量，它仍然会保留在此图层中**，并且其值可能会被丢弃。你可以通过像下面这样创建一个Dockerfile来测试它，然后构建它。

```dockerfile
FROM alpine
ENV ADMIN_USER="mark"
RUN echo $ADMIN_USER > ./mark
RUN unset ADMIN_USER
CMD sh
```

```shell
$ docker run --rm -it test sh echo $ADMIN_USER

mark
```

为了防止出现这种情况并真正取消设置环境变量，请使用`RUN`带有shell命令的命令，**在单个图层中设置，使用和取消设置所有变量**。你可以用`;`或分隔你的命令`&&`。如果使用第二种方法，并且其中一个命令失败，那么`docker build`也会失败，这通常是一个好主意。**使用`\`作为Linux的Dockerfile续行符提高可读性**。你也可以**把所有的命令放到一个shell脚本中，让`RUN`命令运行这个shell脚本**。

```
FROM alpine
RUN export ADMIN_USER="mark" \
    && echo $ADMIN_USER > ./mark \
    && unset ADMIN_USER
CMD sh

$ docker run --rm -it test sh echo $ADMIN_USER
```

### ADD & COPY

- [Dock指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#add)
- [适用于COPY指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#copy)

虽然`ADD`和`COPY`在功能上类似，**一般说来`COPY` 是优选的，这是因为它比`ADD`更透明**。`COPY`仅**支持将本地文件复制到容器中**，同时`ADD`还具有一些不明显的功能（如仅限本地的`tar`提取和远程`URL`支持）。因此，**最好的用`ADD`是将本地tar文件自动提取到图像中**，如`ADD rootfs.tar.xz /`。

如果有多个`Dockerfile`步骤使用来自上下文的不同文件，那么`COPY`它们是单独的，而不是一次使用全部。这可确保**每个步骤的构建缓存**仅在特定所需文件发生更改时才会失效（强制重新运行该步骤）。

例如：

```dockerfile
COPY requirements.txt /tmp/
RUN pip install --requirement /tmp/requirements.txt
COPY . /tmp/
```

导致`RUN`步骤中缓存失效的次数少于放弃 `COPY . /tmp/`之前的缓存失效次数。

由于镜像大小很重要，因此`ADD`强烈建议**不要使用从远程`URL`获取软件包**; 你应该使用`curl`或`wget`代替。这样，可以**删除解压缩后不再需要的文件**，并且不必在图像中添加其他图层。例如，应该避免执行以下操作：

```dockerfile
ADD http://example.com/big.tar.xz /usr/src/things/
RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
RUN make -C /usr/src/things all
```

而是，做一些事情：

```dockerfile
RUN mkdir -p /usr/src/things \
    && curl -SL http://example.com/big.tar.xz \
    | tar -xJC /usr/src/things \
    && make -C /usr/src/things all
```

对于其他不需要`ADD`自动解压功能的项目（文件，目录），应该始终使用`COPY`。

### ENTRYPOINT

[用于ENTRYPOINT指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#entrypoint)

`ENTRYPOINT`的最佳用途是设置镜像的主要命令，允许该镜像像命令一样运行（然后`CMD`用作默认标志）。

我们从命令行工具的镜像示例`s3cmd`开始：

```dockerfile
ENTRYPOINT ["s3cmd"]
CMD ["--help"]
```

现在可以像这样运行镜像来显示命令的帮助：

```shell
$ docker run s3cmd
```

或者使用正确的参数来执行命令：

```shell
$ docker run s3cmd ls s3://mybucket
```

这很有用，因为如上面的命令所示，镜像名称可以作为对二进制文件的引用的双倍值。

该`ENTRYPOINT`指令还可以与辅助脚本结合使用，使其能够以类似于上述命令的方式工作，即使启动该工具可能需要多个步骤。

例如，[Postgres官方图像](https://hub.docker.com/_/postgres/) 使用以下脚本作为它的`ENTRYPOINT`：

```shell
#!/bin/bash
set -e

if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"

    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi

    exec gosu postgres "$@"
fi

exec "$@"
```

> **注**：此脚本使用[的`exec`bash命令](http://wiki.bash-hackers.org/commands/builtin/exec) ，使最终运行的应用程序成为容器的`PID` 这允许应用程序接收发送到容器任何Unix信号。查看[`ENTRYPOINT`](https://docs.docker.com/engine/reference/builder/#entrypoint) 帮助以获取更多详细信息。

脚本被复制到容器中并通过`ENTRYPOINT`容器启动运行：

```dockerfile
COPY ./docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["postgres"]
```

这个脚本允许用户以几种方式与Postgres交互。

它可以简单地启动Postgres：

```shell
$ docker run postgres
```

或者，它可以用来运行Postgres并将参数传递给服务器：

```shell
$ docker run postgres postgres --help
```

最后，它也可以用来启动一个完全不同的工具，比如Bash：

```shell
$ docker run --rm -it postgres bash
```

### VOLUME

[VOLUME指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#volume)

`VOLUME`指令应该用于公开任何数据库存储区域，配置存储或由docker容器创建的文件/文件夹。强烈建议将镜像的任何可变和用户可维修部分的使用`VOLUME`配置。

### USER

[用户指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#user)

如果服务可以在没有权限的情况下运行，则使用`USER`更改为非root用户。首先通过类似的方式在`dockerfile`中创建用户和组`RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres`。

> **注意**：图像中的用户和组会得到非确定性的UID/GID，因为无论镜像重建如何，都会分配“下一个”UID/GID。所以，如果这很关键，你应该分配一个明确的UID/GID。

> **注意**：由于 `Go archive / tar`包处理松散文件时存在一个[未解决的错误](https://github.com/golang/go/issues/13548)，尝试在Docker容器内创建具有足够大`UID`的用户可能导致磁盘耗尽，因为`/var/log/faillog`容器层中充满了NUL（\ 0 ）字符。将该`--no-log-init `选项传递给`useradd`可以解决此问题。`Debian/Ubuntu` `adduser`包不支持该`--no-log-init`标志，应该避免。

避免安装或使用，因为`sudo`它具有可能导致问题的不可预测的TTY和信号转发行为。如果绝对需要类似于的功能`sudo`，例如初始化守护程序`root`但将其作为非运行`root`），请考虑使用 [“gosu”](https://github.com/tianon/gosu)。

最后，要减少层次和复杂性，请避免`USER`频繁来回切换。

### WORKDIR

[用于WORKDIR指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#workdir)

为了清晰和可靠，应该**始终使用绝对路径 `WORKDIR`**。此外应该使用`WORKDIR`而不是`RUN cd … && do-something`繁琐的命令，这是很难阅读和维护。

### ONBUILD

[ONBUILD指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#onbuild)

在当前的`dockerfile`构建完成后执行`ONBUILD`命令。`ONBUILD`在从**当前镜像派生的任何子镜像中执行**。把这个`ONBUILD`命令看作父`Dockerfile`给子`Dockerfile`的一条指令。

Docker构建在子`Dockerfile`文件中的任何命令之前执行`ONBUILD`命令。

`ONBUILD`对于将从给定镜像构建的镜像非常有用。例如，可以使用`ONBUILD`语言编排镜像来构建用该语言编写的任意用户软件 `Dockerfile`，如在[Ruby的`ONBUILD`变体中所](https://github.com/docker-library/ruby/blob/master/2.4/jessie/onbuild/Dockerfile)看到[的](https://github.com/docker-library/ruby/blob/master/2.4/jessie/onbuild/Dockerfile)。

`ONBUILD`建立的镜像应该得到一个单独的标签，例如： `ruby:1.9-onbuild`或`ruby:2.0-onbuild`。

`ADD`或`COPY`时要小心`ONBUILD`。如果新构建的上下文缺少正在添加的资源，那么“构建”镜像就会发生灾难性的失败。按照上面的建议添加单独的标签，通过允许`Dockerfile`做出选择来帮助缓解这种情况。

# 创建一个基础图像

大多数`Dockerfiles`从父镜像开始。如果需要完全控制图像的内容，则可能需要创建基础镜像。以下是区别：

- 一个[父镜像](https://docs.docker.com/glossary/?term=parent%20image)是你的镜像的基础镜像。它`FROM`指向`Dockerfile`中指令的内容。`Dockerfile`中的每个后续声明都会修改此父镜像。大多数`Dockerfiles`从父镜像开始，而不是基础镜像。但是，这些术语有时可以相互使用。
- 基础镜像在Dockerfile中要么不具有`FROM`，或具有`FROM scratch`。

下面展示了创建基础镜像的几种方法。具体的过程将在很大程度上取决于你想打包的Linux发行版。我们在下面有一些例子，并提交请求拉取以提供新的镜像。

## 使用tar创建一个完整的镜像


一般来说，从运行想要打包为父镜像的发行版的工作机器开始，尽管对于像`Debian`的[Debootstrap](https://wiki.debian.org/Debootstrap)这样的工具[来说](https://wiki.debian.org/Debootstrap)，你也可以使用它来构建`Ubuntu` 镜像，但这不是必需的 。

它可以像创建`Ubuntu`父图像一样简单：

```shell
$ sudo debootstrap xenial xenial > /dev/null
$ sudo tar -C xenial -c . | docker import - xenial

a29c15f1bf7a

$ docker run xenial cat /etc/lsb-release

DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04 LTS"
```

在`Docker GitHub Repo`中有更多用于创建父镜像的示例脚本：

- [BusyBox](https://github.com/moby/moby/blob/master/contrib/mkimage/busybox-static)
- CentOS / Scientific Linux CERN (SLC) [on Debian/Ubuntu](https://github.com/moby/moby/blob/master/contrib/mkimage/rinse) or [on CentOS/RHEL/SLC/etc.](https://github.com/moby/moby/blob/master/contrib/mkimage-yum.sh)
- [Debian / Ubuntu](https://github.com/moby/moby/blob/master/contrib/mkimage/debootstrap)

## 使用scratch创建一个简单的父图像


可以使用Docker的保留最小镜像`scratch`作为构建容器的起点。使用`scratch`镜像构建过程中，希望下一个命令`Dockerfile`成为图像中的第一个文件系统层。

当`scratch`出现在Docker存储库中时，无法将其拉出、运行或标记具有该名称的任何图像。相反，你可以参考你的`Dockerfile`。例如，要使用`scratch`以下命令创建最小容器 ：

```shell
FROM scratch
ADD hello /
CMD ["/hello"]
```

假设按照[https://github.com/docker-library/hello-world/中](https://github.com/docker-library/hello-world/)的说明构建了“hello”可执行文件示例，并且使用该`-static`标志编译了该示例 ，则可以使用以下`docker build`命令构建此Docker映像：

```shell
docker build --tag hello .
```

不要忘记最后的`.`字符，它将构建上下文设置为当前目录。

> **注意**：由于Docker for Mac和Docker for Windows使用Linux VM，因此需要Linux二进制文件，而不是Mac或Windows二进制文件。可以使用Docker容器来构建它：
>
> ```shell
> $ docker run --rm -it -v $PWD:/build ubuntu:16.04
>
> container# apt-get update && apt-get install build-essential
> container# cd /build
> container# gcc -o hello -static -nostartfiles hello.c
> ```

要运行新图像，请使用以下`docker run`命令：

```
docker run --rm hello
```

本示例将创建教程中使用的hello-world图像。如果你想测试它，你可以克隆 [图像回购](https://github.com/docker-library/hello-world)。



# 多阶段构建

多阶段构建是一项新功能，需要守护程序和客户端上的`Docker 17.05`或更高版本。多阶段构建对于那些努力优化Dockerfiles的人来说非常有用，同时让他们易于阅读和维护。

## 不使用多阶段构建


关于构建图像最具挑战性的事情之一是**保持图像的大小**。Dockerfile中的**每条指令都会为镜像添加一个图层**，并且需要记住在**移动到下一图层之前清理不需要的任何组件**。为了编写一个非常高效的Dockerfile，传统上需要使用shell技巧和其他逻辑来尽可能地减小层次，并确保每层都具有它需要的来自前一层的组件，而不是其他任何东西。

实际上，有一个Dockerfile用于开发（其中包含构建应用程序所需的所有内容）以及一个用于生产的瘦身客户端，它只包含应用程序以及运行它所需的内容。这被称为“建造者模式”。维护两个Dockerfiles并不理想。

这里有一个例子`Dockerfile.build`和`Dockerfile`它遵守上面建造者模式：

**Dockerfile.build**：

```dockerfile
FROM golang:1.7.3
WORKDIR /go/src/github.com/alexellis/href-counter/
COPY app.go .
RUN go get -d -v golang.org/x/net/html \
  && CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .
```

请注意，此示例还使用`RUN`Bash `&&`运算符人为地压缩两个命令，以避免在图像中创建额外的图层。这很容易失败并且很难维护。例如，插入另一个命令并忘记继续使用该`\`字符行很容易。

**Dockerfile**：

```dockerfile
FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY app .
CMD ["./app"]  
```

**build.sh**：

```shell
#!/bin/sh
echo Building alexellis2/href-counter:build

docker build --build-arg https_proxy=$https_proxy --build-arg http_proxy=$http_proxy \  
    -t alexellis2/href-counter:build . -f Dockerfile.build

docker container create --name extract alexellis2/href-counter:build  
docker container cp extract:/go/src/github.com/alexellis/href-counter/app ./app  
docker container rm -f extract

echo Building alexellis2/href-counter:latest

docker build --no-cache -t alexellis2/href-counter:latest .
rm ./app
```

运行`build.sh`脚本时，需要构建第一个镜像，从中创建一个容器以将该构件复制出来，然后构建第二个镜像。这两个镜像都占用了系统空间，并且`app` 的本地磁盘上仍然存在工件。

多阶段构建大大简化了这种情况！

## 使用多阶段构建


使用多阶段构建，可以`FROM`在Dockerfile中使用多个语句。每条`FROM`指令都可以使用不同的基础，并且每条指令都开始构建的新阶段。可以选择性地将工件从一个阶段复制到另一个阶段，在最终镜像中留下不需要的所有内容。为了演示这是如何工作的，让我们修改上一节中的Dockerfile以使用多阶段构建。

**Dockerfile**：

```dockerfile
FROM golang:1.7.3
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html  
COPY app.go .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=0 /go/src/github.com/alexellis/href-counter/app .
CMD ["./app"]  
```

你只需要单个Dockerfile，也不需要单独的构建脚本。运行命令`docker build`。

```shell
$ docker build -t alexellis2/href-counter:latest .
```

最终的结果是生产与以前相同的小型镜像，并显着降低了复杂性。不需要创建任何中间镜像，也不需要将任何组件提取到本地系统。

它是如何工作的？第二`FROM`条指令以`alpine:latest`图像为基础开始新的构建阶段。**该`COPY --from=0`行只将前一阶段构建的组件复制到这个新阶段**。Go SDK和任何中间组件都被留下，并未保存在最终图像中。

## 为构建阶段命名


默认情况下，这些阶段没有命名，并且**可以用它们的整数来引用它们，从第一`FROM`条指令的0开始。但是，可以通过`as <NAME>`在`FROM`指令中添加一个名称来命名阶段**。此示例通过命名阶段并在`COPY`指令中使用名称来改进前一个示例。这意味着即使Dockerfile中的指令稍后重新排序，`COPY`也不会中断。

```dockerfile
FROM golang:1.7.3 as builder # 进行命名
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html  
COPY app.go    .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
# --from=builder 不再是 0
COPY --from=builder /go/src/github.com/alexellis/href-counter/app .
CMD ["./app"]  
```

## 停止在特定的构建阶段


当构建图像时，不一定需要构建包括每个阶段的整个Dockerfile。可以指定目标构建阶段。以下命令假定正在使用`previous`，`Dockerfile`但在名为的阶段停止`builder`：

```shell
$ docker build --target builder -t alexellis2/href-counter:latest .
```

这可能非常强大的几种情况是：

- 调试特定的构建阶段
- 使用`debug`启用了所有调试符号或工具的调试阶段以及生产阶段
- 使用一个`testing`阶段，应用程序获取测试数据，但使用实际数据的不同阶段构建生产

## 将外部镜像用到阶段构建中

在使用多阶段构建时，不限于从之前在Dockerfile中创建的阶段进行复制。可以使用该`COPY --from`指令从单独的镜像进行复制，可以使用本地镜像名称，本地或Docker注册表中可用的标记或标记ID。Docker客户端在必要时拉取镜像并从那里复制组件。语法是：

```dockerfile
# 复制nginx镜像中的/etc/nginx/nginx.conf 配置
COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```



# Docker Engine SDK & API 的应用

Docker提供了一个用于与Docker守护进程（称为`Docker Engine API`）交互的API，以及用于Go和Python的SDK。**SDK允许您快速轻松地构建和扩展Docker应用程序和解决方案**。如果Go或Python不适合你，你可以直接使用`Docker Engine API`。

`Docker Engine API`是由诸如`wget`之类的HTTP客户端或 作为`curl`大多数现代编程语言的一部分的HTTP库访问的`RESTful API` 。

## 安装 Python SDK


使用以下命令来安装Go或Python SDK。这两个SDK都可以安装并共存在一起。

- **建议**：运行`pip install docker`。
- **如果你不能使用pip**：
  1. [直接下载软件包](https://pypi.python.org/pypi/docker/)。
  2. 提取它并更改为提取的目录，
  3. 运行`python setup.py install`。

[阅读完整的Docker引擎Python SDK参考](https://docker-py.readthedocs.io/)。



## API参考


可以 [查看API最新版本的参考](https://docs.docker.com/engine/api/latest/) 或[选择特定版本](https://docs.docker.com/engine/api/version-history/)



## SDK & API 快速入门


运行示例前请先确保 python环境已经安装完毕，并且对python有一定的使用时间。下面的是python调用SDK接口的示例：

```python
#!/usr/bin/python3
import docker

'''
SDK 使用参考文档：https://docker-py.readthedocs.io/en/stable/client.html
API 接口文档：https://docs.docker.com/engine/api/latest/
示例参考：https://docs.docker.com/develop/sdk/examples/
'''

#help(docker.from_env)
client = docker.from_env()

# 设置API的版本： https://192.168.99.100:2376/v1.70/containers/xxx
#client = docker.from_env(version="1.70")

# 执行容器程序
print(client.containers.run("alpine", ["echo", "hello", "world"]))
print(client.containers.run("alpine", "echo hello world！"))

# 容器列表
list = client.containers.list()
for item in list:
    #print(item.attrs['Config'])
    print('container: %s \t %s \t %s' % (item.id, item.name, item.image))
    
# 获取一个容器
item = client.containers.get("a36193c9bd3c")
print('container: %s \t %s \t %s' % (item.id, item.name, item.image))  

# 容器日志
#for line in item.logs(stream=True):
#    print(line.strip())  
    

# 镜像列表
list = client.images.list()    
for item in list:
    #print(item.attrs['Config'])
    print('image: %s \t %s \t %s' % (item.id, item.labels, item.tags))
    
# docker info
print(client.info())    

# ping
print(client.ping())
```



