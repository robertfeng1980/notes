# Docker Compose 入门与实践

[TOC]

`Docker Compose`是一个用Docker定义和运行复杂应用程序、运行多容器的工具。使用Compose可以在单个文件(使用`Compose`可以使用`YAML`文件来配置应用程序的服务)中定义一个多容器应用程序，然后将应用程序放在一个单独的命令中，该命令将完成所有需要完成的操作以使其运行。

使用Docker容器的应用程序通常由多个容器组成。使用`Docker Compose`，不需要编写`shell`脚本来启动容器。所有容器都使用服务在配置文件中定义，然后使用`docker-compose`脚本启动，停止和重新启动应用程序以及该应用程序中的所有服务以及该服务中的所有容器。



# 概述

`compose`适用于所有环境：生产、演示、开发、测试以及`CI`工作流程。可以了解有关常见使用案例中的更多信息。<br/>
使用`Compose`基本上是一个三步过程：
- 用一个定义你的应用程序的环境，`Dockerfile`这样它就可以在任何地方运行。
- 定义组成应用的服务，`docker-compose.yml` 以便它们可以在独立的环境中一起运行。
- 运行`docker-compose up`和编写启动并运行整个应用程序。


通常`docker-compose.yml`文件内容都像这样：

```yaml
version: '3'
services:
  web:
    build: .
    ports:
    - "5000:5000"
    volumes:
    - .:/code
    - logvolume01:/var/log
    links:
    - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```



## 作用

`Compose`具有管理应用程序整个生命周期：
- 启动，停止和重建服务
- 查看正在运行的服务的状态
- 流式传输运行服务的日志输出
- 在服务上运行一次性命令



## 内容

