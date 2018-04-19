# Docker Compose 开发指南

[TOC]

`Compose`是定义和运行多容器Docker应用程序的工具。使用`Compose`可以使用`YAML`文件来配置应用程序的服务。然后使用单个命令，可以创建并启动配置中的所有服务。要详细了解`Compose`的所有功能，请参阅功能列表。



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

  ​

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



### 变量并在环境之间移动合成

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

- 从Python 3.4图像开始构建一个图像。

- 将当前目录添加`.`到`/code`图像的路径中。

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

- 使用`Dockerfile`当前目录中构建的图像。
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

`compose`会拉取`redis`的镜像，为代码构建图像，并启动定义的服务。在这种情况下，代码在构建时会复制到镜像中。

2、测试程序 http://192.168.99.100:5000/ ，如果是`win7` 电脑的情况，通过宿主主键访问 docker上的机器需要用 `machine` 机器的`ip`去访问。也可以直接到构建镜像的机器上运行curl访问

```shell
$ docker-machine ssh default
$ curl http://192.168.99.100:5000/
Hello World! I have been seen 1 times.
```

其他系统可以直接在机器上访问，也就是说你操作的docker构建程序的机器：http://localhost:5000

不知道 `machine`  ip地址的情况可以运行命令查看：`docker-machine ip default`

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

5、停止应用程序的运行 `docker-compose down` ，先进入到有 `docker-componse.yml`的目录下执行

```shell
$ cd docker/compose_example/
$ docker-compose down
Stopping composeexample_redis_1 ... done
Stopping composeexample_web_1   ... done
Removing composeexample_redis_1 ... done
Removing composeexample_web_1   ... done
Removing network composeexample_default
```

这样程序就被成功停止

## 编辑 Compose 文件





