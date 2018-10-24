# `hyperledger fabric` 入门
检查是否已在将要开发区块链应用程序和运行`Hyperledger Fabric`的平台上安装了所有[必要环境](https://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html)。

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

> 以下软件都是在Linux/ Ubuntu 16.04 系统上进行安装，首先请安装好系统。

- [ ] 安装`cURL` 工具：https://curl.haxx.se/download.html

- [ ] 安装`Docker 和 Docker Compose`：https://www.docker.com/get-docker
  - `docker` 安装完成后可以用 `docker --version` 检查docker的版本
  - `docker compose` 安装完成后可以用`docker-compose --version` 查看版本号

- [ ] 安装`Go` 语言环境的安装：https://golang.org/dl/
  - 版本必须在`1.10.x`以上
  - 环境变量配置：`export GOPATH=$HOME/go` 和 `export PATH=$PATH:$GOPATH/bin`

- [x] nodejs 和 npm 的安装：https://nodejs.org/en/download/
  - npm的安装 `npm install npm@5.6.0 -g`

- [x] python 的安装

  - `sudo apt-get install python`





## cURL 安装

> cURL 是一个下载工具，支持HTTP、HTTPS、FTP、FTPS、DICT、TELNET、LDAP、FILE和GOPHER。利用[URL](https://baike.baidu.com/item/URL)语法在命令行方式下工作的开源文件传输工具。它被广泛应用在[Unix](https://baike.baidu.com/item/Unix)、多种[Linux](https://baike.baidu.com/item/Linux)发行版中，并且有[DOS](https://baike.baidu.com/item/DOS/32025)和[Win32](https://baike.baidu.com/item/Win32)、Win64下的移植版本。

### 下载cURL
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

# 下载和运行示例代码

> 将示例代码下载下来，并且解压安装到 `opt/fabric-samples`目录下

## 下载代码

```shell
git clone -b master https://github.com/hyperledger/fabric-samples.git
cd fabric-samples
git checkout v1.1.0-preview

cd /opt/fabric-samples
wget https://github.com/hyperledger/fabric/blob/master/scripts/bootstrap.sh
```



## 运行示例

**生成必要的二进制文件和配置文件**

```shell
cd /opt/fabric-samples
./bootstrap.sh
```

> 上面的代码运行完成后，会生成`bin`目录和`config`目录,这两个目录里面的文件是后面示例运行的依赖文件



 **配置 环境变量** `vi /etc/profile`

```shell
export PATH=$PATH:/opt/fabric-samples/bin:
```



**生成证书**

```shell
cd /opt/fabric-samples/first-network
./byfn.sh -m generate
```



**启动网络**

```shell
./byfn.sh -m up
```

> 执行命令后会提示你确认和取消，输入`y`后，可以看到 `start`。启动完成后可以看到`end`



**停止网络**

```shell
./byfn.sh -m down
```



# Crypto 加密器的使用

> 加密器会为我们生成MSP必须的证书和秘钥（x509证书和签名密钥），这些证书是身份的代表，它允许我们的实体在交易时进行验证身份和签名。
>
> 它会为组织生成证书，每个组织都有ca证书，将绑定到 排序服务ordering和对等节点 peer上进行使用。它会生成公钥和私钥，方便在超级账本中进行通信和交易。