- [安装Compose](https://docs.docker.com/compose/install/)
- [入门](https://docs.docker.com/compose/gettingstarted/)
- [开始使用Django](https://docs.docker.com/compose/django/)
- [开始使用Rails](https://docs.docker.com/compose/rails/)
- [开始使用WordPress](https://docs.docker.com/compose/wordpress/)
- [经常问的问题](https://docs.docker.com/compose/faq/)
- [命令行参考](https://docs.docker.com/compose/reference/)
- [撰写文件参考](https://docs.docker.com/compose/compose-file/)



## 特点

`Compose`功能包括：

- [单个主机上有多个独立的环境](https://docs.docker.com/compose/overview/#Multiple-isolated-environments-on-a-single-host)
- [创建容器时保留卷数据](https://docs.docker.com/compose/overview/#preserve-volume-data-when-containers-are-created)
- [只重新创建已更改的容器](https://docs.docker.com/compose/overview/#only-recreate-containers-that-have-changed)
- [变量并在环境之间移动合成](https://docs.docker.com/compose/overview/#variables-and-moving-a-composition-between-environments)




### 单个主机上有多个独立的环境

使用`compose`项目来隔离项目彼此的环境。可以在多个不同的环境中使用此项目：

- 在开发主机上创建单个环境的多个副本，例如，当要为项目的每个功能分支运行稳定副本时
- 在CI服务器上，为防止构建互相干扰，可以将项目名称设置为唯一的构建编号
- 在共享主机或开发主机上，防止可能使用相同服务名称的不同项目相互干扰

默认项目名称是项目目录的名称。您可以使用[`-p`命令行选项](https://docs.docker.com/compose/reference/overview/)或 [`COMPOSE_PROJECT_NAME`环境变量](https://docs.docker.com/compose/reference/envvars/#compose-project-name)设置自定义项目名称 。



### 创建容器时保留卷数据

`compose`会保留服务使用的所有卷，当`docker-compose up` 运行时，如果它发现之前运行的容器，它会将旧容器中的内容复制到新容器中。此过程可确保在卷中创建的任何数据都不会丢失。

如果在`Windows`计算机上使用`docker-compose`，请参阅`环境变量`并根据特定需求调整必要的`环境变量`。



### 只重建已修改过的容器

`Compose`缓存用于创建容器的配置。当重新启动未更改的服务时，`Compose`会重新使用现有容器。使用现有的容器意味着可以快速更改环境。



### 变量在环境之间移动合成

`Compose`支持`yml`文件中的变量。可以使用这些变量为不同的环境或不同的用户定制编排服务。更多细节请参阅变量替换。您可以使用扩展字段或创建多个`Compose`文件来扩展`Compose`文件。



## 使用场景

`Compose`可用于许多不同的方式。下面概述了一些常见场景。



### 开发环境

在开发软件时，在独立环境中运行应用程序并与其交互的能力至关重要。`Compose`命令行工具可用于创建环境并与之交互。

在[`Compose`文件](https://docs.docker.com/compose/compose-file/)提供了一种记录和配置所有应用程序的服务依赖（数据库、队列、高速缓存、Web服务的API等等）。使用`Compose`命令行工具，可以使用单个命令（`docker-compose up`）为每个依赖项创建和启动一个或多个容器。

总之，这些功能为开发人员开始项目提供了一种便捷方式。`Compose`可以将多个服务配置文件缩减为一个机器可读的`Compose`文件和一些命令。



### 自动化测试环境

任何持续部署或持续集成过程的一个重要部分是自动化测试组件。自动化的端到端测试需要一个运行测试的环境。`Compose`提供了一种创建和销毁测试组件的独立测试环境的便捷方式。通过在[Compose文件中](https://docs.docker.com/compose/compose-file/)定义完整的环境，可以通过几条命令创建和销毁这些环境：

```
$ docker-compose up -d
$ ./run_tests
$ docker-compose down
```



### 单个主机部署

`Compose`传统上一直专注于开发和测试工作流程，但每次发布我们都在更多面向生产的功能方面取得进展。可以使用`Compose`部署到远程Docker引擎。Docker引擎可以是配备[Docker Machine](https://docs.docker.com/machine/overview/)或整个 [Docker Swarm](https://docs.docker.com/engine/swarm/)集群的单个实例 。



# 安装 `Compose`

`Compose` 可以在`macOS`，`Windows`和64位`Linux`上运行。在安装 `compose` 之前，必须先安装 `docker engine`。因此，如果没有安装 `docker engine` 就先安装。正常情况下，安装了`docker` 这两个东西都会安装。

检查是否安装过 `compose`，在终端窗口运行命令行：

```shell
$ docker-compose --version
docker-compose version 1.20.1, build 5d8c71b2
```

出现以上表明已经安装过`compose`



## 安装

**win 7 用户安装 compose，先下载文件：**

https://github.com/docker/compose/releases/download/1.21.0/docker-compose-Windows-x86_64.exe

下载完成后直接安装运行。

Linux 用户安装，直接下载文件解压到指定目录，随后修改权限即可：

```shell
$ sudo curl -L https://github.com/docker/compose/releases/download/1.21.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```



## 升级

如果要从Compose 1.2或更低版本升级，请在升级Compose后删除或迁移现有容器。这是因为，从版本1.3开始，Compose使用Docker标签来跟踪容器，并且需要重新创建容器以添加标签。

如果Compose检测到没有标签创建的容器，它会拒绝运行，这样你就不会得到两套。如果您想继续使用现有的容器（例如，因为它们有要保留的数据卷），则可以使用Compose 1.5.x通过以下命令来迁移它们：

```shell
$ docker-compose migrate-to-labels
```

另外，如果你不担心保留它们，你可以删除它们

```shell
$ docker container rm -f -v myapp_web_1 myapp_db_1 ...
```



# 运用

构建一个在`Docker Compose`上运行的简单`Python Web`应用程序。该应用程序使用`Flask`框架并在`Redis`中维护一个计数器。

## 编写程序和依赖

1、创建一个目录存放程序

```shell
$ mkdir compose_example
$ cd compose_example
```

2、编写程序内容，新建一个 `app.py`，内容如下：

```python
import time

import redis
from flask import Flask


app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```

在上面程序中，`redis`是应用程序网络上的`redis`容器的主机名。我们使用Redis的默认端口`6379`。

3、在项目目录中创建另一个文件`requirements.txt`，并将依赖的组件框架写入到文件中：

```
flask
redis
```



## 创建一个Dockerfile

编写一个`Dockerfile`来构建一个Docker镜像。该镜像包含`Python`应用程序需要的所有依赖项，包括`Python`本身。

在项目目录`/compose_example`中，创建一个名为`Dockerfile`并粘贴以下内容的文件：

```dockerfile
# 依赖的基础 python
FROM python:3.4-alpine
# 将当前目录代码添加到 code目录
ADD . /code
# 设置当前工作目录 为code
WORKDIR /code
# 运行pip 命令 安装依赖
RUN pip install -r requirements.txt
# 执行 python app.py
CMD ["python", "app.py"]
```

上面的文件内容告诉Docker：

- 从Python 3.4镜像开始构建一个镜像。
- 将当前目录添加`.`到`/code`镜像的路径中。
- 将工作目录设置为`/code`。
- 安装Python依赖项。
- 将容器的默认命令设置为`python app.py`。
  ​

## 定义服务

创建一个`docker-compose.yml` 文件放在项目目录`/compose_example`中，文件内容如下：

```yaml
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
  redis:
    image: "redis:alpine"
```

这个撰写文件定义了两个服务：`web`和`redis`服务：

- 使用`Dockerfile`当前目录中构建的镜像。
- 将容器上的暴露端口`5000`转发到主机上的端口`5000`。我们使用`Flask Web`，服务器的默认端口`5000`。

该`redis`服务使用从Docker Hub注册表中提取的公共 [Redis](https://registry.hub.docker.com/_/redis/)镜像。



## 构建镜像和运行程序

1、从项目目录`/compose_example`中，运行启动应用程序`docker-compose up`

```shell
$ docker-compose up
Creating network "composeexample_default" with the default driver
Building web
Step 1/5 : FROM python:3.4-alpine
3.4-alpine: Pulling from library/python
Digest: sha256:989b6044c434ffadf4dbc116719d73e7e31f5ac0f75f59b7591aeb766c874e26
Status: Downloaded newer image for python:3.4-alpine
 ---> 6610ae9fa51a
Step 2/5 : ADD . /code
 ---> 260f5df5d413
Step 3/5 : WORKDIR /code
Removing intermediate container a9713061bce0
 ---> 0e45bc38d999
Step 4/5 : RUN pip install -r requirements.txt
 ---> Running in c093d42d0b6d
......
web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
web_1    |  * Restarting with stat
web_1    |  * Debugger is active!
```

`compose`会拉取`redis`的镜像，为代码构建镜像，并启动定义的服务。在这种情况下，代码在构建时会复制到镜像中。

2、测试程序 http://192.168.99.100:5000/ ，如果是`win7` 电脑的情况，通过宿主主键访问 docker上的机器需要用 `machine` 机器的`ip`去访问。也可以直接到构建镜像的机器上运行curl访问

```shell
$ docker-machine ssh default
$ curl http://192.168.99.100:5000/
Hello World! I have been seen 1 times.
```

其他系统可以直接在机器上访问，也就是说你操作的docker构建程序的机器：http://localhost:5000 <br/>不知道 `machine`  ip地址的情况可以运行命令查看：`docker-machine ip default`

3、当我们重新请求 http://192.168.99.100:5000/ 发现里面的数据在累加

```shell
Hello World! I have been seen 3 times.
```

4、打开一个新的终端窗口，由于之前的窗口运行没有开启后台模式。输入命令`docker image ls`查看镜像列表

```shell
$ docker image ls
REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
composeexample_web                              latest              bc9f08679f4f        16 minutes ago      94.6MB
myhello                                         latest              75be173204aa        5 days ago          150MB
registry.cn-hangzhou.aliyuncs.com/hoo_jo/test   hello_world         75be173204aa        5 days ago          150MB
hoo_jo/test                                     hello_world         75be173204aa        5 days ago          150MB
hoojo/test                                      my_hello_world      75be173204aa        5 days ago          150MB
hello-world                                     latest              e38bc07ac18e        6 days ago          1.85kB
nginx                                           latest              b175e7467d66        7 days ago          109MB
redis                                           alpine              98bd7cfc43b8        3 weeks ago         27.8MB
python                                          2.7-slim            b16fde09c92c        3 weeks ago         139MB
python                                          3.4-alpine          6610ae9fa51a        3 weeks ago         83.6MB
```

上面有些是之前构建的镜像，可以看到依赖的 `python`、`redis`、`composeexample_web` 都在。

5、卸载删除应用程序运行命令 `docker-compose down` ，先进入到有 `docker-componse.yml`的目录下执行

```shell
$ cd docker/compose_example/
$ docker-compose down
Stopping composeexample_redis_1 ... done
Stopping composeexample_web_1   ... done
Removing composeexample_redis_1 ... done
Removing composeexample_web_1   ... done
Removing network composeexample_default
```

这样程序就被成功卸载删除并且停止

## 编辑 Compose 文件挂载程序

从项目目录`/compose_example`中，修改 `docker-compose.yml` 文件添加挂载目录，挂载目录指向本地的程序工作目录。

```yaml
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
    volumes:
     - .:/code
  redis:
    image: "redis:alpine"
```

新的`volumes`将主机上的项目目录（当前目录）挂载到容器内的`/code`目录中，允许即时修改代码，而无需重新构建发布镜像。

## 热部署应用程序

在项目目录`/compose_example`中执行构建启动命令`docker-compose up`

```shell
$ docker-compose up
Creating network "composeexample_default" with the default driver
Creating composeexample_web_1   ... done
Creating composeexample_redis_1 ... done
Attaching to composeexample_redis_1, composeexample_web_1
web_1    | python: can't open file 'app.py': [Errno 2] No such file or directory
```

运行后发现找不到程序，说目录或文件不存在。这个是由于我们的程序没有放在当前操作用户的目录下，如：`C:\Users` 或 `cd ~` 的目录。解决办法就是需要将当前工作目录的所在磁盘进行共享。让当前虚拟机能够访问到当前工作目录的磁盘。

> 共享当前工作目录到指定虚拟机，这里操作的虚拟机是 `default`。<br/>1、在桌面找到`Oracle VM VirtualBox`打开软件，在左侧找到虚拟机`default`，右键点击设置。弹出设置窗口后，选择`共享文件夹`，添加当前工作目录`D:\docker\compose_example`进行共享设置，共享目录名称为 `compose_example`。<br/>2、将刚才设置共享的目录`compose_example` 挂载到指定目录`/mnt/compose_example`上，命令行如下：
>
> ```shell
> $ docker-machine ssh default "sudo mkdir /mnt/compose_example"
> $ docker-machine ssh default "sudo chmod 755 /mnt/compose_example"
> $ docker-machine ssh default "sudo mount -t vboxsf compose_example /mnt/compose_example"
> ```

**再次修改 `docker-compose.yml`文件，修改挂载目录设置配置**，内容如下：

```yaml
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
    volumes:
     - "/mnt/compose_example:/code" 
  redis:
    image: "redis:alpine"
```

这样再次启动`docker-compose up`后，容器内的`/code`目录中，允许即时修改代码，而无需重新构建发布镜像。通过访问请求 http://192.168.99.100:5000/ 发现里面的数据在累加

```shell
Hello World! I have been seen 3 times.
```

## 修改应用程序

由于应用程序代码现在使用`volumes`挂载到容器中，因此可以更改代码并立即查看效果，而无需重新构建镜像进行重启操作。

修改`app.py`文件内容，修改返回值的内容，修改后的内容如下

```python
return 'Hello World! OMG!! I have been seen {} times.\n'.format(count)
```

通过访问请求 http://192.168.99.100:5000/ 发现内容已经变化了

```
Hello World! OMG!! I have been seen 12 times.
```



## 其他命令

### 后台模式

---

如果想在后台运行服务，可以将`-d`标志（“分离或后台”模式）传递给`docker-compose up`

```shell
$ docker-compose up -d
composeexample_redis_1 is up-to-date
composeexample_web_1 is up-to-date
```

### 查看日志

---

在后台模式下可以查看日志 `docker-compose logs`

```shell
$ docker-compose logs
$ docker-compose logs redis
```

### 查看运行的服务

---

并用于`docker-compose ps`查看当前正在运行的内容：

```shell
$ docker-compose ps
         Name                       Command               State           Ports
----------------------------------------------------------------------------------------
composeexample_redis_1   docker-entrypoint.sh redis ...   Up      6379/tcp
composeexample_web_1     python app.py                    Up      0.0.0.0:5000->5000/tcp
```

### 运行一次性

---

`docker-compose run`命令允许为服务运行一次性命令。例如，查看哪些环境变量可用于`Web`服务：

```shell
$ docker-compose run web env
PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=22e8f668029c
LANG=C.UTF-8
GPG_KEY=97FC712E4C024BBEA48A61ED3A5CA953F73C700D
PYTHON_VERSION=3.4.8
PYTHON_PIP_VERSION=9.0.3
HOME=/root
```

### 停止服务

---

如果开始使用`docker-compose up -d`进行启动服务，在完成后停止服务操作如下：

```shell
$ docker-compose stop
Stopping composeexample_web_1   ...
Stopping composeexample_redis_1 ...
```

### 卸载应用

---

可以把所有东西都卸载下来，用`docker-compose down`命令完全移除容器。通过`--volumes`还可以删除`redis`容器使用的数据量：

```shell
$ docker-compose down --volumes
Removing composeexample_web_run_1 ... done
Removing composeexample_web_1     ... done
Removing composeexample_redis_1   ... done
Removing network composeexample_default
```



## 命令行汇总

```shell
# 安装 docker-compose
$ sudo curl -L https://github.com/docker/compose/releases/download/1.21.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
# 授权
$ sudo chmod +x /usr/local/bin/docker-compose
$ docker-compose migrate-to-labels				# 升级compose
$ docker container rm -f -v myapp_web_1 myapp_db_1   # 删除容器
$ docker-compose up							# 启动应用
$ docker-compose up -d					# 后台模式启动应用
$ docker-compose stop					# 停止应用
$ docker-compose down					# 卸载应用
$ docker-compose ps						# 查看应用状态
$ docker-compose run web env			# 查看应用服务环境变量
$ docker-compose down --volumes			# 卸载应用并删除data数据
```



# Compose 命令行

通过`docker-compose --help`来查看compose命令行的帮助，会显示配置和命令行列表。可以使用`Docker Compose`二进制文件`docker-compose [-f <arg>...][options] [COMMAND][ARGS...]`在Docker容器中构建和管理多个服务。

```shell
$ docker-compose -h
使用Docker定义和运行多容器应用程序

Usage:
  docker-compose [-f <arg>...] [options] [COMMAND] [ARGS...]
  docker-compose -h|--help

Options:
  -f, --file FILE             指定一个备用的compose file
                              (默认: docker-compose.yml)
  -p, --project-name NAME     指定一个替代项目名称
                              (默认: 目录名称)
  --verbose                   显示更多输出
  --log-level LEVEL           设置日志级别 (DEBUG, INFO, WARNING, ERROR, CRITICAL)
  --no-ansi                   不要打印ANSI控制字符
  -v, --version               打印版本并退出
  -H, --host HOST             用于连接到的HOST守护程序套接字

  --tls                       使用TLS;--tlsverify
  --tlscacert CA_PATH         仅由此CA签署的信任证书
  --tlscert CLIENT_CERT_PATH  证书文件的路径
  --tlskey TLS_KEY_PATH       密钥文件的路径
  --tlsverify                 使用TLS并验证远程
  --skip-hostname-check       不要检查守护进程的主机名
                              在客户端证书中指定的名称
  --project-directory PATH    指定一个备用工作目录
                              (默认: Compose file的路径)
  --compatibility             如果设置，Compose将尝试转换部署
                              将v3文件中的密钥添加到其非Swarm等效项

Commands:
  build              构建或重建服务
  bundle             从Compose文件中生成一个Docker bundle
  config             验证并查看Compose file
  create             创建服务
  down               停止并移除容器，网络，图像和卷
  events             接收来自容器的实时事件
  exec               在正在运行的容器中执行命令 
  images             镜像列表
  kill               杀死容器
  logs               查看容器的日志输出
  pause              暂停服务
  port               打印端口绑定的公共端口
  ps                 容器列表
  pull               拉取服务镜像
  push               推送服务镜像
  restart            重启服务
  rm                 移除停止的容器
  run                运行一次性命令
  scale              设置服务的容器数量
  start              开始服务
  stop               停止服务
  top                显示正在运行的进程
  unpause            暂停服务
  up                 创建并启动容器 
```



## 选项 `-f` 用法

使用`-f`指定一个或多个`compose`文件名称和路径，使用`-f`标志来指定`Compose`配置文件的位置。

### 指定多个 compose 文件

---

可以提供多个`-f`来配置文件。当提供多个文件时，Compose将它们组合成单个配置。Compose按提供文件的顺序构建配置。随后的文件会覆盖并添加到他们的前置文件。以下是示例：

```shell
$ docker-compose -f docker-compose.yml -f docker-compose.admin.yml run backup_db
```

该`docker-compose.yml`文件可能会指定一项`webapp`服务。

```yaml
webapp:
  image: examples/web
  ports:
    - "8000:8000"
  volumes:
    - "/data"
```

如果`docker-compose.admin.yml`也指定了这个相同的服务，任何匹配的字段都会覆盖前一个文件。新的值添加到`webapp`服务配置。**也就是说相同的字段会进行覆盖，不同的就会进行组合。**

```yaml
webapp:
  build: .
  environment:
    - DEBUG=1
```

该`-f`选项是可选的。**如果不在命令行上提供此选项**，则Compose会遍历工作目录及其父目录，以查找文件`docker-compose.yml`和`docker-compose.override.yml`文件，至少提供`docker-compose.yml`文件。**如果两个文件都存在于相同的目录级别，则Compose会将这两个文件组合到一个配置中**。

`docker-compose.override.yml`文件中的配置将应用于`docker-compose.yml`文件中的值。

### 指定单个 compose 文件路径

---

可以使用`-f`标志来指定不在当前目录中的文件的路径，无论是从命令行还是通过在`shell`或环境文件中设置`compose_file`环境变量。

对于在命令行中使用`-f`选项的示例，假设正在运行`compose example`示例，并在名为`docker/compose_example`的目录中具有`docker-compose.yml`文件。可以使用像`docker-compose pull`这样的命令，通过使用`-f`标志从任何地方获取`cache`服务的`redis`镜像，如下所示：`docker-compose -f /d/docker/compose_example/docker-compose.yml pull redis`

```shell
$ cd ~
$ docker-compose -f /d/docker/compose_example/docker-compose.yml pull redis
Pulling redis (redis:alpine)...
alpine: Pulling from library/redis
Digest: sha256:e6e3a62b67b4e5c956b8814ac64ce3fe531c1093606f2a4fe5492921f6592388
Status: Image is up to date for redis:alpine
```



## 使用`-p`指定项目名称

每个配置都有一个项目名称。如果提供`-p`标志，则可以指定项目名称。如果未指定标志，则`compose`使用**当前目录**名称。项目名称会对应`COMPOSE_PROJECT_NAME`环境变量。



## 命令行环境变量

有几个环境变量可供配置`Docker Compose`命令行行为。以`DOCKER_`开头的变量与用于配置`docker`命令行客户端的变量相同。如果正在使用`docker-machine`，那么`eval "$(docker-machine env my-docker-vm)"`命令应该将它们设置为正确的值。（在本例中，`my-docker-vm`是创建的机器的名称。）



### 基本的环境变量

---

### COMPOSE_PROJECT_NAME

设置项目名称，在启动时，此值将与服务名称一起预先添加到容器中。例如，如果你的项目名称为`myapp`，它包括两个服务`db`和`web`，然后compose分别启动名为`myapp_db_1`和`myapp_web_1`的容器。

设置是可选的，如果不设置此项，则`COMPOSE_PROJECT_NAME` 默认为项目目录的基本名称。另请参阅`-p`命令行选项。

### COMPOSE_FILE

指定compose文件的路径。如果未提供，则compose会在当前目录中查找名为`docker-compose.yml`的文件，然后依次查找每个父目录，直到找到该名称的文件。

此变量支持多个由路径分隔符分隔的Compose文件（在Linux和MacOS上，路径分隔符是`:`在Windows上`;`）。例如：`COMPOSE_FILE=docker-compose.yml:docker-compose.prod.yml`。路径分隔符也可以使用自定义`COMPOSE_PATH_SEPARATOR`。

### COMPOSE_API_VERSION

`Docker API`仅支持来自特定版本的客户端的请求。如果使用 `docker-compose`收到`client and server don't have same version`错误，可以通过设置此环境变量来解决此错误，设置版本值以匹配服务器版本。

设置此变量解决需要临时运行客户端和服务器版本之间不匹配的情况。例如如果您可以升级客户端，但需要等待升级服务器。

使用此变量和已知的不匹配的版本，会阻止某些Docker功能正常工作。功能失败将取决于Docker客户端和服务器版本。出于这个原因，使用此变量集运行仅用作解决方法，并且不受官方支持。

如果遇到使用此设置运行的问题，请通过**升级解决不匹配问题并删除此设置**，以查看通知是否解决了问题。

### DOCKER_HOST

设置`docker`守护进程的`URL `。与Docker客户端一样，默认为`unix:///var/run/docker.sock`。

### DOCKER_TLS_VERIFY

当设置为除空字符串之外的任何其他字符时，都将启用与`docker`守护程序`TLS`的通信。

### DOCKER_CERT_PATH

配置用于TLS验证的`ca.pem`，`cert.pem`以及`key.pem`文件的路径。默认为`~/.docker`。

### COMPOSE_HTTP_TIMEOUT

配置`docker`守护进程的请求在`compose`认为失败之前允许挂起的时间（以秒为单位）。默认为`60`秒。

### COMPOSE_TLS_VERSION

配置哪些`TLS`版本用于与docker守护进程的通信。默认为`TLSv1`。支持的值是：`TLSv1`, `TLSv1_1`, `TLSv1_2`。

### COMPOSE_CONVERT_WINDOWS_PATHS

在卷定义中启用从`Windows`风格到`unix`风格的路径转换。`docker`和`docker toolbox`的用户应该始终设置这个。默认值为0，支持的值：`true`或`1 `启用，`false`或`0 `禁用。

### COMPOSE_PATH_SEPARATOR

如果设置`COMPOSE_FILE`则使用此字符作为路径分隔符，来分隔环境变量的值。

### COMPOSE_FORCE_WINDOWS_HOST

如果设置了，则假定主机路径是`Windows`路径，则使用短语法的卷声明将被解析，即使`compose`正在基于`Unix`的系统上运行。支持的值：`true`或`1`启用，`false`或`0`禁用。

### COMPOSE_IGNORE_ORPHANS

如果设置，compose不会尝试检测孤立项目容器。支持的值：`true`或`1`启用，`false`或`0`禁用。

### COMPOSE_PARALLEL_LIMIT

设置compose可以并行执行的操作数量的限制。默认值是`64`，可能不会低于`2`。

### COMPOSE_INTERACTIVE_NO_CLI

如果设置，compose不会尝试使用`docker cli`进行交互式运行和执行操作。此选项在上述操作需要`cli`的窗口中不可用。支持：`true`或`1`启用，`false`或`0`禁用。



### 在文件中配置环境变量
---

compose支持在执行`docker-compose`命令（当前工作目录）的文件夹中，建立名为`.env`的环境配置文件中声明默认环境变量。

#### 基本语法规则

这些语法规则适用于`.env`文件：

- compose期望`.env`文件中的每一行都是`VAR=VAL`格式。
- 开头的行`#`被处理为注释并被忽略。
- 空白行被忽略。
- 没有特殊的引号处理。这意味着 **它们是VAL的一部分**。


#### compose 文件和命令行变量

在此定义的环境变量用于Compose文件中的[变量替换](https://docs.docker.com/compose/compose-file/#variable-substitution)，也可用于定义以下[CLI变量](https://docs.docker.com/compose/reference/envvars/)：

- `COMPOSE_API_VERSION`
- `COMPOSE_CONVERT_WINDOWS_PATHS`
- `COMPOSE_FILE`
- `COMPOSE_HTTP_TIMEOUT`
- `COMPOSE_TLS_VERSION`
- `COMPOSE_PROJECT_NAME`
- `DOCKER_CERT_PATH`
- `DOCKER_HOST`
- `DOCKER_TLS_VERIFY`

运行时环境中的值始终会覆盖`.env`文件中定义的值。同样，通过命令行参数传递的值最优先。

在`.env`文件中定义的环境变量在容器中不可见。要设置容器适用的环境变量，请遵循compose中的[主题环境变量](https://docs.docker.com/compose/environment-variables/)中的指导原则，该指南描述了如何将`shell`环境变量传递给容器，如何在compose文件中定义环境变量等等。



## 命令

### up 装载

---

```shell
Usage: up [options] [--scale SERVICE=NUM...] [SERVICE...]

Options:
    -d, --detach               分离模式：在后台运行容器，打印新的容器名称。
                               不兼容 --abort-on-container-exit.
    --no-color                 产生单色输出.
    --quiet-pull               拉出没有打印进度信息
    --no-deps                  不要启动链接的服务.
    --force-recreate           重新创建容器，即使它们的配置和图像没有改变                               
    --always-recreate-deps     重新创建相关容器.
                               不兼容 --no-recreate.
    --no-recreate              如果容器已经存在，则不要重新创建他们
                               不兼容 --force-recreate and -V.
    --no-build                 不要构建图像，即使它不存在.
    --no-start                 创建后不要启动服务.
    --build                    在启动容器之前构建图像.
    --abort-on-container-exit  如果有容器，则停止所有容器停止
                               不兼容 -d.
    -t, --timeout TIMEOUT      对容器使用此超时（以秒为单位）
                               当连接时或容器被关闭时已经运行。 （默认：10）
                               
    -V, --renew-anon-volumes   重新创建匿名卷而不是检索来自之前容器的数据.
    --remove-orphans           移除未定义服务的容器在compose文件.
    --exit-code-from SERVICE   返回所选服务的退出代码容器
                               不兼容 --abort-on-container-exit.
    --scale SERVICE=NUM        将服务扩展到num实例. 如果存在覆盖，compose文件中的“缩放”设置.
```

**构建，（重新）创建，启动并附加**到服务的容器。除非它们已经在运行，否则该命令还会**启动任何关联**的服务。

该`docker-compose up`命令**汇总每个容器日志的输出**（相当于运行`docker-compose logs -f`），当命令退出时，所有容器都停止。<br/>运行`docker-compose up -d` 将在**后台启动容器并使其运行**。

**支持随时更新即热部署**：如果服务的已经有容器，并且在创建容器后服务的配置或镜像已更改，则`docker-compose up`通过停止并重新创建容器（保留已装入的卷）来提取更改。要防止`compose`的更改，请使用该`--no-recreate` 标志。

如果您想强制Compose停止并重新创建所有容器，请使用该 `--force-recreate`标志。

如果进程遇到错误，则此命令的退出代码为`1`。
如果使用`SIGINT`（`ctrl`+ `C`）中断进程或者`SIGTERM`容器停止，并且退出代码为`0`。
如果`SIGINT`或`SIGTERM`在此关闭阶段再次发送，正在运行的容器将被终止，并且退出代码为`2`。



### down 卸载

------

停止容器并移除由`up`创建的容器，网络，卷和镜像。

默认情况下，唯一删除的内容是：

- 在compose文件中定义的服务容器
- 在compose文件中定义的`networks`部分
- 默认网络（如果使用的话）

定义为外部`external` 的网络和卷永远不会被删除。

```shell
$ docker-compose down
Removing composeexample_redis_run_1 ... done
Removing composeexample_web_run_1   ... done
Removing network composeexample_default
```



### build 构建

---

服务构建一次，然后名称默认为`project_service`。例如，`composetest_db`。如果Compose文件指定了 [镜像](https://docs.docker.com/compose/compose-file/#image)名称，则镜像将用该名称进行设置。

如果修复也有程序的`Dockerfile`或其构建目录的内容，请运行`docker-compose build`以重建它。

```shell
$ docker-compose build
Building web
Step 1/5 : FROM python:3.4-alpine
 ---> 6610ae9fa51a
Step 2/5 : ADD . /code
 ---> 598dbdff6939
Step 3/5 : WORKDIR /code
.......
```

### bundle 打包

---

从Compose文件生成分布式应用程序包（DAB）。

镜像必须存储摘要，需要与Docker注册表进行交互。如果未为所有镜像存储摘要，可以使用`docker-compose pull` 或`docker-compose push`，打包时自动推送镜像，通过`--push-images`。只有`build`指定选项的服务才会推送其镜像。

```shell
$ docker login -u xxx -p xxxx
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Login Succeeded

$ docker-compose.exe build web
Building web
Step 1/5 : FROM python:3.4-alpine
 ---> 6610ae9fa51a
Step 2/5 : ADD . /code
 ---> ead50f337cd7
Step 3/5 : WORKDIR /code
.....

$ docker-compose bundle --push-image -o ./compose.dab
Pushing web (hoojo/test:compose_example)...
The push refers to repository [docker.io/hoojo/test]
compose_example: digest: sha256:1adecad81d64505cba06d071487ef16485d3de18d134d4cdcb308f4f82769409 size: 1786
Unsupported key 'volumes' in services.web - ignoring
Wrote bundle to ./compose.dab
```

上面的三步完成后，可以看到最后一步将打包好的镜像上传到仓库，并在当前目录生成了一个`compose.dab`文件。这个文件是摘要文件数据。



### config 配置

---

验证并查看compose文件，一般可以通过此命令检查`docker-compose.yml`是否编写正确。并且能显示`docker-compose.yml`的内容。

错误的配置文件：

```shell
$ docker-compose config
The Compose file '.\docker-compose.yml' is invalid because:
Unsupported config option for services.web: 'image2'
```

正确的配置文件：

```shell
$ docker-compose config
services:
  redis:
    image: redis:alpine
  web:
    build:
      context: D:\docker\compose_example
    image: hoojo/test:compose_example
    ports:
    - 5000:5000/tcp
    volumes:
    - /code:/code:rw
version: '3.0'
```



### events 事件

---

监控项目应用程序容器的事件动态。使用该`--json`标志，每行打印一个json对象，格式为：

```shell
$ docker-compose.exe events --json
{"time": "2018-04-19T18:00:54.483975", "type": "container", "action": "create", "id": "510e7d26795531b6ed9aa4e7deaeb982338726d251f9d4864e242fb0a3c2bb54", "service": "web", "attributes": {"name": "composeexample_web_1", "image": "hoojo/test:compose_example"}}
{"time": "2018-04-19T18:00:54.493158", "type": "container", "action": "attach", "id": "510e7d26795531b6ed9aa4e7deaeb982338726d251f9d4864e242fb0a3c2bb54", "service": "web", "attributes": {"name": "composeexample_web_1", "image": "hoojo/test:compose_example"}}
{"time": "2018-04-19T18:00:54.920329", "type": "container", "action": "start", "id": "510e7d26795531b6ed9aa4e7deaeb982338726d251f9d4864e242fb0a3c2bb54", "service": "web", "attributes": {"name": "composeexample_web_1", "image": "hoojo/test:compose_example"}}
```

启动后控制台将处于监控模式，这时候可以再打开一个窗口执行 `docker-compose build `和`docker-compose up`就可以看到数据。



### exec 执行

---

这相当于`docker exec`。使用此子命令可以在服务中运行任意命令。命令默认分配一个TTY，所以可以使用一个命令`docker-compose exec web sh`来获得交互式提示。

```shell
$ docker-compose ps --services
web
redis

$ docker-compose exec web sh
the input device is not a TTY.  If you are using mintty, try prefixing the command with 'winpty'

$ winpty docker-compose exec web sh
```

由于用`Git bash`在安装的时候没有选择**Use MinTTY**，所以这里就到 `Power Shell`工具中进行操作。还有一个解决办法就是根据提示执行命令：`$ winpty docker-compose exec web sh`

```powershell
PS D:\docker\compose_example> docker-compose.exe exec web sh
/code # ls
Dockerfile          compose.dab         requirements.txt
app.py              docker-compose.yml
```

运行上面的命令后，就可以查看当前`web`镜像的工作命令数据。还能与远程镜像数据进行交互。



### kill 杀死

---

通过发送信号`SIGKILL`来强制停止运行容器。可以选择传递信号，例如：

```shell
$ docker-compose kill -s SIGINT
Killing composeexample_redis_1 ...
Killing composeexample_web_1   ... done
Killing composeexample_redis_1 ... done
```



### logs 日志

---

显示服务的日志输出

```shell
$ docker-compose ps --services
web
redis

$ docker-compose logs redis

$ docker-compose logs web
Attaching to composeexample_web_1
web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
web_1    |  * Restarting with stat
web_1    |  * Debugger is active!
web_1    |  * Debugger PIN: 184-102-760
```



### pause 暂停

---

暂停运行服务的容器，可以解除暂停 `docker-compose unpause`

```shell
$ docker-compose ps --services
web
redis

$ docker-compose pause web
Pausing composeexample_web_1 ... done

$ docker-compose unpause web
Unpausing composeexample_web_1 ... done
```



### port 端口

---

打印用于端口绑定的公共端口。下面是部署的web应用的端口，从之前编辑的`docker-compose.yml`中的`services`可以看出此服务。

```shell
$ docker-compose ps --services
web
redis

$ docker-compose port web 5000
0.0.0.0:5000
```



### ps 容器列表

---

显示容器列表

```shell
$ docker-compose ps
         Name                       Command               State           Ports
----------------------------------------------------------------------------------------
composeexample_redis_1   docker-entrypoint.sh redis ...   Up      6379/tcp
composeexample_web_1     python app.py                    Up      0.0.0.0:5000->5000/tcp

$ docker-compose ps --services
web
redis
```



### pull 拉取

---

提取在`docker-compose.yml`或`docker-stack.yml `文件中定义的服务相关联的镜像，但不会基于这些镜像启动容器。

```yaml
version: '3'
services:
  web:
    image: hoojo/test:compose_example
    build: .
    ports:
     - "5000:5000"
    volumes:
     - "/code:/code"
  redis:
    image: "redis:alpine"
```

执行定义服务的拉取`docker-compose pull ServiceName`需要在`docker-compose.yml`文件的所在的目录中运行，则Docker会拉取关联的镜像。例如，要在我们的示例中调用`redis:alpine`配置为`redis`服务的镜像，可以运行`docker-compose pull redis`。

```shell
$ docker-compose pull redis
Pulling redis (redis:alpine)...
alpine: Pulling from library/redis
Digest: sha256:e6e3a62b67b4e5c956b8814ac64ce3fe531c1093606f2a4fe5492921f6592388
Status: Image is up to date for redis:alpine
```



### push 推送

---

将服务的镜像推送到它们各自的位置`registry/repository`

做出以下假设：

- 您正在推送您在本地创建的镜像
- 您可以访问构建密钥

```yaml
version: '3'
services:
  service1:
    build: .
    image: localhost:5000/web  # goes to local registry

  service2:
    build: .
    image: hoojo/my_image:first  # goes to youruser DockerHub registry
```

当程序被构建执行时，会自动`push`到远程服务器仓库中。

```shell
$ docker-compose push
Pushing web (hoojo/test:compose_example)...
The push refers to repository [docker.io/hoojo/test]
compose_example: digest: sha256:aa0aa4df76f3676765a44f47825aabf1b1b579e37836f7d3b6a866416f3ac18e size: 1787
```



### start 启动

---

启动一个服务的现有容器。

```shell
$ docker-compose start
```



### stop 停止

---

停止运行容器而不删除它们。可以用`docker-compose start`重新启动。

```shell
$ docker-compose stop
```



### restart 重启

---

重新启动所有停止和正在运行的服务。<br/>如果对`docker-compose.yml`配置进行了更改，则运行此命令后这些更改不会反映出来。<br/>例如，对环境变量（在构建容器后但在容器的命令执行之前添加）的更改在重新启动后不会更新。

```shell
$ docker-compose restart
Restarting composeexample_redis_1 ... done
Restarting composeexample_web_1   ... done
```



### run 运行

---

**针对服务运行一次性命令**。例如，以下命令启动该`web`服务并`bash`作为其命令运行。

```shell
$ docker-compose run web sh # 执行命令后进入交互模式，随后可以执行shell的基本命令行
ls
Dockerfile
app.py
compose.dab
docker-compose.yml
requirements.txt

```

使用命令`run`从具有由服务定义的配置的新容器中启动，包括卷，链接和其他详细信息。但有两个重要的区别。

首先，通过命令`run`将覆盖服务配置中定义的命令。例如，如果`web`服务配置是用`bash`启动的，那么`docker-compose run web python app.py`会用`python app.py`覆盖它。

第二个区别是该`docker-compose run`命令不会创建服务配置中指定的任何端口。这可以防止端口与已打开的端口发生冲突。如果您*确实想要*创建服务的端口并将其映射到主机，请指定`--service-ports`标志：

```shell
$ docker-compose run --service-ports web python app.py shell
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 184-102-760
```

或者，可以使用`--publish`或`-p`选项指定手动端口映射，就像在使用时一样`docker run`：

```shell
$ docker-compose run --publish 8080:80 -p 2022:22 -p 127.0.0.1:2021:21 web python app.py shell
```

如果启动一个配置了链接的服务，则该`run`命令首先检查链接服务是否正在运行，并在服务停止时启动服务。一旦所有链接的服务正在运行，`run`执行通过它传递的命令。例如运行：

```shell
$ docker-compose run db psql -h db -U docker
```

这将为链接的`db`容器打开一个交互式`PostgreSQL shell`。

如果您不希望`run`命令启动链接的容器，请使用`--no-deps`标志：

```shell
$ docker-compose run --no-deps web python app.py shell
```



### rm 删除

------

删除已停止的服务容器。

默认情况下，附加到容器的匿名卷不会被删除。你可以用这个覆盖它`-v`。要列出所有卷，请使用`docker volume ls`。<br/>任何不在卷中的数据都会丢失。<br/>在没有选项的情况下运行该命令也会删除由`docker-compose up`或创建的一次性容器`docker-compose run`：

```shell
$ docker-compose rm
Going to remove djangoquickstart_web_run_1
Are you sure? [yN] y
Removing djangoquickstart_web_run_1 ... done
```



### top 显示

---

显示正在运行的进程。

```shell
$ docker-compose top

composeexample_redis_1
PID      USER       COMMAND
------------------------------
7373   dockrema   redis-server

composeexample_web_run_4
PID    USER                COMMAND
------------------------------------------------
7071   root   python app.py shell
7121   root   /usr/local/bin/python app.py shell

composeexample_web_run_5
PID    USER                COMMAND
------------------------------------------------
7556   root   python app.py shell
7606   root   /usr/local/bin/python app.py shell
```



# Compose file 应用

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



## 服务配置参考

`Compose`文件是一个定义[服务](https://docs.docker.com/compose/compose-file/#service-configuration-reference)、 [网络](https://docs.docker.com/compose/compose-file/#network-configuration-reference)和 [卷](https://docs.docker.com/compose/compose-file/#volume-configuration-reference)的[YAML](http://yaml.org/)文件 。`Compose`文件的默认路径是。`./docker-compose.yml`

> **提示**：可以对此文件使用 `.yml`或`.yaml`扩展名，他们都支持。

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



### build 构建

---

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



#### context 上下文

---

可以是`Dockerfile`所在的目录的路径，也可以是`Git`仓库的`url`。

当`context`的值是相对路径时，它被解释为相对于`docker-compose.yam`文件的位置。这个目录也是推送到Docker守护进程的构建上下文。

```yaml
build:
  context: ./sample
```



#### dockerfile 

---

备用`Dockerfile`。`compose`命令使用替代文件来构建，还必须指定构建路径 `build` 的值。

```yacas
 build:
      context: ./sample
      dockerfile: Dockerfile-alternate
```

在 `sample` 目录中必须有文件名称为 `Dockerfile-alternate`的`dockerfile`



#### args 参数

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



#### cache_from 缓存

---

引擎用于缓存解析的镜像列表。**在版本：3.2 可用**

```yaml
build:
  context: .
  cache_from:
    - redis:latest
    - hoojo/web_app:3.14
```



#### lables 标签

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



#### shm_size 分区大小

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



#### target 目标

---

根据构建阶段在`Dockerfile`中定义指定目标配置。**在3.4版本可用**

```yaml
  build:
    context: .
    target: prod # 在dockerfile中的prod 构建阶段
```



### cap_add，cap_drop 添加删除容器

---

添加或删除容器功能。请参阅`man 7 capabilities`完整列表。

```yaml
cap_add:
  - ALL

cap_drop:
  - NET_ADMIN
  - SYS_ADMIN
```

> **注意**：[在](https://docs.docker.com/engine/reference/commandline/stack_deploy/) 使用（版本3）Compose文件的[群集模式下部署堆栈](https://docs.docker.com/engine/reference/commandline/stack_deploy/)时，会忽略这些选项 。



### command 命令

---

启动后执行的命令，会覆盖默认命令。

```
command: bundle exec thin -p 3000
```

该命令也可以是一个列表，方式类似于 [dockerfile](https://docs.docker.com/engine/reference/builder/#cmd)：

```
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



### configs 配置

---

使用每项服务`configs` 配置为每个服务授予对配置的访问权限。支持两种不同的语法变体。

> **注意**：配置必须已经存在或 [在`configs`](https://docs.docker.com/compose/compose-file/#configs-configuration-reference) 此堆栈文件[的顶层](https://docs.docker.com/compose/compose-file/#configs-configuration-reference)[配置](https://docs.docker.com/compose/compose-file/#configs-configuration-reference)中[定义](https://docs.docker.com/compose/compose-file/#configs-configuration-reference)，否则堆栈部署失败。



#### SHORT SYNTAX 短语法

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

#### LONG SYNTAX 长语法

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



### cgroup_parent 上级组

---

为容器指定一个可选的父cgroup。

```yaml
cgroup_parent: m-executor-abcd
```

> **注意**：当 使用（版本3）Compose文件[在群集模式下部署堆栈时，](https://docs.docker.com/engine/reference/commandline/stack_deploy/)忽略此选项 。



### container_name 容器名称

---

指定一个自定义容器名称，而不是生成的默认名称。

```
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



### credential_spec 凭据规范

> **注意：**该选项已添加到v3.3中

为托管服务帐户配置凭据规范。此选项仅用于使用Windows容器的服务。在`credential_spec`必须在格式`file://<filename>`或`registry://<value-name>`。

使用时`file:`，引用的文件必须存在于`CredentialSpecs` docker数据目录的子目录中，该目录默认为`C:\ProgramData\Docker\` 在Windows上。以下示例从名为的文件加载凭证规范`C:\ProgramData\Docker\CredentialSpecs\my-credential-spec.json`：

```
credential_spec:
  file: my-credential-spec.json
```

使用时`registry:`，将从守护进程主机上的Windows注册表中读取凭据规范。具有给定名称的注册表值必须位于：

```
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\CredentialSpecs
```

以下示例从`my-credential-spec` 注册表中指定的值加载凭证规范：

```
credential_spec:
  registry: my-credential-spec
```



### deploy 部署

---

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

#### endpoint_mode 端点模式

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

#### labels 标签

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

#### mode 模式

全局唯一模式：`global`，还是副本多实例模式：`replicated`，若是副本模式就需要指定副本数量`replicas`。要么`global`（每个群集节点只有一个容器）或`replicated`（指定数量的容器）。默认是`replicated`。（要了解更多信息，请参阅[swarm](https://docs.docker.com/engine/swarm/)主题 中的[复制和全局服务](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/#replicated-and-global-services)。）

```yaml
version: '3'
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    deploy:
      mode: global
```

#### placement 放置

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

#### replicas 副本

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

#### resources 资源

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



##### Out Of Memory Exceptions 内存异常（OOME）

如果服务或容器尝试使用比系统可用的内存更多的内存，则可能会遇到内存异常（`OOME`），并且容器或Docker守护程序可能会被内核OOM错误所杀。要防止发生这种情况，请确保应用程序在具有足够内存的主机上运行，并且请参阅[了解耗尽内存的风险](https://docs.docker.com/engine/admin/resource_constraints/#understand-the-risks-of-running-out-of-memory)。

#### restart_policy 重启策略

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

#### update_config 更新配置

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

### 不支持 `DOCKER STACK DEPLOY`

---

下面的子选项（支持`docker compose up`和`docker compose run`）是*不支持*的`docker stack deploy`或`deploy`关键的。

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

> **提示：**请参阅关于[如何为服务，群集和docker-stack.yml文件配置卷](https://docs.docker.com/compose/compose-file/#volumes-for-services-swarms-and-stack-files)的部分。支持卷*，*但要与群集和服务一起使用，它们必须配置为命名卷或与限制为可访问必需卷的节点的服务相关联

