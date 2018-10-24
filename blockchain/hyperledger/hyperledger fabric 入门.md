# `hyperledger fabric` 入门
入门章节会检查是否已在将要开发区块链应用程序和运行`Hyperledger Fabric`的平台上安装了所有[必要环境](https://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html)。

安装必备组件后，即可下载并安装`HyperLedger Fabric`。在我们为`Fabric`二进制文件开发真正的安装程序时，我们提供了一个脚本，可以[将示例、二进制文件和Docker镜像安装](https://hyperledger-fabric.readthedocs.io/en/latest/install.html)到系统中。该脚本还将`Docker`镜像下载到本地仓库中。

## `Hyperledger Fabric SDK`

`Hyperledger Fabric`提供了许多`SDK`来支持各种编程语言。有两个正式发布的`Node.js`和`Java SDK`：

 - [Hyperledger Fabric Node SDK](https://github.com/hyperledger/fabric-sdk-node)和[Node SDK文档](https://fabric-sdk-node.github.io/)。
 - [Hyperledger Fabric Java SDK](https://github.com/hyperledger/fabric-sdk-java)。

此外，还有三个尚未正式发布的`SDK`（适用于`Python`，`Go`和`REST`），但它们仍可供下载和测试：

 - [Hyperledger Fabric Python SDK](https://github.com/hyperledger/fabric-sdk-py)。
 - [Hyperledger Fabric Go SDK](https://github.com/hyperledger/fabric-sdk-go)。
 - [Hyperledger Fabric REST SDK](https://github.com/hyperledger/fabric-sdk-rest)。

## `Hyperledger Fabric CA`

`Hyperledger Fabric`提供可选的 [证书颁发机构服务](http://hyperledger-fabric-ca.readthedocs.io/en/latest) ，您可以选择使用该服务生成证书和密钥材料，以配置和管理区块链网络中的身份。但是，可以使用任何可以生成`ECDSA`证书的CA。



# 运行环境

在我们开始之前，如果还没有这样做，可能希望检查以下所有环境条件是否已安装在您将开发区块链应用程序和运行`Hyperledger Fabric`的平台上。 

以下软件都是在`Linux/ Ubuntu 16.04` 系统上进行安装，首先请安装好系统。

- [ ] 安装`cURL` 工具：https://curl.haxx.se/download.html

- [ ] 安装`Docker 和 Docker Compose`：https://www.docker.com/get-docker
  - `docker` 安装完成后可以用 `docker --version` 检查docker的版本
  - `docker compose` 安装完成后可以用`docker-compose --version` 查看版本号

- [ ] 安装`Go` 语言环境的安装：https://golang.org/dl/
  - 版本必须在`1.10.x`以上
  - 环境变量配置：`export GOPATH=$HOME/go` 和 `export PATH=$PATH:$GOPATH/bin`

- [x] `nodejs` 和 `npm` 的安装：https://nodejs.org/en/download/

  - `npm`的安装 `npm install npm@5.6.0 -g`

- [x] `python` 的安装：

  - `sudo apt-get install python`

    安装完成后检查版本：`python --version`

- [ ] 安装`Git` 工具：https://git-scm.com/downloads

  - 设置必要的配置

    ```sh
    git config --global core.autocrlf false
    git config --global core.longpaths true
    ```


## `cURL` 安装

`cURL` 是一个下载工具，支持`HTTP、HTTPS、FTP、FTPS、DICT、TELNET、LDAP、FILE`和`GOPHER`。利用[URL](https://baike.baidu.com/item/URL)语法在命令行方式下工作的开源文件传输工具。它被广泛应用在[Unix](https://baike.baidu.com/item/Unix)、多种[Linux](https://baike.baidu.com/item/Linux)发行版中，并且有[DOS](https://baike.baidu.com/item/DOS/32025)和[Win32](https://baike.baidu.com/item/Win32)、`Win64`下的移植版本。

### 下载`cURL`
找到最新的版本 https://curl.haxx.se/download/，下载地址：https://curl.haxx.se/download/curl-7.59.0.tar.xz

```shell
wget https://curl.haxx.se/download/curl-7.59.0.tar.xz
```

### 解压

```shell
xz -d curl-7.59.0.tar.xz
tar -C /opt/ -xvf curl-7.59.0.tar
```

### 安装

```shell
cd curl-7.59.0/
./configure
 sudo make
 如果没有make命令，那就需要安装下
 sudo apt install yum
 sudo yum -y install make 或 sudo apt-get install make
 
 make install
```

# 安装示例和二进制文件

利用脚本，可以下载并安装样本和二进制文件到系统中。

> **注意**：如果在**Windows**上运行，则需要使用`Docker Quickstart Terminal` 或 `Git bash`来执行即将发布的终端命令。如果之前没有安装，请访问[环境条件](https://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html)。
>
> 如果您在`Windows 7`或`macOS`上使用`Docker Toolbox`，则在安装和运行示例时，需要使用`C:\Users`（`Windows 7`）或 `/Users`（`macOS`）下的位置。
>
> 如果使用的`docker`的`Mac`，你需要在使用位置`/Users`，`/Volumes`，`/private`，或`/tmp`。要使用其他位置，请参阅`Docker`文档以获取 [文件共享](https://docs.docker.com/docker-for-mac/#file-sharing)。
>
> 如果使用的是`Docker for Windows`，请参阅`Docker`文档以获取[共享驱动器，](https://docs.docker.com/docker-for-windows/#shared-drives) 并使用其中一个共享驱动器下的位置。

## 安装流程

确定计算机上要放置`fabric-samples` 代码库的位置，并在终端窗口中输入该目录。后面的命令将执行以下步骤：

1. 如果需要，克隆[hyperledger/fabric-samples](https://github.com/hyperledger/fabric-samples)存储库
2. 切到适当的版本标签
3. 将指定版本的`Hyperledger Fabric`平台特定二进制文件和配置文件安装到`fabric-samples`存储库的根目录中
4. 下载指定版本的`Hyperledger Fabric docker`镜像



## 下载

### 下载示例代码

将示例代码下载下来，并且解压安装到 `opt/fabric-samples`目录下。准备好后，在要安装`Fabric Samples`和二进制文件的目录中，继续执行以下命令： 

```sh
$ cd opt/gopath/src/github.com/hyperledger/fabric-samples
$ curl -sSL http://bit.ly/2ysbOFE | bash -s 1.3.0
```
> **注意**：您可以对任何已发布的`Hyperledger Fabric`版本使用上述命令。只需将`1.3.0`替换为您要安装的版本的版本标识符即可。



如果上面的不行可以尝试下面命令：

```shell
$ cd opt/gopath/src/github.com/hyperledger/fabric-samples
$ git clone -b master https://github.com/hyperledger/fabric-samples.git
$ cd fabric-samples
$ git checkout v1.1.0
```

### 下载`Fabric`，`Fabric-ca`和第三方`Docker`镜像

下载`Fabric`，`Fabric-ca`和第三方`Docker`镜像，则必须将**版本标识符**传递给脚本。

```sh
$ cd opt/gopath/src/github.com/hyperledger/fabric-samples
#curl -sSL http://bit.ly/2ysbOFE | bash -s <fabric> <fabric-ca> <thirdparty>
$ curl -sSL http://bit.ly/2ysbOFE | bash -s 1.3.0 1.3.0 0.4.13
```

> **注意**：如果运行上述`curl`命令时出错，则可能是旧版本的`curl`不能处理重定向或不受支持的环境。
>
> 请访问“ [运行环境条件”](https://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html)页面，了解有关在何处查找最新版本`curl`并获取正确环境的其他信息。或者，可以替换未缩短的`URL`：[https](https://raw.githubusercontent.com/hyperledger/fabric/master/scripts/bootstrap.sh):[//raw.githubusercontent.com/hyperledger/fabric/master/scripts/bootstrap.sh](https://raw.githubusercontent.com/hyperledger/fabric/master/scripts/bootstrap.sh)

### 下载二进制文件

```sh
$ cd opt/gopath/src/github.com/hyperledger/fabric-samples
$ wget https://github.com/hyperledger/fabric/blob/master/scripts/bootstrap.sh
$ ./bootstrap.sh

# 或者
$ git clone -b master https://github.com/hyperledger/fabric.git
$ cd opt/gopath/src/github.com/hyperledger/fabric/scripts
$ ./bootstrap.sh
```

**生成必要的二进制文件和配置文件**

```shell
cd /opt/fabric-samples
./bootstrap.sh
```

上面的代码运行完成后，会生成`bin`目录和`config`目录，这两个目录里面的文件是后面示例运行的依赖文件。

上面的命令下载并执行一个`bash`脚本，该脚本将下载并提取设置网络所需的所有特定于平台的二进制文件，并将它们放入在上面创建的克隆仓库中。它检索以下特定于平台的二进制文件：

 - `cryptogen`
 - `configtxgen`
 - `configtxlator`
 - `peer`
 - `orderer`
 - `idemixgen`
 - `fabric-ca-client`

并将它们放在`bin`当前工作目录的子目录中。将其添加到`PATH`环境变量中：

```sh
export PATH=<path to download location>/bin:$PATH
```

 **配置 环境变量** `vi /etc/profile`

```shell
export PATH=$PATH:/opt/gopath/src/github.com/hyperledger/fabric-samples/bin:
```

最后，脚本会将[Docker Hub中](https://hub.docker.com/u/hyperledger/)的`Hyperledger Fabric docker`映像下载到本地`Docker`注册表中，并将其标记为**最新**。

## 运行示例

**生成证书**

```shell
cd /opt/gopath/src/github.com/hyperledger/fabric-samples/first-network
./byfn.sh -m generate
```

**启动网络**

```shell
./byfn.sh -m up
```

执行命令后会提示你确认和取消，输入`y`后，可以看到 `start`。启动完成后可以看到`end`

**卸载网络**

```shell
./byfn.sh -m down
```




