# docker plugin 应用

[TOC]

# docker plugin 基本命令

```sh
$ docker plugin -h
Usage:  docker plugin COMMAND
插件管理

Commands:
  create      # 从rootfs和配置创建一个插件。插件数据目录必须包含config.json和rootfs目录。
  disable     # 禁用插件
  enable      # 启用插件
  inspect     # 显示一个或多个插件的详细信息
  install     # 安装一个插件
  ls          # 插件列表
  push        # 将插件推送到注册表
  rm          # 删除一个或多个插件
  set         # 更改插件的设置
  upgrade     # 升级现有的插件
```

## create 创建

创建一个插件。在创建插件之前，请准备插件的根文件系统以及 [config.json](https://docs.docker.com/engine/extend/config/)。以下示例显示如何创建示例plugin：

```sh
$ ls -ls /home/pluginDir
total 4

4 -rw-r--r--  1 root root 431 Nov  7 01:40 config.json
0 drwxr-xr-x 19 root root 420 Nov  7 01:40 rootfs

$ docker plugin create plugin /home/pluginDir
# docker plugin create plugin /home/pluginDir --compress
plugin

$ docker plugin ls
ID                  NAME                TAG                 DESCRIPTION                  ENABLED
672d8144ec02        plugin              latest              A sample plugin for Docker   false
```

插件可以随后启用本地使用或推送到公共注册表。

## install 安装

安装并启用插件。Docker首先查找Docker主机上的插件。如果插件本地不存在，则插件将从注册表中提取。请注意，分发插件所需的最低注册表版本是2.3.0

 ### 命令参数选项

---

| 选项，简写                | 默认   | 描述                       |
| ------------------------- | ------ | -------------------------- |
| `--alias`                 |        | 插件的本地名称             |
| `--disable`               |        | 不要在安装时启用插件       |
| `--disable-content-trust` | `true` | 跳过图像验证               |
| `--grant-all-permissions` |        | 授予运行插件所需的所有权限 |

### 示例

---

下面的示例安装`vieus/sshfs`插件，并[设置](https://docs.docker.com/engine/reference/commandline/plugin_set/)它的 `DEBUG`环境变量`1`。要安装Docker Hub的插件并提示用户接受插件所需的权限列表，请设置插件的参数并启用插件。

```sh
$ docker plugin install vieux/sshfs DEBUG=1
```

## disable 禁用

禁用插件。插件必须先安装，然后才能禁用，请参阅[`docker plugin install`](https://docs.docker.com/engine/reference/commandline/plugin_install/)。没有这个`-f`选项，一个有引用的插件（例如卷，网络）不能被禁用。

 ```sh
$ docker plugin disable tiborvass/sample-volume-plugin
tiborvass/sample-volume-plugin

$ docker plugin ls
 ```

## enable 启用

启用插件。插件必须先安装才能启用，请参阅[`docker plugin install`](https://docs.docker.com/engine/reference/commandline/plugin_install/)。

```sh
$ docker plugin lsID                  NAME                             TAG                 DESCRIPTION                ENABLED69553ca1d123        tiborvass/sample-volume-plugin   latest              A test plugin for Docker   false

$ docker plugin enable tiborvass/sample-volume-plugin
tiborvass/sample-volume-plugin

$ docker plugin ls
ID                  NAME                             TAG                 DESCRIPTION                ENABLED
69553ca1d123        tiborvass/sample-volume-plugin   latest              A test plugin for Docker   true
```

## inspect 查看

返回关于插件的信息。默认情况下，该命令将所有结果呈现在JSON数组中。

 ```sh
$ docker plugin inspect tiborvass/sample-volume-plugin:latest
$ docker plugin inspect -f '{{.Id}}' tiborvass/sample-volume-plugin:latest
 ```

## ls 列表

列出当前安装的所有插件。您可以使用该[`docker plugin install`](https://docs.docker.com/engine/reference/commandline/plugin_install/)命令安装插件。您也可以使用`-f`或`--filter`标志进行过滤。有关可用过滤器选项的更多信息，请参阅[过滤](https://docs.docker.com/engine/reference/commandline/plugin_ls/#filtering)部分。

 ```sh
$ docker plugin ls
ID                  NAME                             TAG                 DESCRIPTION                ENABLED
69553ca1d123        tiborvass/sample-volume-plugin   latest              A test plugin for Docker   true

# 过滤
$ docker plugin ls --filter enabled=true
 ```

### 格式化

格式化选项（`--format`）使用Go模板漂亮地打印插件输出。

下面列出了Go模板的有效占位符：

| 占位符             | 描述                      |
| ------------------ | ------------------------- |
| `.ID`              | 插件ID                    |
| `.Name`            | 插件名称                  |
| `.Description`     | 插件描述                  |
| `.Enabled`         | 是否启用插件              |
| `.PluginReference` | 用于从注册表中推/拉的参考 |

使用该`--format`选项时，`plugin ls`命令将按照模板声明输出数据，或者在使用该 `table`指令时也包含列标题。

下面的示例使用的模板没有报头，并输出 `ID`和`Name`通过对所有的插件冒号分隔的条目：

```sh
$ docker plugin ls --format "{{.ID}}: {{.Name}}"
```



# 管理插件

Docker Engine的插件系统允许使用Docker Engine安装、启动、停止和移除插件。

有关Docker Engine 1.12及更早版本中提供的传统插件系统的信息，请参阅[了解传统Docker引擎插件](https://docs.docker.com/engine/extend/legacy_plugins/)。

> **注意**：Windows守护程序目前不支持Docker Engine托管插件。

## 安装和使用插件

插件以Docker镜像的形式发布，可以托管在Docker Hub或私有注册表中。要安装插件，请使用`docker plugin install`从Docker Hub或您的私人注册表中提取插件的命令，并在必要时提示您授予权限或功能，并启用该插件。

要检查已安装插件的状态，请使用该`docker plugin ls`命令。成功启动的插件在输出中列为启用。

在安装插件之后，可以将其用作另一个Docker操作的选项，例如创建卷。在以下示例中，将安装`sshfs`插件，验证它是否已启用，并使用它来创建卷。

> **注意**：本示例仅用于说明目的。创建卷后，在检查卷时，远程主机的SSH密码将作为纯文本公开。应该在完成示例后立即删除该卷。

1. 安装`sshfs`插件。

   ```sh
   $ docker plugin install vieux/sshfs
   
   Plugin "vieux/sshfs" is requesting the following privileges:
   - network: [host]
   - capabilities: [CAP_SYS_ADMIN]
   Do you grant the above permissions? [y/N] y
   
   vieux/sshfs
   ```

   该插件请求2个权限：

   - 它需要访问`host`网络。
   - 它需要`CAP_SYS_ADMIN`能够让插件运行`mount`命令的功能。

2. 检查插件是否在输出中启用`docker plugin ls`。

   ```sh
   $ docker plugin ls
   ```

