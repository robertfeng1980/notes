# docker 镜像开发

[TOC]

# 概述

## 在Docker上开发新的应用程序

介绍以下内容的应用：

- 学习[从Dockerfile构建镜像](https://docs.docker.com/get-started/part2/)
- 使用[多阶段构建](https://docs.docker.com/engine/userguide/eng-image/multistage-build/)来保持镜像精简
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

Docker可以通过从`Dockerfile`文本文件中读取指令自动构建镜像 ，以便构建给定镜像。`Dockerfile`是一个文本文档，其中包含用户可以在命令行上调用以组装镜像的所有命令。`Dockerfile`遵循特定的格式并使用特定的指令集。可以在[Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)页面上了解基础知识 。

本文档介绍了由`Docker Inc.`和Docker社区推荐的用于**构建高效镜像的最佳实践和方法**。要查看许多实践和建议，请查看Dockerfile for [buildpack-deps](https://github.com/docker-library/buildpack-deps/blob/master/jessie/Dockerfile)。

> **注意**：有关此处提及的任何Dockerfile命令的更详细说明，请访问[Dockerfile参考](https://docs.docker.com/engine/reference/builder/)页面。



## 准则和建议

### 容器保持小巧轻盈

---

由`dockerfile`定义的镜像生成的容器应尽可能短暂。通过“短暂的”，我们的意思是它可以被停止并被销毁，并且一个新的建立和配置到位，并具有绝对最小的设置和配置。您可能需要查看**12因子**应用程序方法的[流程](https://12factor.net/processes)部分，以了解以这种无状态方式运行容器的动机。

### 构建上下文

---

当发出一个`docker build`命令时，当前的工作目录被称为*构建上下文*。默认情况下，假定`Dockerfile`位于此处，但可以使用文件选项（`-f`）指定不同的位置。无论`Dockerfile`文件实际在哪里，当前目录中的所有文件和目录的递归内容都将作为构建上下文发送到Docker守护进程。

> **构建上下文示例**
>
> 为构建上下文创建一个目录，并`cd`进入文件目录。写入一个名为“hello”文本文件，内容是`hello`，并创建一个Dockerfile运行`cat`查看文件内容。从构建上下文（`.`）中构建镜像：
>
> ```shell
> $ mkdir sample && cd sample
>
> $ echo "hello" > hello
> $ echo -e "FROM busybox\nCOPY /hello /\nRUN cat /hello" > Dockerfile
> $ docker build -t hello_app:v1 .
> ```
>
> 现在移动`Dockerfile`和`hello`到不同的目录，并建立了镜像的第二个版本（不依赖于缓存中的最后一个版本）。使用`-f`指向Dockerfile并指定构建上下文的目录：
>
> ```shell
> $ mkdir -p dockerfiles codes
> $ mv Dockerfile dockerfiles && mv hello codes
> $ docker build --no-cache -t hello_app:v2 -f dockerfiles/Dockerfile codes
> ```

包含不需要构建镜像的文件会**导致更大的构建上下文和更大的镜像大小**。这可以**增加构建时间，拉取和推送镜像的时间以及容器的运行时间大小**。要查看构建环境有多大，请在构建系统时查找这样的消息`Dockerfile`：

```shell
Sending build context to Docker daemon  2.607kB
```

### 使用`.dockerignore`文件

---

要**排除与构建无关的文件，而不重构源代码库**，请使用`.dockerignore`文件。该文件支持与`.gitignore`文件类似的排除模式。创建一个有关的信息，请参阅[.dockerignore文件](https://docs.docker.com/engine/reference/builder/#dockerignore-file)。除了使用`.dockerignore`文件外，请查看以下关于[多阶段构建的信息](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#use-multi-stage-builds)。

### 使用多阶段构建

---

如果使用Docker 17.05或更高版本，则可以使用 [多阶段构建](https://docs.docker.com/develop/develop-images/multistage-build/)来大幅**缩减最终镜像的大小**，而无需在构建过程中跳过环节以减少中间层数量或删除中间文件。

镜像仅由最后阶段构建，**大多数时间都可以同时构建缓存和镜像层**。

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

# 最终这导致了单层镜像
FROM scratch
COPY --from=build /bin/project /bin/project
ENTRYPOINT ["/bin/project"]
CMD ["--help"]
```

### 避免安装不必要的软件包

---

为了减少复杂性、依赖性、文件大小和构建时间，应该避免安装额外的或不必要的软件包，只是因为它们可能“很好”。例如，不需要在数据库镜像中包含文本编辑器或第三方编辑工具。

### 每个容器一个进程

---

应用程序**解耦为多个容器使得横向扩展和重用容器**变得更容易。例如，Web应用程序编排服务可能由三个独立的容器组成，每个容器都有其独立的镜像，以分离的方式管理Web应用程序、数据库和内存缓存。

可能听说应该有“每个容器一个进程”。虽然这个口头禅的意图很好，但并不一定每个容器只有一个操作系统进程。除了现在可以[使用init进程创建](https://docs.docker.com/engine/reference/run/#specifying-an-init-process)容器之外，一些程序可能会自行产生其他进程。例如，[Celery](http://www.celeryproject.org/)可以派生多个工作进程，或者[Apache](https://httpd.apache.org/)可能会为每个请求创建一个进程。虽然“每个容器一个进程”往往是一个很好的经验法则，但它并不是一条硬性规定。尽你最大的努力使容器**保持干净和模块化**。

**如果容器相互依赖，则可以使用[Docker容器网络](https://docs.docker.com/engine/userguide/networking/) 来确保这些容器可以通信**。

### 镜像图层最小化

---

在Docker 17.05之前，甚至更多的是在Docker 1.10之前，重要的是**尽量减少镜像中的图层数量**。以下改进缓解了这种需求：

- 在docker 1.10和更高，只有`RUN`，`COPY`和`ADD`指令创建图层。其他动作指令会创建临时中间镜像，不再直接增加构建的大小。
- Docker 17.05和更高版本添加了对[多阶段构建的](https://docs.docker.com/develop/develop-images/multistage-build/)支持 ，这允许只将需要的**构件复制到最终镜像中**。允许在**中间构建阶段包含工具和调试信息，而不增加最终镜像的大小**。

### 多行参数进行排序

---

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

---

在构建镜像的过程中，`Dockerfile`按照指定顺序执行每个指令。在检查每条指令时，**Docker会在其缓存中查找可重用的现有镜像**，而不是创建新的（重复）镜像。如果根本不想使用缓存，则可以使用命令中的`--no-cache=true`选项进行`docker build`。

但是，如果确实让Docker使用了它的缓存，那么了解它何时可以并且无法找到匹配的镜像是非常重要的。Docker遵循的基本规则如下：

- 从已经在缓存中的父镜像开始，**将下一条指令与从该基本镜像派生的所有子镜像进行比较**，以查看是否使用完全相同的指令构建。否则缓存失效。
- 在大多数情况下，简单地比较`Dockerfile`一个**子镜像中的指令**就足够了。但是，某些指令动作需要更多的**检查和解释**。
- 对于`ADD`和`COPY`，将检查镜像中文件的内容，并为每个文件计算校验`checksum `。这些校验`checksum `不考虑文件的最后修改时间和最后访问时间。在缓存查找过程中，**将校验`checksum `与现有镜像中的校验`checksum `进行比较。如果文件中有任何内容已更改，如内容和元数据，则缓存将失效**。
- 除了`ADD`和`COPY`命令之外，缓存检查**不会查看容器中**的文件以确定缓存匹配。例如，在处理`RUN apt-get -y update`命令时，**不检查容器中更新的文件以确定是否存在缓存命中**。在这种情况下，只是**命令字符串本身**用于查找匹配。

一旦缓存失效，所有后续的`Dockerfile`命令将**生成新镜像**，并且不使用缓存。



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

Docker守护进程会逐个运行`Dockerfile`指令，在必要时将每条指令的结果提交给新镜像，最终输出新镜像的标识。**Docker守护进程将自动清理发送的上下文**。

请注意，**每条指令都是独立运行的，并会创建一个新镜像， 所以`RUN cd /tmp`不会对下一条指令产生任何影响**。

只要有可能Docker将重新使用中间镜像（缓存），以`docker build`显著加速该过程。这由`Using cache`控制台输出中的消息指示。（有关更多信息，请参阅最佳做法指南中的 “ [构建缓存”部分](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#build-cache)`Dockerfile`）：

```shell
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

构建缓存仅用于具有本地父链的镜像。这意味着这些镜像是由以前的版本创建的，或者是整个镜像链`docker load`加载的。如果您希望使用特定镜像的构建缓存，则可以使用`--cache-from`选项指定它。指定的镜像`--cache-from`不需要有父链，可以从其他注册表中提取。

当你完成你的构建时，你已经准备好考虑[*将存储库推送到它的注册表*](https://docs.docker.com/engine/tutorials/dockerrepos/#/contributing-to-docker-hub)。



## 格式

以下是`Dockerfile`格式：

```dockerfile
# Comment
INSTRUCTION arguments
```

该指令**不区分大小写**。然而，约定是为了更容易将它们与其他文本区分开来。

Docker按顺序运行 `Dockerfile`指令，`Dockerfile` **必须用`FROM`指令启动**。`FROM`指令指定正在构建的[*基本镜像*](https://docs.docker.com/engine/reference/glossary/#base-image)。`FROM`只可以在前面有一个或多个`ARG`指令，这些指令声明`Dockerfile`中来`FROM` 的参数

docker以`＃`开头的行作为注释，除非该行是一个有效的[解析器指令](https://docs.docker.com/engine/reference/builder/#parser-directives)。`＃`标记被视为参数，这允许像这样的语句：

```shell
# Comment
RUN echo 'we are running some # of cool things'
```

注释中不支持换行符。



## 解析器指令

解析器指令是可选的，并影响 `Dockerfile`中后续行的处理方式。**解析器指令不会向构建添加图层**，并且不会显示为构建步骤。解析器指令在表单中被写为特殊类型的注释`# directive=value`。单个指令只能使用一次。

**一旦注释、空行或构建器指令已被处理**，Docker不再查找解析器指令。相反，它将格式化为分析器指令的任何内容视为注释，并且不会尝试验证它是否可能是解析器指令。因此，所有解析器指令都必须位于最顶端`Dockerfile`。

解析器指令不区分大小写。但是，**约定是小写**。约定还包括在任何解析器指令之后的空白行。解析器指令不支持行连续字符。

由于这些规则，以下示例都是无效的：

由于行延续而无效：

```
# direc \
tive=value
```

由于出现两次而无效：

```
# directive=value1
# directive=value2

FROM ImageName
```

由于出现在建造者指令之后，作为注释对待：

```
FROM ImageName
# directive=value
```

由于在不是解析器指令的注释之后出现，所以作为注释处理：

```
# About my dockerfile
# directive=value
FROM ImageName
```

未知指令由于未被识别而被视为注释。另外，由于在不是解析器指令的注释之后出现，已知指令被视为注释。

```
# unknowndirective=value
# knowndirective=value
```

解析器指令中允许使用非分行空白。因此，以下几行都是一致对待的：

```
#directive=value
# directive =value
#	directive= value
# directive = value
#	  dIrEcTiVe=value
```

以下解析器指令受支持：

- `escape`

```
# escape=\ (backslash)
```

要么

```
# escape=` (backtick)
```

该`escape`指令设置**用于转义字符**的字符 `Dockerfile`。如果未指定，则默认转义字符为`\`。转义字符既用于转义一行中的字符，又用于转义换行符。这允许`Dockerfile`指令跨越多行。请注意，无论`escape`解析器指令是否包含在`Dockerfile`中，*转义都不在RUN命令中执行，除了在行的末尾。*

将转义字符设置为在 `Windows`特别有用，其中`\`是目录路径分隔符。与[Windows PowerShell](https://technet.microsoft.com/en-us/library/hh847755.aspx)一致。

考虑下面的例子，它将在Windows上以非明显的方式失败。在第二行结尾的第二个`\`将被解释为换行符的转义，而不是从第一个`\`的转义目标。同样，`\`假设它实际上是作为一条指令处理的，那么在第三行的末尾会导致它被视为连续行。这个dockerfile的结果是第二行和第三行被认为是一条指令：

```dockerfile
FROM microsoft/nanoserver
COPY testfile.txt c:\\
RUN dir c:\
```

结果是：

```powershell
PS C:\John> docker build -t cmd .
Sending build context to Docker daemon 3.072 kB
Step 1/2 : FROM microsoft/nanoserver
 ---> 22738ff49c6d
Step 2/2 : COPY testfile.txt c:\RUN dir c:
GetFileAttributesEx c:RUN: The system cannot find the file specified.
PS C:\John>
```

上述的一个解决方案将是`/`用作`COPY` 指令和目标`dir`。然而，这种语法充其量是令人困惑的，因为路径并不是自然的`Windows`，并且最坏的情况是容易出错，因为并非所有`Windows`支持的命令都 `/`作为路径分隔符。

通过添加`escape`解析器指令，可以`Dockerfile`按照预期的方式成功执行以下文件路径的自然平台语义`Windows`：

```dockerfile
# escape=`

FROM microsoft/nanoserver
COPY testfile.txt c:\
RUN dir c:\
```

结果是：

```powershell
PS C:\John> docker build -t succeeds --no-cache=true .
Sending build context to Docker daemon 3.072 kB
Step 1/3 : FROM microsoft/nanoserver
 ---> 22738ff49c6d
Step 2/3 : COPY testfile.txt c:\
 ---> 96655de338de
Removing intermediate container 4db9acbb1682
Step 3/3 : RUN dir c:\
 ---> Running in a2c157f842f5
 Volume in drive C has no label.
 Volume Serial Number is 7E6D-E0F7

 Directory of c:\

10/05/2016  05:04 PM             1,894 License.txt
10/05/2016  02:22 PM    <DIR>          Program Files
10/05/2016  02:14 PM    <DIR>          Program Files (x86)
10/28/2016  11:18 AM                62 testfile.txt
10/28/2016  11:20 AM    <DIR>          Users
10/28/2016  11:20 AM    <DIR>          Windows
           2 File(s)          1,956 bytes
           4 Dir(s)  21,259,096,064 bytes free
 ---> 01c7f3bef04f
Removing intermediate container a2c157f842f5
Successfully built 01c7f3bef04f
PS C:\John>
```



## 环境替换

环境变量（与声明[的`ENV`声明](https://docs.docker.com/engine/reference/builder/#env)），也可以在特定指令作为变量用来被解释 `Dockerfile`。转义也被处理为从字面上将类似于变量的语法包括到语句中。

环境变量在`Dockerfile`中用 `$variable_name`或`${variable_name}`表示。它们被等同处理，大括号语法通常用于解决变量名称中没有空格的问题，如`${foo}_bar`。

该`${variable_name}`语法还支持一些标准`bash` 修饰符，如下所示：

- `${variable:-word}`表明如果`variable`设置，则结果将是该值。如果`variable`没有设置，那么`word`将会是结果。
- `${variable:+word}`表示如果`variable`设置则返回`word`结果，否则结果为空字符串。

在任何情况下，`word`都可以是任何字符串，包括其他环境变量。

通过`\`在变量之前添加一个变量可以进行转义：`\$foo`或者`\${foo}`，例如分别转换为文字`$foo`和`${foo}`文字。

示例（在之后显示解析的表示`#`）：

```dockerfile
FROM busybox
ENV foo /bar
WORKDIR ${foo}   # WORKDIR /bar
ADD . $foo       # ADD . /bar
COPY \$foo /quux # COPY $foo /quux
```

以下列出的指令支持环境变量`Dockerfile`：

- `ADD`
- `COPY`
- `ENV`
- `EXPOSE`
- `FROM`
- `LABEL`
- `STOPSIGNAL`
- `USER`
- `VOLUME`
- `WORKDIR`

以及：

- `ONBUILD` （当与上述支持的指令之一结合使用时）

> **注意**：在1.4之前，`ONBUILD`说明**不**支持环境变量，即使与上面列出的任何指令结合使用。

整个指令中的环境变量替换将对每个变量使用相同的值。换句话说，在这个例子中：

```dockerfile
ENV abc=hello
ENV abc=bye def=$abc
ENV ghi=$abc
```

将导致`def`的值为`hello`，而不是`bye`。然而`ghi`将具有`bye`值，因为它不是将`abc`设置为`bye`的同一指令的一部分。

## .dockerignore文件

在`docker CLI`将上下文发送到docker守护程序之前，它会查找`.dockerignore`上下文根目录中指定的文件。如果此文件存在，CLI会修改上下文以排除与其中的模式匹配的文件和目录。这有助于避免不必要地将大型或敏感文件和目录发送到守护进程，并可能使用`ADD`或将其添加到映像`COPY`。

CLI将`.dockerignore`文件解释为与`Unix shell`文件大小相似的以换行符分隔的模式列表。为了匹配的目的，上下文的根被认为是工作目录和根目录。例如，这些模式 `/foo/bar`和`foo/bar`两者都不包含位于位于的git存储库`bar` 的`foo`子目录`PATH`或根目录中指定的文件或目录`URL`。两者都没有排除其他任何东西。

如果`.dockerignore`文件中的一行以第一`#`列开始，那么该行将被视为注释，并在被CLI解释之前被忽略。

这是一个示例`.dockerignore`文件：

```
# comment
*/temp*
*/*/temp*
temp?
```

该文件导致以下构建行为：

| 规则        | 行为                                                         |
| ----------- | ------------------------------------------------------------ |
| `# comment` | 忽略。                                                       |
| `*/temp*`   | 排除名称以`temp`根目录的任何直接子目录开头的文件和目录。例如，普通文件`/somedir/temporary.txt`和目录一样被排除`/somedir/temp`。 |
| `*/*/temp*` | `temp`从根目录下两个级别的任何子目录开始排除文件和目录。例如，`/somedir/subdir/temporary.txt`被排除在外。 |
| `temp?`     | 排除名称为一个字符扩展名的根目录中的文件和目录`temp`。例如，`/tempa`与`/tempb`被排除在外。 |

匹配是使用Go的 [filepath.Match](http://golang.org/pkg/path/filepath#Match)规则完成的。预处理步骤删除前导和尾随空白，`.`并`..`使用Go的[filepath.Clean](http://golang.org/pkg/path/filepath/#Clean)消除和元素 。预处理后空白的行将被忽略。

除了Go的filepath.Match规则，Docker还支持`**`匹配任意数量目录（包括零）的特殊通配符字符串。例如，`**/*.go`将排除`.go` 以所有目录中包含的文件结尾的所有文件，包括构建上下文的根目录。

以`!`（感叹号）开头的行可用于排除例外情况。以下是`.dockerignore`使用此机制的示例文件：

```
    *.md
    !README.md
```

*除* `README.md`上下文*外*，所有文件均被排除在外，有点类似白名单的功能。

`!`例外规则的放置会影响行为：`.dockerignore`匹配特定文件的最后一行确定是否包含或排除。考虑下面的例子：

```
    *.md
    !README*.md
    README-secret.md
```

上下文中除了包含README文件外，不包括任何降价文件 `README-secret.md`。

现在考虑这个例子：

```
    *.md
    README-secret.md
    !README*.md
```

所有的README文件都包含在内。中间行不起作用，因为`!README*.md`与`README-secret.md`最终匹配成功。

你甚至可以使用该`.dockerignore`文件来排除`Dockerfile` 和`.dockerignore`文件。这些文件仍然被发送到守护进程，因为它需要它们来完成它的工作。但是`ADD`和`COPY`指令不会将它们复制到镜像。

最后，可能需要指定要在上下文中包含哪些文件，而不是要排除的文件。要实现这一点，请指定`*`第一个模式，然后指定一个或多个`!`异常模式。

**注意**：由于历史原因，该模式`.`被忽略。



## Dockerfile 指令



### FROM

---

[用于FROM指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#from)

只要有可能，请使用当前的官方存储库作为基础镜像。我们推荐使用[Alpine镜像](https://hub.docker.com/_/alpine/) 因为它的控制非常严格，并且保持最低限度（目前低于`5 MB`），同时仍然是完整的发行版。

```dockerfile
FROM <image> [AS <name>]
FROM <image>[:<tag>] [AS <name>]
FROM <image>[@<digest>] [AS <name>]

example：
FROM redis as firstBuild
FROM redis:v2.3 as firstBuild
FROM redis@digest as firstBuild
```

`FROM`指令初始化一个新的编译阶段并为后续指令设置 [*基础镜像*](https://docs.docker.com/engine/reference/glossary/#base-image)。因此，有效的`Dockerfile`必须以`FROM`指令开始。镜像可以是任何有效的镜像—通过从[*公共存储库中*](https://docs.docker.com/engine/tutorials/dockerrepos/)**拉取镜像**特别容易。

- `ARG`是在`Dockerfile`中可能在`FROM`之前的唯一指令。
- `FROM`可以在单个`Dockerfile`内出现多次以创建多个镜像，或者使用一个构建阶段作为另一个构建阶段的依赖项。只需在每条新`FROM`指令之前记录提交输出的最后一个镜像ID 。每条`FROM`指令都会清除以前指令创建的任何状态。
- 通过`FROM`指令添加`AS name`，可以给新的构建阶段赋予名称。该名称可以用于后续`FROM`和`COPY --from=<name|index>`指令中，以引用此阶段中构建的镜像。
- 该`tag`或`digest`值是可选的。如果忽略其中的任何一个，则默认情况下，构建器会采用`latest`标签。如果找不到`tag`值，构建器将返回错误。



#### ARG 和 FROM 如何交互

`FROM`指令支持由第一个`FROM`之前发生的任何`ARG`指令声明的变量。

```dockerfile
ARG  CODE_VERSION=latest
FROM base:${CODE_VERSION}
CMD  /code/run-app

FROM extras:${CODE_VERSION}
CMD  /code/run-extras
```

在`FROM`之前声明的`ARG`在构建阶段，它不能在`FROM`之后的任何指令使用。要使用在第一个`FROM`之前声明的`ARG`的默认值，在构建阶段中使用没有值的`ARG`指令：

```dockerfile
ARG VERSION=latest
FROM busybox:$VERSION
ARG VERSION
RUN echo $VERSION > image_version
```

### LABEL
---

[了解对象标签](https://docs.docker.com/config/labels-custom-metadata/)

可以将标签添加到镜像中，以帮助按项目组织镜像、记录许可信息、帮助自动化或出于其他原因。添加`LABEL`以一个或多个键值对开头的行。以下示例显示了不同的格式。

> **注意**：如果你的字符串包含空格，那么它必须被引号包围或者**空格必须被转义**。如果你的字符串包含内部引号字符（`"`），也可以将它们转义。

```dockerfile
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

该`LABEL`指令将元数据添加到图像。 `LABEL`是一个键值对。要在`LABEL`值中包含空格，请像在命令行解析中一样使用引号和反斜杠。几个用法示例：

```dockerfile
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```

可以在一行中指定多个标签。在Docker 1.10之前，这减少了最终映像的大小，但现在不再是这种情况。仍然可以选择使用以下两种方法之一在单条指令中指定多个标签：

```dockerfile
LABEL multi.label1="value1" multi.label2="value2" other="value3"
```

```dockerfile
LABEL multi.label1="value1" \
      multi.label2="value2" \
      other="value3"
```

包含在基本图像或父图像中的标签（行中的图像`FROM`）由图像继承。**如果标签已经存在但使用不同的值，则最近应用的值将覆盖任何先前设置的值**。

要查看图像的标签，请使用该`docker inspect`命令。

```
"Labels": {
    "com.example.vendor": "ACME Incorporated"
    "com.example.label-with-value": "foo",
    "version": "1.0",
    "description": "This text illustrates that label-values can span multiple lines.",
    "multi.label1": "value1",
    "multi.label2": "value2",
    "other": "value3"
},
```

### RUN

---

[运行指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#run)

为了使`Dockerfile`可读性更强，易于理解并且可维护，可以`RUN`在**多行用反斜杠分隔长分或复杂语句**

RUN有两种形式：

- `RUN <command>`（`shell`形式，该命令在shell中运行，默认情况下`/bin/sh -c`在Linux或`cmd /S /C`Windows 上运行）
- `RUN ["executable", "param1", "param2"]`（`exec`形式）

该`RUN`指令将在当前镜像顶部的新图层中执行任何命令并提交结果。由此产生的提交镜像将用于下一步`Dockerfile`。

分层`RUN`指令和生成提交符合Docker的核心概念，其中提交很方便，容器可以从镜像历史中的任意点创建，就像源代码控制一样。

在*EXEC*形式使得能够避免`shell`串改写（munging），并使用不包含指定的`shell`可执行文件的基本镜像`RUN`命令。

可以使用`shell`命令更改`shell`模式的默认`shell`。

在*shell*模式中，可以使用`\`（反斜杠）将单个`RUN`指令继续到下一行。例如，考虑这两行：

```dockerfile
RUN /bin/bash -c 'source $HOME/.bashrc; \
echo $HOME'
```

它们一起等同于：

```dockerfile
RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'
```

> **注意**：
>
> 要使用不同于`/bin/sh` 的版本的`shell`命令行 ，请使用传递需要的shell 命令行客户端给*exec*模式。<br/>例如，`RUN ["/bin/bash", "-c", "echo hello"]`
>
> *exec*模式被解析为JSON数组，这意味着必须在单词周围使用双引号。<br/>例如，`RUN ["/bin/bash", "-c", "echo hello"]`
>
> 与*shell*形式不同，*exec*形式不会调用命令shell。这意味着正常的shell处理不会发生。<br/>例如，`RUN [ "echo", "$HOME" ]`不会对变量进行替换`$HOME`。<br/>如果想要进行shell处理，请使用*shell* 模式或直接执行shell，<br/>例如：`RUN [ "sh", "-c", "echo $HOME" ]` & `RUN /bin/sh -c echo $HOME`<br/>当使用exec模式并直接执行一个shell时（如shell格式的情况），它是在执行环境变量扩展的shell，而不是docker。
>
> 在*JSON*形式中，有必要避免反斜杠。反斜杠是路径分隔符在Windows上尤其重要。由于不是有效的JSON，以下行将被视为*shell*形式，并以意外的方式失败： `RUN ["c:\windows\system32\tasklist.exe"]` 此示例的正确语法是： `RUN ["c:\\windows\\system32\\tasklist.exe"]`

`RUN`指令缓存在下一次构建期间不会自动失效。类似指令的缓存 `RUN apt-get dist-upgrade -y`将在下一次构建时重用。例如，`RUN`指令缓存可以通过使用`--no-cache` 标志来使其失效`docker build --no-cache`。

说明的高速缓存`RUN`可能会因`ADD`指令而失效。详情请参阅 [下文](https://docs.docker.com/engine/reference/builder/#add)。

**已知问题**

- [问题783](https://github.com/docker/docker/issues/783)是关于使用AUFS文件系统时可能发生的文件权限问题。例如，可能会在尝试`rm`使用文件时注意到它。

  对于具有最新aufs版本的系统（即，`dirperm1`可以设置安装选项），docker将尝试通过安装图层`dirperm1`选项来自动修复问题。有关`dirperm1`选项的更多详细信息，请参见[`aufs`手册页](https://github.com/sfjro/aufs3-linux/tree/aufs3.18/Documentation/filesystems/aufs)

  如果您的系统不支持`dirperm1`，该问题描述了一种解决方法。

  ​

#### apt-get

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

Docker使用`/bin/sh -c`解释器执行这些命令，解释器只评估管道中最后一个操作的退出代码以确定是否成功。在上面的示例中，只要`wc -l`命令成功，即使`wget`命令失败，该构建步骤也会成功并生成新镜像。

如果由于管道中任何阶段的错误而导致命令失败，请预先`set -o pipefail &&`确保意外错误可防止构建无意中成功。例如：

```dockerfile
RUN set -o pipefail && wget -O - https://some.site | wc -l > /number
```

> **注意**：并非所有`shell`都支持该`-o pipefail`选项。在这种情况下（例如`dash`shell，这是基于`Debian`的镜像上的默认`shell`），考虑使用`run`的`exec`形式来显式选择一个支持`pipefail`选项的`shell`。例如：

```dockerfile
RUN ["/bin/bash", "-c", "set -o pipefail && wget -O - https://some.site | wc -l > /number"]
```

### CMD

---

[CMD指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#cmd)

该`CMD`指令应该用于运行镜像像中包含的软件以及任何参数。`CMD`应该几乎总是以`CMD [“executable”, “param1”, “param2”…]`形式使用。因此，如果镜像是用于服务的，比如`Apache`和`Rails`，那么你可以运行类似的 `CMD ["apache2","-DFOREGROUND"]`。这种形式的指令被推荐用于任何基于服务的镜像。

在其他大多数情况下，`CMD`应该给出一个交互式`shell`，比如`bash`，`python`和`perl`。例如，`CMD ["perl", "-de0"]`，`CMD ["python"]`，或 `CMD [“php”, “-a”]`。使用这种形式意味着当你执行`docker run -it python`类似的东西时，你会被放入一个可用的`shell`中，随时可以使用。应该很少使用 `CMD`的方式`CMD [“param”, “param”]`会同[`ENTRYPOINT`](https://docs.docker.com/engine/reference/builder/#entrypoint)，除非你和你预期的用户已经非常熟悉如何`ENTRYPOINT` 工作的。

该`CMD`指令有三种形式：

- `CMD ["executable","param1","param2"]`（`exec`方式，这是首选方式）
- `CMD ["param1","param2"]`（作为`ENTRYPOINT`*的默认参数*，每次都运行相同的可执行文件）
- `CMD command param1 param2`（`shell`方式其实是 `/bin/sh -c`上执行）

**`Dockerfile`只能有一条`CMD`指令。如果列出多个`CMD` 则只有最后一个`CMD`会生效。**

**CMD的主要目的是为执行容器提供默认值。**这些默认值可以包含可执行文件，也可以省略可执行文件，在这种情况下，还必须指定一条`ENTRYPOINT` 指令。

> **注意**：
>
> 如果`CMD`用于为`ENTRYPOINT` 指令提供缺省参数，则应该使用JSON数组格式指定`CMD`和`ENTRYPOINT`指令。
>
> *exec*形式被解析为JSON数组，这意味着必须在单词周围使用双引号。<br/>例如，`CMD ["/bin/bash", "-c", "echo hello"]`
>
> 与*shell*形式不同，*exec*形式不会调用命令shell。这意味着正常的shell处理不会发生。<br/>例如，`CMD [ "echo", "$HOME" ]`不会对变量进行替换`$HOME`。<br/>如果您想要进行shell处理，请使用*shell*方式或直接执行shell，<br/>例如：`CMD [ "sh", "-c", "echo $HOME" ]`。<br/>当使用exec方式并直接执行一个shell时（如shell格式的情况），它是在执行环境变量扩展的shell，而不是docker。

当以shell或exec格式使用时，`CMD`指令设置运行映像时要执行的命令。<br/>如果你使用的是*shell的*形式`CMD`，那么`<command>`将执行在 `/bin/sh -c`：

```
FROM ubuntu
CMD echo "This is a test." | wc -
```

如果想在 `<command>` **没有shell** **的情况下运行你的程序，**那么必须将该命令表示为一个**JSON数组并给出可执行文件的完整路径。 这个数组形式是的首选格式CMD。任何附加参数都必须单独表示为数组中的字符串**：

```
FROM ubuntu
CMD ["/usr/bin/wc","--help"]
```

**如果希望容器每次都运行相同的可执行文件**，那么应该考虑`ENTRYPOINT`与其结合使用`CMD`。见 [*入口点*](https://docs.docker.com/engine/reference/builder/#entrypoint)。

如果用户指定了`docker run`参数，那么它们将覆盖`CMD`中指定的默认值。

> **注意**：<b color='red'>**不要混淆`RUN`使用`CMD`。`RUN`实际上运行一个命令并提交结果; `CMD`在构建时不执行任何操作，但指定图像的预期命令**。</b>

### MAINTAINER

```dockerfile
MAINTAINER <name>
```

`MAINTAINER`指令设置生成图像的*作者*字段。`LABEL`指令是一个更加灵活的版本，应该使用它，因为它可以设置需要的任何元数据，并且可以很容易地查看，例如`docker inspect`。要设置与`MAINTAINER`可以使用的字段对应的标签 ：

```
LABEL maintainer="SvenDowideit@home.org.au"
```

这将从`docker inspect`其他标签中可见。

### EXPOSE

---

[EXPOSE指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#expose)

该`EXPOSE`指令指示容器侦听连接的端口。因此，应该为应用程序使用通用的传统端口。例如，包含`Apache Web`服务器的镜像将使用`EXPOSE 80`，而包含`MongoDB`的镜像将使用`EXPOSE 27017`等等。

对于外部访问，用户可以执行`docker run`一个标志，指示如何将指定端口映射到他们选择的端口。对于容器链接，Docker为从收件人容器返回到源路径（即，`MYSQL_PORT_3306_TCP`）提供环境变量。

```dockerfile
EXPOSE <port> [<port>/<protocol>...]
```

`EXPOSE`**指令通知Docker容器在运行时侦听指定的网络端口。可以指定端口是侦听TCP还是UDP，如果未指定协议，则默认为TCP。**

该`EXPOSE`指令并不实际发布该端口，它用作构建映像的对象和运行容器的对象之间的文档类型，关于哪些端口打算发布。要在运行容器时发布实际端口，请使用此`-p`标志`docker run` 发布和映射一个或多个端口，或者使用`-P`标志发布所有公开的端口并将它们映射到高阶端口。

默认情况下，假设`EXPOSE`TCP。你也可以指定UDP：

```dockerfile
EXPOSE 80/udp
```

要在TCP和UDP上公开，请包含两行：

```dockerfile
EXPOSE 80/tcp
EXPOSE 80/udp
```

在这种情况下，如果使用`-P`与`docker run`，该端口将被一次TCP和一次UDP曝光。请记住，`-P`在主机上使用短暂的高阶主机端口，因此TCP和UDP的端口不会相同。

`EXPOSE`无论如何设置，都可以使用该`-p`标志在运行时覆盖它们。例如

```dockerfile
docker run -p 80:80/tcp -p 80:80/udp ...
```

要在主机系统上设置端口重定向，请参阅[使用-P标志](https://docs.docker.com/engine/reference/run/#expose-incoming-ports)。该`docker network`命令支持为容器之间的通信创建网络，而无需公开或发布特定的端口，因为连接到网络的容器可以通过任何端口相互通信。有关详细信息，请参阅[此功能](https://docs.docker.com/engine/userguide/networking/)的 [概述](https://docs.docker.com/engine/userguide/networking/)）。

### ENV

---

[Dock指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#env)

为了使新软件更容易运行，可以使用`ENV`更新容器安装软件的 `PATH`环境变量。例如，`ENV PATH /usr/local/nginx/bin:$PATH`确保`CMD [“nginx”]` 正常工作。

`ENV`指令对于提供特定于你希望容器化的服务所需的环境变量也很有用，例如`Postgres's`  `PGDATA`。

```
ENV <key> <value>
ENV <key>=<value> ...
```

`ENV`指令将环境变量`<key>`设置为值 `<value>`。这个值将在构建阶段的所有后续指令的环境中，并且可以在许多[内联](https://docs.docker.com/engine/reference/builder/#environment-replacement)中被[替换](https://docs.docker.com/engine/reference/builder/#environment-replacement)。

`ENV`指令有两种形式：<br/>第一种形式是`ENV <key> <value>`，将一个变量设置为一个值。第一个空格后的整个字符串将被视为`<value>`包括空格字符。该值将被解释为其他环境变量，因此引号字符未被转义，则它们将被删除。<br/>第二种形式`ENV <key>=<value> ...`允许一次设置多个变量。请注意，第二种形式在语法中使用等号（=），而第一种形式不使用。与命令行解析一样，引号和反斜杠可用于包含值中的空格。

例如：

```dockerfile
ENV myName="John Doe" myDog=Rex\ The\ Dog \
    myCat=fluffy
```

```dockerfile
ENV myName John Doe
ENV myDog Rex The Dog
ENV myCat fluffy
```

将在最终图像中产生相同的结果。

`ENV`从结果图像运行容器时，使用的环境变量将保持不变。可以使用`docker inspect`查看这些值，并使用`docker run --env <key>=<value>`进行更改。

> **注意**：环境持久性可能会导致意想不到的副作用。<br/>例如，设置`ENV DEBIAN_FRONTEND noninteractive`可能会将apt-get用户混淆在基于Debian的映像上。要为单个命令设置一个值，请使用 `RUN <key>=<value> <command>`。

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

---

- [Dock指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#add)
- [适用于COPY指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#copy)

虽然`ADD`和`COPY`在功能上类似，**一般说来`COPY` 是优选的，这是因为它比`ADD`更透明**。`COPY`仅**支持将本地文件复制到容器中**，同时`ADD`还具有一些不明显的功能（如仅限本地的`tar`提取和远程`URL`支持）。因此，**最好的用`ADD`是将本地tar文件自动提取到镜像中**，如`ADD rootfs.tar.xz /`。

如果有多个`Dockerfile`步骤使用来自上下文的不同文件，那么`COPY`它们是单独的，而不是一次使用全部。这可确保**每个步骤的构建缓存**仅在特定所需文件发生更改时才会失效（强制重新运行该步骤）。

例如：

```dockerfile
COPY requirements.txt /tmp/
RUN pip install --requirement /tmp/requirements.txt
COPY . /tmp/
```

导致`RUN`步骤中缓存失效的次数少于放弃 `COPY . /tmp/`之前的缓存失效次数。

由于镜像大小很重要，因此`ADD`强烈建议**不要使用从远程`URL`获取软件包**; 你应该使用`curl`或`wget`代替。这样，可以**删除解压缩后不再需要的文件**，并且不必在镜像中添加其他图层。例如，应该避免执行以下操作：

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



#### ADD

ADD 有两种形式：

- `ADD [--chown=<user>:<group>] <src>... <dest>`
- `ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]` （此形式对于包含空格的路径是必需的）

> **注意**：`--chown`仅在用于构建Linux容器的Dockerfiles上受支持，并且不适用于Windows容器。由于用户和组所有权概念不能在Linux和Windows之间进行转换，因此使用`/etc/passwd`和`/etc/group`将用户名和组名转换为ID会限制此功能仅适用于基于Linux操作系统的容器。

`ADD`指令从`<src>`中复制新文件、目录或远程文件URL并将它们添加到路径`<dest>`中图像的文件系统中。

可以指定多个`<src>`资源，但如果它们是文件或目录，则它们的路径将被解释为相对于构建的上下文来源。

每个`<src>`可能包含通配符，匹配将使用Go的 [filepath.Match](http://golang.org/pkg/path/filepath#Match)规则完成。例如：

```dockerfile
ADD hom* /mydir/        # adds all files starting with "hom"
ADD hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"
```

`<dest>`是绝对路径或相对`WORKDIR`的路径，源将被复制到目标容器中。

```dockerfile
ADD test relativeDir/          # adds "test" to `WORKDIR`/relativeDir/
ADD test /absoluteDir/         # adds "test" to /absoluteDir/
```

添加包含特殊字符（例如`[` 和`]`）的文件或目录时，需要按照Golang规则转义这些路径，以防止它们被视为匹配模式。例如，要添加一个名为的文件`arr[0].txt`，请参考以下内容;

```dockerfile
ADD arr[[]0].txt /mydir/    # copy a file named "arr[0].txt" to /mydir/
```

除非可选`--chown`标志指定给定的用户名、组名或`UID/GID`组合来请求所添加内容的特定所有权，否则所有新文件和目录均使用`UID/GID`创建。该`--chown`标志的格式允许使用用户名和组名字符串，或以任意组合形式直接使用整数`UID/GID`。提供没有组名的用户名或没有GID的UID将使用与GID相同的数字UID。如果提供了用户名或组名，容器的根文件系统 `/etc/passwd`和`/etc/group`文件将分别用于执行从名称到整数UID或GID的转换。以下示例显示`--chown`标志的有效定义：

```dockerfile
ADD --chown=55:mygroup files* /somedir/
ADD --chown=bin files* /somedir/
ADD --chown=1 files* /somedir/
ADD --chown=10:11 files* /somedir/
```

如果容器根文件系统不包含任何文件`/etc/passwd`或 `/etc/group`文件，并且在该`--chown` 标志中使用了用户名或组名，则构建`ADD`操作将失败。使用数字标识不需要查找，也不依赖于容器根文件系统内容。

在`<src>`远程文件URL 的情况下，目标将具有600的权限。如果正在检索的远程文件具有HTTP `Last-Modified`标头，则将使用来自该标头的时间戳来设置`mtime`目标文件上的时间戳。然而像`ADD`期间处理的任何其他文件一样，它`mtime`不会被包含在确定文件是否已经改变并且应该更新缓存中。

> **注意**：如果通过传递`Dockerfile` `STDIN`（`docker build - < somefile`）来构建，则没有构建上下文，因此`Dockerfile` 只能包含基于URL的`ADD`指令。也可以通过`STDIN`传递一个压缩存档：（`docker build - < archive.tar.gz`），`Dockerfile`存档的根目录和存档的其余部分将用作构建的上下文。
> 如果网址文件都使用认证保护，将需要使用`RUN wget`，`RUN curl`或使用其它工具从容器内的`ADD`指令不支持验证。
> 如果`<src>`内容已更改，则第一次遇到`ADD`的指令将使Dockerfile中所有后续指令的缓存无效。这包括使`RUN`指令的缓存无效。

`ADD` **遵守以下规则**：

- 该`<src>`路径必须位于构建的上下文中； 你不能`ADD ../something /something`，因为 `docker build`的第一步是将上下文目录（和子目录）发送到docker守护进程。
- 如果`<src>`是URL并且`<dest>`不以尾部斜线结尾，则从URL下载文件将被复制到`<dest>`中。
- 如果`<src>`是一个URL并且`<dest>`以结尾的斜线结尾，那么文件名将从url中推断出来，并将该文件下载到该URL`<dest>/<filename>`。例如，`ADD http://example.com/foobar /`将创建该文件`/foobar`。该URL必须有一个不平凡的路径，以便在这种情况下可以找到适当的文件名（`http://example.com` 不起作用）。
- 如果`<src>`是目录，则复制目录的全部内容，包括文件系统元数据。**目录本身不被复制，只是它的内容。**


- 如果`<src>`是以可识别的压缩格式（`identity，gzip，bzip2、xz`）的*本地* tar归档文件，则将其解压缩为目录。来自*远程* URL的资源**不被**解压缩。当一个目录被复制或解包时，它的行为`tar -x`与结果相同，结果是：

  1. 无论在目的地路径和目的地存在什么
  2. 源代码树的内容，在逐个文件的基础上解决了冲突，支持“2.”。

  > **注意**：文件是否被识别为可识别的压缩格式完全是基于文件的内容而不是文件的名称。例如，如果空文件碰巧以`.tar.gz`结尾，则不会将其识别为压缩文件，并且**不会**生成任何类型的解压缩错误消息，而是将该文件简单地复制到目标。

- 如果`<src>`是其他类型的文件，则将其与其元数据一起单独复制。在这种情况下，如果`<dest>`以结尾斜线结尾`/`，它将被视为一个目录，其内容`<src>`将被写入`<dest>/base(<src>)`。

- 如果`<src>`指定了多个资源（直接或由于使用通配符），则`<dest>`必须是目录，并且必须以斜线结尾`/`。

- 如果`<dest>`不以尾部斜线结尾，则将其视为常规文件，`<src>`并将写入内容到`<dest>`。

- 如果`<dest>`不存在，则会在其路径中创建所有缺少的目录。



#### COPY

COPY有两种形式：

- `COPY [--chown=<user>:<group>] <src>... <dest>`
- `COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]` （此形式对于包含空格的路径是必需的）

> **注意**：`--chown`仅在用于构建Linux容器的Dockerfiles上受支持，并且不适用于Windows容器。由于用户和组所有权概念不能在Linux和Windows之间进行转换，因此使用`/etc/passwd`和`/etc/group`将用户名和组名转换为ID会限制此功能仅适用于基于Linux操作系统的容器。

`COPY`指令从`<src>` 复制新文件或目录并将其添加到路径`<dest>`中容器的文件系统。

`COPY`可以指定多个`<src>`资源，但文件和目录的路径将被解释为与构建的上下文来源。

每个`<src>`可能包含通配符，匹配将使用Go的 [filepath.Match](http://golang.org/pkg/path/filepath#Match)规则完成。例如：

```dockerfile
COPY hom* /mydir/        # adds all files starting with "hom"
COPY hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"
```

`<dest>`是一个绝对路径或相对`WORKDIR`的路径，源将被复制到目标容器中。

```dockerfile
COPY test relativeDir/   # adds "test" to `WORKDIR`/relativeDir/
COPY test /absoluteDir/  # adds "test" to /absoluteDir/
```

在复制包含特殊字符（如`[` 和`]`）的文件或目录时，您需要按照Golang规则转义这些路径，以防止它们被视为匹配模式。例如，要复制一个名为的文件`arr[0].txt`，请参考以下内容;

```dockerfile
COPY arr[[]0].txt /mydir/    # copy a file named "arr[0].txt" to /mydir/
```

除非可选`--chown`标志指定给定的用户名、组名或`UID/GID`组合来请求所添加内容的特定所有权，否则所有新文件和目录均使用`UID/GID`创建。该`--chown`标志的格式允许使用用户名和组名字符串，或以任意组合形式直接使用整数`UID/GID`。提供没有组名的用户名或没有GID的UID将使用与GID相同的数字UID。如果提供了用户名或组名，容器的根文件系统 `/etc/passwd`和`/etc/group`文件将分别用于执行从名称到整数`UID/GID`的转换。以下示例显示`--chown`标志的有效定义：

```dockerfile
COPY --chown=55:mygroup files* /somedir/
COPY --chown=bin files* /somedir/
COPY --chown=1 files* /somedir/
COPY --chown=10:11 files* /somedir/
```

如果容器根目录文件系统不包含任何文件`/etc/passwd`或 `/etc/group`文件，并且在该`--chown` 标志中使用了用户名或组名，则构建`COPY`操作将失败。**使用数字标识不需要查找，也不依赖于容器根文件系统内容**。

> **注意**：如果使用STDIN（`docker build - < somefile`）构建，则没有构建上下文，因此`COPY`无法使用。

可以选择`COPY`接受一个选项`--from=<name|index>`，该选项可用于将源位置设置为之前的构建阶段（使用创建的`FROM .. AS <name>`），这将用于代替用户发送的构建上下文。该选项还接受为以`FROM`指令开始的所有先前构建阶段分配的**数字索引**。如果**无法找到具有指定名称的构建阶段**，则尝试使用具有**相同名称的图像**。

`COPY` 遵守以下规则：

- 该`<src>`路径必须是内部*语境*的构建; 你不能`COPY ../something /something`，因为a的第一步 `docker build`是将上下文目录（和子目录）发送到docker守护进程。
- 如果`<src>`是目录，则复制目录的全部内容，包括文件系统元数据。**目录本身不被复制，只是它的内容。**


- 如果`<src>`是其他类型的文件，则将其与其元数据一起单独复制。在这种情况下，如果`<dest>`以结尾斜线结尾`/`，它将被视为一个目录，其内容`<src>`将被写入`<dest>/base(<src>)`。
- 如果`<src>`指定了多个资源（直接或由于使用通配符），则`<dest>`必须是目录，并且必须以斜线结尾`/`。
- 如果`<dest>`不以尾部斜线结尾，则将其视为常规文件，`<src>`并将写入其内容`<dest>`。
- 如果`<dest>`不存在，则会在其路径中创建所有缺少的目录。



### ENTRYPOINT

---

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

例如，[Postgres官方镜像](https://hub.docker.com/_/postgres/) 使用以下脚本作为它的`ENTRYPOINT`：

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

---

[VOLUME指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#volume)

`VOLUME`指令应该用于公开任何数据库存储区域，配置存储或由docker容器创建的文件/文件夹。强烈建议将镜像的任何可变和用户可维修部分的使用`VOLUME`配置。

### USER

---

[用户指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#user)

如果服务可以在没有权限的情况下运行，则使用`USER`更改为非root用户。首先通过类似的方式在`dockerfile`中创建用户和组`RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres`。

> **注意**：镜像中的用户和组会得到非确定性的UID/GID，因为无论镜像重建如何，都会分配“下一个”UID/GID。所以，如果这很关键，你应该分配一个明确的UID/GID。

> **注意**：由于 `Go archive / tar`包处理松散文件时存在一个[未解决的错误](https://github.com/golang/go/issues/13548)，尝试在Docker容器内创建具有足够大`UID`的用户可能导致磁盘耗尽，因为`/var/log/faillog`容器层中充满了NUL（\ 0 ）字符。将该`--no-log-init `选项传递给`useradd`可以解决此问题。`Debian/Ubuntu` `adduser`包不支持该`--no-log-init`标志，应该避免。

避免安装或使用，因为`sudo`它具有可能导致问题的不可预测的TTY和信号转发行为。如果绝对需要类似于的功能`sudo`，例如初始化守护程序`root`但将其作为非运行`root`），请考虑使用 [“gosu”](https://github.com/tianon/gosu)。

最后，要减少层次和复杂性，请避免`USER`频繁来回切换。

### WORKDIR

---

[用于WORKDIR指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#workdir)

为了清晰和可靠，应该**始终使用绝对路径 `WORKDIR`**。此外应该使用`WORKDIR`而不是`RUN cd … && do-something`繁琐的命令，这是很难阅读和维护。

### ONBUILD

---

[ONBUILD指令的Dockerfile参考](https://docs.docker.com/engine/reference/builder/#onbuild)

在当前的`dockerfile`构建完成后执行`ONBUILD`命令。`ONBUILD`在从**当前镜像派生的任何子镜像中执行**。把这个`ONBUILD`命令看作父`Dockerfile`给子`Dockerfile`的一条指令。

Docker构建在子`Dockerfile`文件中的任何命令之前执行`ONBUILD`命令。

`ONBUILD`对于将从给定镜像构建的镜像非常有用。例如，可以使用`ONBUILD`语言编排镜像来构建用该语言编写的任意用户软件 `Dockerfile`，如在[Ruby的`ONBUILD`变体中所](https://github.com/docker-library/ruby/blob/master/2.4/jessie/onbuild/Dockerfile)看到[的](https://github.com/docker-library/ruby/blob/master/2.4/jessie/onbuild/Dockerfile)。

`ONBUILD`建立的镜像应该得到一个单独的标签，例如： `ruby:1.9-onbuild`或`ruby:2.0-onbuild`。

`ADD`或`COPY`时要小心`ONBUILD`。如果新构建的上下文缺少正在添加的资源，那么“构建”镜像就会发生灾难性的失败。按照上面的建议添加单独的标签，通过允许`Dockerfile`做出选择来帮助缓解这种情况。

# 创建一个基础镜像

大多数`Dockerfiles`从父镜像开始。如果需要完全控制镜像的内容，则可能需要创建基础镜像。以下是区别：

- 一个[父镜像](https://docs.docker.com/glossary/?term=parent%20image)是你的镜像的基础镜像。它`FROM`指向`Dockerfile`中指令的内容。`Dockerfile`中的每个后续声明都会修改此父镜像。大多数`Dockerfiles`从父镜像开始，而不是基础镜像。但是，这些术语有时可以相互使用。
- 基础镜像在Dockerfile中要么不具有`FROM`，或具有`FROM scratch`。

下面展示了创建基础镜像的几种方法。具体的过程将在很大程度上取决于你想打包的Linux发行版。我们在下面有一些例子，并提交请求拉取以提供新的镜像。

## 使用tar创建一个完整的镜像


一般来说，从运行想要打包为父镜像的发行版的工作机器开始，尽管对于像`Debian`的[Debootstrap](https://wiki.debian.org/Debootstrap)这样的工具[来说](https://wiki.debian.org/Debootstrap)，你也可以使用它来构建`Ubuntu` 镜像，但这不是必需的 。

它可以像创建`Ubuntu`父镜像一样简单：

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

## 使用scratch创建一个简单的父镜像


可以使用Docker的保留最小镜像`scratch`作为构建容器的起点。使用`scratch`镜像构建过程中，希望下一个命令`Dockerfile`成为镜像中的第一个文件系统层。

当`scratch`出现在Docker存储库中时，无法将其拉出、运行或标记具有该名称的任何镜像。相反，你可以参考你的`Dockerfile`。例如，要使用`scratch`以下命令创建最小容器 ：

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

要运行新镜像，请使用以下`docker run`命令：

```
docker run --rm hello
```

本示例将创建教程中使用的hello-world镜像。如果你想测试它，你可以克隆 [镜像回购](https://github.com/docker-library/hello-world)。



# 多阶段构建

多阶段构建是一项新功能，需要守护程序和客户端上的`Docker 17.05`或更高版本。多阶段构建对于那些努力优化Dockerfiles的人来说非常有用，同时让他们易于阅读和维护。

## 不使用多阶段构建


关于构建镜像最具挑战性的事情之一是**保持镜像的大小**。Dockerfile中的**每条指令都会为镜像添加一个图层**，并且需要记住在**移动到下一图层之前清理不需要的任何组件**。为了编写一个非常高效的Dockerfile，传统上需要使用shell技巧和其他逻辑来尽可能地减小层次，并确保每层都具有它需要的来自前一层的组件，而不是其他任何东西。

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

请注意，此示例还使用`RUN`Bash `&&`运算符人为地压缩两个命令，以避免在镜像中创建额外的图层。这很容易失败并且很难维护。例如，插入另一个命令并忘记继续使用该`\`字符行很容易。

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

它是如何工作的？第二`FROM`条指令以`alpine:latest`镜像为基础开始新的构建阶段。**该`COPY --from=0`行只将前一阶段构建的组件复制到这个新阶段**。Go SDK和任何中间组件都被留下，并未保存在最终镜像中。

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


当构建镜像时，不一定需要构建包括每个阶段的整个Dockerfile。可以指定目标构建阶段。以下命令假定正在使用`previous`，`Dockerfile`但在名为的阶段停止`builder`：

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
  + 直接下载软件包](https://pypi.python.org/pypi/docker/)。
  + 提取它并更改为提取的目录，
  + 运行`python setup.py install`。

[阅读完整的Docker引擎Python SDK参考](https://docker-py.readthedocs.io/)。



## API 参考


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



