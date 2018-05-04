# Docker 应用数据管理

# 概述

默认情况下，容器内创建的所有文件都**存储在可写容器层**上。这意味着：

- **复杂性高**：当容器**不再运行**时，数据**不会持续存在**，并且如果另一个进程需要数据，则很难从容器中获取数据。
- **移植性差**：容器的可写层紧密耦合容器运行的主机。无法轻松地将数据移到其他地方。
- **性能低下**：写入容器的可写层需要 [存储驱动程序](https://docs.docker.com/storage/storagedriver/)来管理文件系统。存储驱动程序使用Linux内核提供联合文件系统。与使用直接写入主机文件系统的**数据卷**相比，这种额外的抽象性能会**降低性能** 。

Docker有两个容器选项（**volumes** 和 **bind mounts**）来将文件存储在主机中，这样即使在容器停止之后这些文件也会被保留：**卷**和 **绑定挂载**。如果你在Linux上运行Docker，你也可以使用**tmpfs挂载**。

## 选择正确的挂载类型

无论选择使用哪种挂载类型，数据在容器内看起来都是相同的。它被公开为容器文件系统中的目录或单个文件。一个简单的方法来看出卷之间的差异，绑定挂载和`tmpfs` 挂载是考虑数据在Docker主机上的位置。

![Docker类型以及它们在Docker主机上的位置](https://docs.docker.com/storage/images/types-of-mounts.png)

- **volumes 卷** 存储在**由Docker管理**的主机文件系统的一部分中（在Linux上`/var/lib/docker/volumes/`）。非Docker进程不应该修改这部分文件系统。**卷是在Docker中保留数据的最佳方式**。
- **Bind mounts 绑定挂载**可以存储在主机系统的**任何位置**。他们甚至可能是重要的系统文件或目录。Docker主机或Docker容器上的非Docker进程可以**随时修改**它们。
- **tmpfs挂载**仅存储在**主机系统的内存**中，而**不会写入主机系统的文件系统**。

### 更多细节介绍

- **volume **：由**Docker创建和管理**。可以使用`docker volume create`命令显式创建一个卷，或者在创建容器或服务期间Docker可以创建一个卷。

  在创建卷时，它将存储在Docker**主机的目录**中。将卷装入容器时，**此目录是装入容器的内容**。这与绑定挂载的工作方式**类似**，区别在于**卷由Docker管理**，并与主机的核心功能隔离。

  给定的卷可以同时安装到多个容器中。当没有正在运行的容器在使用卷时，卷**仍然可用**，并且**不会自动删除**。可以使用`docker volume prune`删除未使用的卷。

  在安装卷时，可能会**命名**或**匿名**。首次将匿名卷挂载到容器时，匿名卷没有被赋予明确的名称，因此Docker会为它们提供一个**随机**名称，该名称在给定的Docker主机中保证是**唯一**的。除名称外，命名和匿名卷的行为方式相同。

  卷还支持使用**卷驱动程序**，这些**卷驱动程序**可让数据存储在**远程主机**或**云提供程序**中，以及其他可能性。

- **Bind mounts**：自Docker早期开始提供。与卷相比，绑定挂载具有有限的功能。当使用绑定挂载时，**主机上的文件或目录被挂载到容器**中。文件或目录由主机上的完整路径引用。该文件或目录**不需要已经存在**于Docker主机上。如果它尚不存在，它会根据需求创建。绑定挂载非常**高效**，但它们**依赖于具有特定目录结构的主机的文件**。如果正在开发新的Docker应用程序，请考虑使用**命名卷**。不能使用Docker CLI命令直接管理绑定挂载。

  > **绑定挂载允许访问敏感文件**
  >
  > 无论好坏，使用绑定挂载的一个副作用是可以通过**容器中**运行的进程更改**主机**文件 ，包括创建，修改或删除重要的系统文件或目录。这是一个强大的能力，可能会对**安全**产生影响，包括影响主机系统上的非Docker进程。

- **tmpfs mounts**：`tmpfs`在Docker**主机上或容器**内，挂载不会永久保存在磁盘上。它可以**在容器的生命周期内由容器使用**，以存储非**持久状态或敏感信息**。例如，在内部集群服务使用`tmpfs挂载`将[私密信息](https://docs.docker.com/engine/swarm/secrets/)装入服务的容器中。

绑定挂载和卷都可以使用`-v`或`--volume`选项挂载到容器中，但每种语法的语法稍有不同。对于`tmpfs`挂载，可以使用`--tmpfs`选项。但是在Docker 17.06及更高版本中，我们建议对容器和服务使用`--mount`选项，绑定挂载、卷或tmpfs挂载，因为语法更清晰明确。

## `volume`的应用场景

卷是在Docker容器和服务中保持数据的**首选方式**。卷的一些应用场景包括：

- 在多个运行容器之间**共享数据**。如果没有显示的创建它，则会在第一次装入容器时创建卷。当**容器停止或被移除**时，**卷仍然存在**。多个容器可以**同时安装相同**的卷，无论是**读写**还是**只读**。仅当显示的删除卷时才会删除卷。
- 当Docker主机不能保证具有给定的目录或文件结构时。卷帮助你将Docker主机的**配置与容器运行时分离**。
- 当你想要将容器的数据存储在**远程主机或云提供商**上而不是本地时。
- 当你需要备份，还原或将数据从一台Docker主机**迁移**到另一台时，卷是更好的选择。你可以使用卷停止容器，然后备份卷的目录（如`/var/lib/docker/volumes/<volume-name>`）。

## `mount`的应用场景

一般来说，你应该尽量使用卷。绑定安装适用于以下类型的应用场景：

- 从**主机共享配置文件到容器**。这是默认情况下，通过`/etc/resolv.conf`从主机挂载到每个容器，Docker如何为容器提供DNS解析 。

- **共享源代码**或在Docker主机上的**开发环境与容器之间构建组件**。例如，你可以将Maven 的`target/` 目录挂载到容器中，并且每次在Docker主机上构建Maven项目时，容器都可以访问重建的构建组件。

  如果你以这种方式使用Docker进行开发，你的生产将**直接将生产就绪组件Dockerfile复制到镜像**中，而不是依赖于绑定安装。

- 当Docker主机的文件或目录结构保证与容器所需的绑定挂载一致时。

## `tmpfs` 的应用场景

如果不希望数据在主机上或容器内持久存在，最适合使用`tmpfs`挂载。这可能出于**安全**原因，或者在应用程序需要编写大量**非持久状态数据**时保护容器的**性能**。

## 有关使用挂载或卷的注意事项

如果你使用绑定挂载或卷，请记住以下事项：

- 如果将**空卷**挂载到**存在文件或目录**的容器中的目录中，则会将这些文件或目录**传播（复制）到卷**中。同样，如果你启动容器并指定一个尚不存在的卷，则会为你**创建**一个空卷。这是**预先填充**其他容器需要的数据的好方法。
- 如果你将**绑定挂载或非空卷挂载**到某个文件或目录存在的容器中的某个目录中，则这些文件或目录会被**挂载遮盖**，就像你将文件保存到Linux主机`/mnt`上一样，然后挂载USB驱动器进入`/mnt`。`/mnt`在USB驱动器卸载之前，内容将被USB驱动器的内容遮盖。隐藏的文件不会被删除或更改，但在装入绑定挂载或卷时**不可访问**。

# volume 基本命令

与绑定挂载不同，你可以创建和管理任何容器范围之外的卷。

## create 创建

### 基本选项

```shell
--driver , -d	#默认值：local，指定卷驱动程序名称
--label		    #设置卷的元数据
--name		    #指定卷名称
--opt , -o		#设置驱动程序特定选项
```

创建容器可以使用和存储数据的新卷。如果未指定名称，则Docker会生成一个随机名称。

### 创建卷

```sh
$ docker volume create my-vol
```

### 使用卷

创建一个卷，然后配置容器以使用它：

```sh
$ docker volume create hello
hello
$ docker run -d -v hello:/world busybox ls /world
```

挂载目录`/world`是在容器的中创建的。Docker不支持容器内的挂载点的相对路径。

多个容器可以在同一时间段内使用相同的挂载卷。如果两个容器需要访问共享数据，这很有用。例如，如果一个容器写入，另一个容器读取数据。

**卷的名字在设备中必须是唯一的**。这意味着你不能在两个不同的设备中使用相同的卷名称。如果你尝试此操作`docker`将返回一个错误：

```sh
A volume named  "hello"  already exists with the "some-other" driver. Choose a different volume name.
```

如果你指定当前设备程序中已使用的卷名称，则Docker会假定你想要重新使用现有卷并且不会返回错误。

### 驱动程序特定的选项

某些卷驱动程序可能会采用选项来创建自定义卷。使用 `-o`或`--opt`选项来传递驱动程序选项：

```sh
$ docker volume create --driver fake \
    --opt tardis=blue \
    --opt timey=wimey \
    foo
    
Error response from daemon: create foo: error looking up volume plugin fake: plugin "fake" not found    
```

这些选项**直接传递给卷**驱动程序。不同卷驱动程序的选项可能会做不同的事情（或根本没有）。

Windows上的内置`local`驱动程序不支持任何选项。

Linux上的内置`local`驱动程序接受类似于linux `mount`命令的选项 。你可以通过多次传递`--opt`选项来提供多个选项。某些`mount`选项（如`o`选项）可以使用逗号分隔的选项列表。可在[此处](http://man7.org/linux/man-pages/man8/mount.8.html)找到完整的可用挂载选项列表。

例如，下面创建一个名为`foo`的`tmpfs`卷，其大小为`100`兆字节，`uid`为`1000`。

```sh
$ docker volume create --driver local \
    --opt type=tmpfs \
    --opt device=tmpfs \
    --opt o=size=100m,uid=1000 \
    foo
```

另一个例子使用`btrfs`：

```sh
$ docker volume create --driver local \
    --opt type=btrfs \
    --opt device=/dev/sda2 \
    foo
```

另一个使用`nfs`从`192.168.1.1`开始以`rw`模式挂载`/path/to/dir`的例子：

```sh
$ docker volume create --driver local \
    --opt type=nfs \
    --opt o=addr=192.168.1.1,rw \
    --opt device=:/path/to/dir \
    foo
```

## ls 查看

### 基本选项

```shell
--filter , -f		#提供过滤器值（例如'dangling = true'）
--format			#使用Go模板的漂亮打印卷
--quiet , -q		#只显示卷名称
```

### 过滤

过滤选项（`-f`或`--filter`）格式为“key = value”。如果有多个过滤器，则传递多个选项（例如`--filter "foo=bar" --filter "bif=baz"`）

目前支持的过滤器是：

- `dangling`（布尔值 - true或false，0或1）
- `driver`  （卷驱动程序的名称）
- `label`（`label=<key>`或`label=<key>=<value>`）
- `name`（卷的名称）

#### dangling 是否引用

过滤器上没有任何容器引用的所有匹配卷

```sh
$ docker run -d  -v tyler:/tmpwork  busybox
f86a7dd02898067079c99ceacd810149060a70528eff3754d0b0f1a93bd0af18
$ docker volume ls -f dangling=true
DRIVER              VOLUME NAME
local               rosemary
```

#### driver 驱动

`driver`过滤器根据其驱动程序匹配卷。以下示例匹配使用该`local`驱动程序创建的卷：

```sh
$ docker volume ls -f driver=local

DRIVER              VOLUME NAME
local               rosemary
local               tyler
```

#### label 标签

标签过滤器根据单独存在标签或标签和值来匹配卷。

首先，我们来创建一些卷来说明这一点;

```sh
$ docker volume create the-doctor --label is-timelord=yes
the-doctor

$ docker volume create daleks --label is-timelord=no
daleks
```

以下示例过滤器将卷与`is-timelord`标签进行匹配，而不考虑值。

```sh
$ docker volume ls --filter label=is-timelord

DRIVER              VOLUME NAME
local               daleks
local               the-doctor
```

正如上面的例子演示的那样，带有`is-timelord=yes`和 `is-timelord=no`被返回的两个卷。

对两者`key` *和* `value`标签进行过滤，产生预期的结果：

```sh
$ docker volume ls --filter label=is-timelord=yes

DRIVER              VOLUME NAME
local               the-doctor
```

指定多个标签过滤器会产生**and**搜索，应该满足所有条件

```sh
$ docker volume ls --filter label=is-timelord=yes --filter label=is-timelord=no
DRIVER              VOLUME NAME
```

#### name 称

名称过滤器匹配全部或部分卷的名称。以下过滤器匹配所有包含该`rose`字符串的名称的卷。

```sh
$ docker volume ls -f name=rose

DRIVER              VOLUME NAME
local               rosemary
```

### 格式化

格式化选项（`--format`）使用Go模板输出卷。下面列出了Go模板的有效占位符：

| 占位符        | 描述                                                   |
| ------------- | ------------------------------------------------------ |
| `.Name`       | 卷名称                                                 |
| `.Driver`     | 卷驱动程序                                             |
| `.Scope`      | 卷范围（本地，全局）                                   |
| `.Mountpoint` | 主机上卷的安装点                                       |
| `.Labels`     | 分配给该卷的所有标签                                   |
| `.Label`      | 此卷的特定标签的值。例如`{{.Label "project.version"}}` |

使用`--format`选项时，`volume ls`命令将按照模板声明输出数据，或者在使用 `table`指令时也包含列标题。

下面的示例使用的模板没有报头，并输出 `Name`与`Driver`由所有卷冒号分隔的条目：

```sh
$ docker volume ls --format "{{.Name}}: {{.Driver}}"
vol1: local
vol2: local
vol3: local

$ docker volume ls --format "table {{.Name}}: {{.Driver}}"
vol1: local
vol2: local
vol3: local
```

## inspect 检查

```sh
$ docker volume inspect my-vol
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]

$ docker volume inspect --format '{{ .Mountpoint }}' my-vol

$ docker volume inspect foo --format '{{.Options}}'
map[o:size=100m,uid=100 type:tmpfs device:tmpfs]

$ docker volume inspect foo --format '{{json .Options}}'
{"device":"tmpfs","o":"size=100m,uid=100","type":"tmpfs"}

$ docker volume inspect foo --format='{{json .Options}}'
```

## rm 删除

删除一个或多个卷。你无法删除容器正在使用的卷。

```sh
$ docker volume rm my-vol
# 删除多个
$ docker volume rm hello e425b890f45
# 强制删除
$ docker volume rm -f hello e425b890f45
```

## prune 裁剪

删除所有未使用的本地卷。未使用的本地卷是那些没有被任何容器引用的卷

```shell
$ docker volume prune
# 强制删除，不提示输入
$ docker volume prune -f
## 过滤裁剪
$ docker volume prune -f --filter driver=local	
```


# Volume 的使用

卷是**持久化**由Docker容器生成和使用的数据的首选机制。虽然[绑定挂载](https://docs.docker.com/storage/bind-mounts/)依赖于**主机的目录结构**，但卷由Docker完全管理。与绑定挂载相比，卷有几个优点：

- **方便迁移**：与绑定挂载相比，卷更容易**备份或迁移**。
- **方便管理**：可以使用`Docker CLI`命令或`Docker API`管理卷。
- **可跨平台**：卷在Linux和Windows容器上均可使用。
- **安全共享**：卷可以在多个容器之间更安全地共享。
- **远程支持**：卷驱动程序允许你在远程主机或云提供程序上存储卷，加密卷的内容或添加其他功能。
- **预加载填充**：新卷的内容可以由容器预先填充。

另外，与容器的可写入层中的数据持久化相比，卷通常是更好的选择，因为使用卷**不会增加使用容器的体积大小**，并且容量的内容存在于给定容器的**生命周期之外**。

![Docker主机上的卷](https://docs.docker.com/storage/images/types-of-mounts-volume.png)

如果你的容器生成**非持久状态**数据，请考虑使用 [tmpfs挂载](https://docs.docker.com/storage/tmpfs/)以**避免将数据永久存储**在任何地方，并通过**避免写入容器的可写层来提高容器的性能**。

`volume`使用`rprivate`绑定传播，并且卷的传播不可配置。

## 选择`-v`或 `--mount` 选项

本来`-v`或`--volume`选项用于独立容器，而`--mount`选项用于集群服务。但是，从Docker 17.06开始，你也可以将`--mount`使用在独立容器。 `--mount`更明确和详细。最大的区别在于`-v` 语法将所有选项组合在一个字段中，而`--mount` 语法将它们分开。

> **提示**：新用户应使用`--mount`语法。有经验的用户可能更熟悉`-v`或`--volume`语法，但鼓励使用`--mount`语法，因为它更易于使用。

如果需要指定`volume`驱动程序选项，则必须使用`--mount`。

- `-v`或者`--volume`：由冒号（`:`）分隔的字段组成。这些字段必须按照正确的**顺序排列**，每个字段的含义并不明显。
  - 对于卷命名，**第一个字段是卷的名称**，并且在给定主机上是唯一的。对于匿名卷，第一个字段被省略。
  - 第二个字段是文件或目录在容器中的**安装路径**。
  - 第三个字段是**可选**的，并且是逗号分隔的选项列表，例如`ro`。这些选项在下面讨论。

- `--mount`：由多个键值对组成，以逗号分隔，每个键值由`<key> = <value>`元组组成。`--mount`语法比`-v`或`--volume`更冗长，但键的顺序并不重要，并且选项的值更易于理解。

  - `type`挂载类型，可以是[`bind`](https://docs.docker.com/storage/bind-mounts/)、`volume`、或 [`tmpfs`](https://docs.docker.com/storage/tmpfs/)。
  - `source`挂载来源。对于命名卷，这是卷的名称。对于匿名卷，该字段被省略。可能被指定为`source`或`src`。
  - `destination`挂载目标。文件或目录在容器中的安装路径。可以指定为`destination`，`dst`或`target`。
  - `readonly`是否只读选项。该选项（如果存在）将导致绑定挂载以[只读](https://docs.docker.com/storage/volumes/#use-a-read-only-volume)方式[挂载到容器中](https://docs.docker.com/storage/volumes/#use-a-read-only-volume)。
  - `volume-opt`选项可以多次指定，它采用由选项名称和值组成的键值对。

> CSV解析器转义
>
> 如果你的卷驱动程序接受逗号分隔列表作为选项，则必须从程序中转义该值。要转义 `volume-opt`，用双引号（`"`）括起来，用单引号（`'`）括住整个mount参数。
>
> 例如，`local`驱动程序接受挂载选项`o`作为参数中的逗号分隔列表。示例显示了转义列表的正确方法。
>
> ```sh
> $ docker service create \
>      --mount 'type=volume,src=<VOLUME-NAME>,dst=<CONTAINER-PATH>,volume-driver=local,volume-opt=type=nfs,volume-opt=device=<nfs-server>:<nfs-path>,"volume-opt=o=addr=<nfs-address>,vers=4,soft,timeo=180,bg,tcp,rw"'
>     --name myservice \
>     <IMAGE>
> ```

### `-v`与`--mount`的差异

与绑定挂载不同，卷的所有选项都可用于`--mount`和`-v`选项。将卷与服务一起使用时，仅支持`--mount`。

## 启动容器使用卷

如果你启动一个卷容量尚不存在的容器，Docker会为你创建卷。以下示例将卷`myvol2`挂载到容器中的`/app/`中。

### 运行容器关联卷

下面的`-v`和`--mount`示例产生相同的结果。除非在运行第一个容器后删除`devtest`容器和`myvol2`卷，否则无法运行它们。

```sh
# --mount 模式
$ docker run -d \
  --name devtest \
  --mount source=myvol2,target=/app \
  nginx:latest

# 删除刚才创建的容器
$ docker stop devtest && docker rm devtest
# -v 模式
$ docker run -d \
  --name devtest \
  -v myvol2:/app \
  nginx:latest
```

### 检查卷

使用`docker inspect devtest`来验证卷是否已正确创建和安装。查看挂载部分：

```sh
$ docker inspect devtest --format '{{json .Mounts}}'
[{
"Type":"volume",
"Name":"myvol2",
"Source":"/mnt/sda1/var/lib/docker/volumes/myvol2/_data",
"Destination":"/app",
"Driver":"local",
"Mode":"z",
"RW":true,
"Propagation":""
}]
```

这表明挂载是一个卷，它显示正确的源和目标，并且挂载是可读写的。

### 清理容器和卷

停止容器并移除卷。注意清除卷是一个单独的步骤。

```sh
$ docker container stop devtest
$ docker container rm devtest
$ docker volume rm myvol2
```

## 启动服务使用卷

当你启动服务并定义卷时，每个服务容器都使用自己的本地卷。如果你使用`local` 卷驱动程序，则任何容器都**不能共享**此数据，但某些卷驱动程序确实**支持共享存储**。`AWS`的Docker和`Azure`的Docker都使用`Cloudstor`插件支持**持久存储**。

以下示例启动一个具有四个副本的`nginx`服务，每个副本使用一个`myvol2`的本地卷。

```sh
$ docker service create -d \
  --replicas=4 \
  --name devtest-service \
  --mount source=myvol2,target=/app \
  nginx:latest
```

使用`docker service ps devtest-service`查看服务正在运行：

```sh
$ docker service ps devtest-service

ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
4d7oz1j85wwn        devtest-service.1   nginx:latest        moby                Running             Running 14 seconds ago   
```

删除停止所有任务的服务：

```sh
$ docker service rm devtest-service
```

**删除服务不会删除该服务创建的任何卷。删除卷是一个单独的步骤**。

### 服务的语法差异

`docker service create`命令**不支持`-v`或`--volume`选项**。将卷安装到服务的容器中时，**必须使用该`--mount` 选项**。

## 使用容器填充卷

如果你启动一个创建新卷的容器（如上所述），并且容器在要挂载的目录中具有文件或目录（如`/app/above`），则该**目录的内容将被复制到卷中**。然后容器安装并使用该卷，而使用该卷的**其他容器也可以访问预先填充的内容**。

为了说明这一点，启动一个`nginx`容器，并用容器`/usr/share/nginx/html`目录的内容 `nginx-vol`填充新卷，这是Nginx存储其默认HTML内容的地方。

```sh
# mount 模式
$ docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html \
  nginx:latest
# -v 模式  
$ docker run -d \
  --name=nginxtest \
  -v nginx-vol:/usr/share/nginx/html \
  nginx:latest  
```

运行这些示例之一后，运行以下命令来清理容器和卷。

```sh
$ docker container stop nginxtest
$ docker container rm nginxtest
$ docker volume rm nginx-vol
```

## 只读卷

对于某些开发应用程序，容器需要写入绑定挂载，以便将更改传播回Docker主机。在其他时候，容器只需要读取数据。请记住，多个容器可以挂载相同的卷，并且可以同时对它们中的某些容器进行**读写**挂载，对其他容器进行**只读**。

这个例子修改了上面的例子，但是通过在容器中的挂载点之后添加选项`ro`（默认为空，`readonly|ro`），将目录挂载为**只读卷**。如果存在多个选项，请用逗号分隔它们。

```sh
# mount 模式
$ docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html,readonly \
  nginx:latest

# -v 模式
$ docker run -d \
  --name=nginxtest \
  -v nginx-vol:/usr/share/nginx/html:ro \
  nginx:latest
```

使用`docker inspect nginxtest`验证挂载是否正确创建。寻找`Mounts`部分：

```sh
$ docker inspect nginxtest
"Mounts": [
    {
        "Type": "volume",
        "Name": "nginx-vol",
        "Source": "/var/lib/docker/volumes/nginx-vol/_data",
        "Destination": "/usr/share/nginx/html",
        "Driver": "local",
        "Mode": "",
        "RW": false,
        "Propagation": ""
    }
],
```

停止并移除容器，然后移除音量。去除体积是一个单独的步骤。

```sh
$ docker container stop nginxtest
$ docker container rm nginxtest
$ docker volume rm nginx-vol
```

## 在机器间共享数据

在构建容错应用程序时，可能需要配置同一服务的多个副本才能访问相同的文件。

![共享存储](https://docs.docker.com/storage/images/volumes-shared-storage.svg)

开发应用程序时有几种方法可以实现这一点。一种是向你的应用程序添加逻辑，以将文件存储在Amazon S3等云对象存储系统上。另一种方法是使用支持文件写入NFS或Amazon S3等外部存储系统的驱动程序来创建卷。

卷驱动程序允许你从应用程序逻辑中抽象出底层存储系统。例如，如果你的服务将卷与NFS驱动程序一起使用，则可以更新服务以使用不同的驱动程序，作为将数据存储在云中的示例，而无需更改应用程序逻辑。

### 使用卷驱动程序

当你使用`docker volume create`创建卷时，或者当你启动使用尚未创建的卷的容器时，可以指定卷驱动程序。以下示例首先在创建独立卷时以及在启动创建新卷的容器时使用`vieux/sshfs`卷驱动程序。

#### 初始设置

假定你有两个节点，其中第一个是Docker主机，可以使用SSH连接到第二个节点。在Docker主机上安装`vieux/sshfs`插件：

```sh
$ docker plugin install --grant-all-permissions vieux/sshfs
```

#### 使用卷驱动程序创建卷

下面指定一个SSH密码，如果两台主机配置了共享密钥，则可以省略该密码。每个卷驱动器可能有零个或多个可配置选项，每个选项都使用一个`-o`选项来指定。

```sh
$ docker volume create --driver vieux/sshfs \
  -o sshcmd=test@node2:/home/test \
  -o password=123456 \
  sshvolume
```

#### 启动一个容器使用卷驱动程序创建卷

指定SSH密码，但如果两台主机配置了共享密钥，则可以省略该密码。每个卷驱动可能有零个或多个可配置选项。**如果卷驱动程序要求你传递选项，则必须使用此`--mount`选项挂载卷，而不是`-v`。**

```sh
$ docker run -d \
  --name sshfs-container \
  --volume-driver vieux/sshfs \
  --mount src=sshvolume,target=/app,volume-opt=sshcmd=test@node2:/home/test,volume-opt=password=123456 \
  nginx:latest
```

# mount 的使用

绑定挂载从Docker早期开始就已经存在。与[卷](https://docs.docker.com/storage/volumes/)相比，绑定挂载具有有限的功能。当你使用绑定挂载时，**主机**上的文件或目录被挂载到容器中。文件或目录由主机上的完整路径或相对路径引用。相比之下，当你使用卷时，会在**主机上的Docker存储目录中创建一个新目录**，并且Docker会管理该目录的内容。

该文件或目录**不需要**已经存在于Docker主机上。如果它尚不存在，它会根据需求**创建**。绑定挂载非常高效，但它们依赖于具有特定目录结构的主机的文件系统。如果你正在开发新的Docker应用程序，请考虑使用[命名卷](https://docs.docker.com/storage/volumes/)。你**不能使用Docker CLI命令直接管理绑定挂载**。

![绑定在Docker主机上](https://docs.docker.com/storage/images/types-of-mounts-bind.png)

## 选择`-v`或`--mount`选项

本来`-v`或`--volume`选项用于独立容器，而`--mount`选项用于集群服务。但是，从Docker 17.06开始，你也可以将`--mount`使用在独立容器。 `--mount`更明确和详细。最大的区别在于`-v` 语法将所有选项组合在一个字段中，而`--mount` 语法将它们分开。

> **提示**：新用户应使用`--mount`语法。有经验的用户可能更熟悉`-v`或`--volume`语法，但鼓励使用`--mount`语法，因为它更易于使用。

- `-v`或者`--volume`：由冒号（`:`）分隔的字段组成。这些字段必须按照正确的**顺序排列**，每个字段的含义并不明显。
  - 在绑定挂载的情况下，第一个字段是**主机**上文件或目录的路径。
  - 第二个字段是文件或目录在容器中的安装路径。
  - 第三个字段是可选的，并且是用逗号分隔的选项，诸如列表`ro`，`consistent`，`delegated`，`cached`，`z`，和`Z`。这些选项在下面讨论。
- `--mount`：由多个键值对组成，以逗号分隔，每个键值由`<key> = <value>`元组组成。`--mount`语法比`-v`或`--volume`更冗长，但键的顺序并不重要，并且选项的值更易于理解。
  - `type`挂载类型，其可以是`bind`，`volume`，或`tmpfs`。本主题讨论绑定挂载，因此类型始终如此`bind`。
  - `source`挂载来源，对于绑定挂载，这是Docker守护程序主机上文件或目录的路径。可能被指定为`source`或 `src`。
  - `destination`挂载目标，值作为文件或目录是在容器中的安装路径。可以指定为`destination`，`dst`或`target`。
  - `readonly`读写选项（可选的）将导致绑定挂载以[只读](https://docs.docker.com/storage/bind-mounts/#use-a-read-only-bind-mount)方式[挂载到容器中](https://docs.docker.com/storage/bind-mounts/#use-a-read-only-bind-mount)。
  - `bind-propagation`绑定传播选项（可选），可以是一个`rprivate`，`private`，`rshared`，`shared`，`rslave`，`slave`。
  - [`consistency`](https://docs.docker.com/storage/bind-mounts/#configure-mount-consistency-for-macos)约束选项，如果存在，可以是一种`consistent`，`delegated`或`cached`。此设置仅适用于Docker for Mac，并在所有其他平台上被忽略。
  - `--mount`选项不支持`z`或`Z`修改selinux标签的选项。

### `-v`与`--mount` 的差异

由于`-v`和`--volume`选项长期以来一直是Docker的一部分，它们的行为不能改变。这意味着在`-v`和`--mount`之间有一种不同的行为。

如果你使用`-v`或`--volume`挂载Docker主机上尚不存在的文件或目录，请使用`-v`为你创建挂载点。**它始终创建为一个目录。**

如果你使用`--mount`绑定挂载docker主机上尚不存在的文件或目录，docker也**不会**自动为你创建它，但会产生一个**错误**。

## 启动容器使用挂载

考虑你有一个`source`目录的情况，并且当你构建源代码时，构建组件被保存到另一个目录`source/target/`中。你希望工件对容器`/app/`可用，并且你希望容器每次在开发主机上构建源时都可以访问新的构建。使用以下命令将`target/` 目录绑定到你的`/app/`容器中。从`source`目录内运行命令 ，`$(pwd)`命令将获取到Linux或者MacOS主机的当前工作目录。

```sh
# mount 模式
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  nginx:latest
  
# -v 模式
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app \
  nginx:latest
```

使用`docker inspect devtest`验证绑定挂载正确创建。寻找`Mounts`部分：

```sh
$ docker inspect devtest --format '{{json .Mounts}}'
"Mounts": [
    {
        "Type": "bind",
        "Source": "/tmp/source/target",
        "Destination": "/app",
        "Mode": "",
        "RW": true,
        "Propagation": "rprivate"
    }
],
```

这表明挂载是一个`bind`挂载，它显示了正确的源和目标，它显示挂载是可读写的，并且传播设置为`rprivate`。

停止容器：

```sh
$ docker container stop devtest
$ docker container rm devtest
```

## 使用非空目录上挂载

如果挂载到容器上的非空目录中，则该目录的现有内容将被**挂载隐藏**。例如，当你想要测试新版本的应用程序而无需构建新镜像时。

这是一个极端的例子，但是用主机上的`/usr/`目录替换容器目录`/tmp/`的内容。在大多数情况下，这会导致容器无法正常工作。

```sh
# --mount 模式
$ docker run -d \
  -it \
  --name broken-container \
  --mount type=bind,source=/tmp,target=/usr \
  nginx:latest
  
# -v 模式  
$ docker run -d \
  -it \
  --name broken-container \
  -v /tmp:/usr \
  nginx:latest
  
docker: Error response from daemon: oci runtime error: container_linux.go:262:
starting container process caused "exec: \"nginx\": executable file not found in $PATH".  
```

由于容器挂载的`/tmp`目录隐藏遮盖了`/usr`目录，导致很多文件无法找到，所以容器没有启动成功。

该容器已创建但未启动。删除容器：

```sh
$ docker container rm broken-container
```

## 只读挂载

对于某些开发应用程序，容器需要**写入**数据到挂载，以便将更改内容传播回Docker主机。在其他时候，容器只需要**读取权限**。

修改上面的示例，通过在容器中的挂载目标之后添加`ro`选项（默认为空，`readonly|ro`），将目录挂载设为只读挂载。如果存在多个选项，请用逗号分隔它们。

```sh
# --mount 模式
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app,readonly \
  nginx:latest
  
# -v 模式
docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app:ro \
  nginx:latest
```

使用`docker inspect devtest`验证绑定安装正确创建。寻找`Mounts`部分：

```sh
$ docker inspect devtest --format '{{json .Mounts}}'
"Mounts": [
    {
        "Type": "bind",
        "Source": "/tmp/source/target",
        "Destination": "/app",
        "Mode": "ro",
        "RW": false,
        "Propagation": "rprivate"
    }
],
```

停止和移除容器：

```sh
$ docker container stop devtest && docker container rm devtest
```

## 配置绑定传播

对于绑定挂载和卷，绑定传播默认为`rprivate`。它只能配置**绑定挂载**，并且只能在**Linux主机**上配置。绑定传播是一个高级话题，许多用户从不需要配置它。

绑定传播是指在给定的绑定挂载或命名卷中创建的挂载是否可以**传播到该挂载的副本**。考虑一个挂载点`/mnt`，它也挂载在`/tmp`上。传播设置控制`/on/tmp/a`上的挂载是否也可用于`/mnt/a`。每个传播设置都有一个递归对应点。在递归的情况下，考虑`/tmp/a`也被挂载为`/foo`。传播设置控制是否存在`/mnt/a`和/或`/tmp/a`。

| 传播设置   | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| `shared`   | 共享挂载，原始挂载的子挂载会**暴露给副本**挂载，并且副挂载的子挂载也会传播到原始挂载。 |
| `slave`    | 类似于**共享挂载**，但仅限于一个方向。如果原始挂载显示了一个子挂载，则副本挂载可以看到它。但是，如果副本挂载程序公开了子挂载，则原始挂载程序无法看到它。 |
| `private`  | 该挂载是**私有**的。其中的子挂载不会暴露给副本挂载，并且副挂载的子挂载不会暴露给原始挂载。 |
| `rshared`  | 与**共享相同**，但传播也扩展到嵌套在任何原始或副本装入点内的挂载点。 |
| `rslave`   | 与从属设备相同，但传播还扩展到嵌套在任何原始或副本安装点内的挂载点。 |
| `rprivate` | 默认值。与**私有**相同，这意味着原始或副本安装点内的任何位置的挂载点都不会沿任一方向传播。 |

在可以在挂载点上设置绑定传播之前，主机文件系统需要已经支持绑定传播。有关绑定传播的更多信息，请参阅[共享子树](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)的 [Linux内核文档](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)。以下示例将`target/`目录挂载到容器中两次，第二个挂载同时设置`ro`选项和`rslave`绑定传播选项。

```sh
# --mount 模式
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  --mount type=bind,source="$(pwd)"/target,target=/app2,readonly,bind-propagation=rslave \
  nginx:latest

# -v 模式
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app \
  -v "$(pwd)"/target:/app2:ro,rslave \
  nginx:latest
```

现在，如果你创建`/app/foo/`，`/app2/foo/`也存在。 

停止和移除容器：

```sh
$ docker container stop devtest && docker container rm devtest
```

## 配置selinux标签

如果你使用`selinux`，则可以添加`z`或`Z`选项来修改正在装入容器的**主机文件或目录**的selinux标签。这会影响主机本身的文件或目录，并可能导致Docker范围之外的问题后果。

- `z`选项指示绑定挂载内容在**多个容器之间共享**。
- `Z`选项指示绑定挂载内容是**私有的和未共享**的。

使用这些选项要**极端**谨慎。挂载系统目录（例如`/home`或`/usr`使用该`Z`选项）会使你的主机无法操作，你可能需要手动重新标记主机文件。

> **重要提示**：使用绑定挂载与服务时，selinux标签（`:Z`和`:z`）以及`:ro`被忽略。有关详细信息，请参阅[moby / moby＃32579](https://github.com/moby/moby/issues/32579)。

此示例设置`z`选项以指定**多个容器可以共享绑定挂载**的内容：

```sh
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app:z \
  nginx:latest
```

# tmpfs 的使用

[卷](https://docs.docker.com/storage/volumes/)和[挂载](https://docs.docker.com/storage/bind-mounts/)使你可以在主机和容器之间共享文件，即使在容器停止后也可以保留数据。

如果你在Linux上运行Docker，你有第三个选择：`tmpfs`挂载。当使用`tmpfs`挂载创建容器时，容器可以在**容器的可写层之外**创建文件。

与卷和挂载相反，`tmpfs`挂载是**暂时非持久**的，并且只保留在**主机内存**中。当容器停止时，`tmpfs`挂载被删除，写入的文件将**不会被保存**。

![Docker主机上的tmpfs](https://docs.docker.com/storage/images/types-of-mounts-tmpfs.png)

这对临时存储你不想保存在主机或容器可写层中的**敏感文件**很有用。

## tmpfs 挂载的限制

- 与卷和绑定挂载不同，不能`tmpfs`在容器之间**共享**挂载。
- 此功能仅在Linux上运行Docker时可用。

## 选择`--tmpfs `或 `--mount`选项

本来这个`--tmpfs`选项被用于独立的容器，并且`--mount`选项被用于集群服务。但是，从Docker 17.06开始，你也可以在独立容器使用`--mount`。 `--mount`更明确和详细，最大的区别是 `--tmpfs`选项**不支持任何可配置**的选项。

- **--tmpfs**：`tmpfs`在**不允许指定任何可配置**选项的情况下挂载，并且只能与独立容器一起使用。
- **--mount**：由多个键值对组成，由逗号分隔，每个由一个`<key>=<value>`元组组成。`--mount`语法比--tmpfs`更详细`：
  - `type`挂载类型，可以是[`bind`](https://docs.docker.com/storage/bind-mounts-md)，`volume`，或 [`tmpfs`](https://docs.docker.com/storage/tmpfs/)。
  - `destination`挂载目标，其中的路径`tmpfs`被挂载在容器中。可以指定为`destination`，`dst`或`target`。
  - `tmpfs-type`和`tmpfs-mode`选项。请参阅 [tmpfs选项](https://docs.docker.com/storage/tmpfs/#tmpfs-options)。

### `--tmpfs`与`--mount`的差异

- `--tmpfs`选项不允许指定任何可配置的选项。
- `--tmpfs`选项不能用于集群服务。必须使用`--mount`。

## 在容器中使用 `tmpfs` 挂载

在容器中使用`tmpfs`挂载，请使用`--tmpfs`选项，或使用带有`type=tmpfs`和`destination`选项的`--mount` 选项。以下示例在Nginx容器 `/app`中创建一个`tmpfs`挂载。

```sh
# --mount 模式
$ docker run -d \
  -it \
  --name tmptest \
  --mount type=tmpfs,destination=/app \
  nginx:latest
  
# --tmpfs 模式
$ docker run -d \
  -it \
  --name tmptest \
  --tmpfs /app \
  nginx:latest  
```

通过运行`docker container inspect tmptest`并查找该`Mounts`部分，验证该安装是否为挂载：

```sh
$ docker container inspect tmptest --format '{{json .HostConfig.Tmpfs}}'
"Tmpfs": {
    "/app": ""
},
```

移除容器：

```sh
$ docker container stop tmptest && docker container rm tmptest
```

## tmpfs选项

`tmpfs`挂载允许两个配置选项，这两者都不是必需的。如果需要指定这些选项，则必须使用`--mount`选项，因为`--tmpfs`选项不支持它们。

| 选项         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| `tmpfs-size` | tmpfs的大小，以字节为单位。无限制默认。                      |
| `tmpfs-mode` | tmpfs的八进制文件模式。例如，`700`或`0770`。默认为`1777`或公共可写。 |

以下示例将设置`tmpfs-mode`为`1770`，以便它在容器内不可通用。

```sh
$ docker run -d \
  -it \
  --name tmptest \
  --mount type=tmpfs,destination=/app,tmpfs-mode=1770 \
  nginx:latest
```

# 参考资料

[format 模板格式化](https://yq.aliyun.com/articles/230067)