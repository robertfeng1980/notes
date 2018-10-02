# `Fabric-CA` 认证中心

`Hyperledger Fabric  CA`是`Hyperledger Fabric`  的认证中心和证书颁发机构（CA）。下面的介绍了可用于`fabric-ca`客户端和`fabric-ca`服务器的命令。

它提供以下功能：

+ 身份注册，或作为用户注册表连接到`LDAP`
+ 签发注册证书（`ECerts`）
+ 证书续签和撤销


# 概述

下图说明了Hyperledger Fabric CA服务器如何适应整个Hyperledger Fabric架构。

![_images /ç"ç©ca.png](https://hyperledger-fabric-ca.readthedocs.io/en/release-1.1/_images/fabric-ca.png)

有两种与`Hyperledger Fabric CA`服务器交互的方式：通过`Hyperledger Fabric CA`客户端或通过其中一个`Fabric SDK`。与`Hyperledger Fabric CA`服务器的所有通信都是通过`REST API`进行的。有关这些`REST API`的swagger文档，请参阅`fabric-ca/swagger/swagger-fabric-ca.json`。

`Hyperledger Fabric CA`客户端或`SDK`可以连接到`Hyperledger Fabric CA`服务器集群中的服务器。这在图的右上部分中说明。客户端路由到`HA`代理端点，通过`HA`负载平衡到其中一个`fabric-ca-server`集群成员的节点。

集群中的所有`Hyperledger Fabric CA`服务器共享相同的数据库，以跟踪身份和证书。如果配置了`LDAP`，则身份信息将保留在`LDAP`而不是数据库中。

服务器可能包含多个CA， 每个CA都是根CA或中间CA， 每个中间CA都有一个父CA，它是根CA或另一个中间CA。

# 入门

## 环境

+ `Go` 1.9+
+ `docker` 17.03+
+ `docker-compose` 1.11+
+ `libtool`、`libtdhl-dev` 软件包

以下是`Ubuntu`上的安装`libtool`依赖项：

```sh
$ sudo apt install libtool libltdl-dev
```

## 安装

在 `$GOPATH/bin` 目录中安装 `fabric-ca-server`和`fabric-ca-client`二进制文件。

```sh
$ go get -u github.com/hyperledger/fabric-ca/cmd/...
```

> 注意：如果您已经克隆了`fabric-ca`存储库，请确保在运行上面的`'go get'`命令之前，您位于主分支上。否则，您可能会看到错误。

## 启动服务器

以下内容使用默认设置启动 `fabric-ca-server`

```sh
$ fabric-ca-server start -b admin:adminpw
```

`-b` 选项为引导程序管理员提供注册`ID`和密码，如果未使用`ldap.enabled` 设置启用`LDAP`，则必须执行此操作。

在本地目录中创建名为 `fabric-ca-server-config.yaml` 的默认配置文件，该文件可以自定义。

## 通过Docker启动服务器

### 选择 docker 镜像

找到与要提取的`fabric-ca`的体系结构和版本相匹配的标记。选择适合环境的 docker 镜像：<https://hub.docker.com/r/hyperledger/fabric-ca/tags/>

进入工作目录到`$GOPATH/src/github.com/hyperledger/fabric-ca/docker/server`并在编辑器中打开`docker-compose.yml`。编辑文件中的docker 镜像版本至自己需要的文件版本。

```yml
fabric-ca-server:
   image: hyperledger/fabric-ca:1.2.0
   container_name: fabric-ca-server
   ports:
     - "7054:7054"
   environment:
     - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
   volumes:
     - "./fabric-ca-server:/etc/hyperledger/fabric-ca-server"
   command: sh -c 'fabric-ca-server start -b admin:adminpw'
```

在与`docker-compose.yml`文件相同的目录中打开终端并执行以下命令：

```sh
$ docker-compose up -d
```

这样就可以通过`docker compose`拉取指定的`fabric-ca`图像（如果它尚不存在），并启动`fabric-ca`服务器的实例。

### 构建自己的Docker镜像

可以通过`docker-compose`构建和启动服务器，如下所示：

```sh
$ cd $GOPATH/src/github.com/hyperledger/fabric-ca
$ make docker

$ cd docker/server
$ docker-compose up -d
```

`hyperledger/fabric-ca docker` 镜像包含`fabric-ca-server`和`fabric-ca-client`。

```sh
$ cd $GOPATH/src/github.com/hyperledger/fabric-ca
$ FABRIC_CA_DYNAMIC_LINK=true make docker
$ cd docker/server
$ docker-compose up -d
```

## 配置设置

Fabric CA提供了3种方法来配置Fabric CA服务器和客户端上的设置。优先顺序是：
+ 1. `CLI`选项
+ 2. 环境变量
+ 3. 配置文件

环境变量或`CLI`选项覆盖配置文件更改。

假定客户端配置文件内容如下：

```yml
tls:
  # Enable TLS (default: false)
  enabled: false

  # TLS for the client's listenting port (default: false)
  certfiles:
  client:
    certfile: cert.pem
    keyfile:
```

通过环境变量覆盖配置文件中的`certfile`值，可以通过以下方式：

```sh
$ export FABRIC_CA_CLIENT_TLS_CLIENT_CERTFILE=cert2.pem
```

通过命令行选项方式覆盖配置文件，可以通过以下方式：

```sh
$ fabric-ca-client enroll --tls.client.certfile cert3.pem
```

同样的方法适用于`fabric-ca-server`，除了使用`FABRICC_CA_CLIENT`作为环境变量的前缀之外，还可以使用`FABRIC_CA_SERVER`。

## 关于文件配置路径

`Fabric CA`服务器和客户机配置文件中指定文件名的所有属性都支持**相对路径和绝对路径**。相对路径相对于配置文件所在的`config`目录。例如，如果`config`目录是`~/ config`并且`tls`部分如下所示，则`Fabric CA`服务器或客户端将在`~/config`目录中查找`root.pem`文件，`~/ config`中的`cert.pem`文件`/certs`目录和`/abs/path`目录中的`key.pem`文件。

```sh
tls:
  enabled: true
  certfiles:
    - root.pem
  client:
    certfile: certs/cert.pem
    keyfile: /abs/path/key.pem
```

# Fabric CA Server

可以在启动之前初始化`Fabric CA`服务器。为提供了生成默认配置文件的机会，该文件可在启动服务器之前进行查看和自定义。

- `Fabric CA`服务器的主目录确定如下：

  如果设置了`-home`命令行选项，请使用其值。否则，如果`FABRIC_CA_SERVER_HOME`设置了环境变量，请使用其值。否则，如果`FABRIC_CA_HOME`设置了环境变量，则使用其值。否则，如果`CA_CFG_PATH`设置了环境变量，请使用其值。否则，使用当前工作目录

对于此服务器部分的其余部分，我们假设已将`FABRIC_CA_HOME`环境变量设置为`$HOME/fabric-ca/server`。

以下说明假定服务器配置文件存在于服务器的主目录中。







## `Fabric-CA` 客户端

`fabric-ca-client` 命令允许管理身份（包括属性管理）和证书（包括续订和撤销）。

```sh
Hyperledger Fabric Certificate Authority Client

Usage:
  fabric-ca-client [command]

Available Commands:
  affiliation Manage affiliations
  enroll      Enroll an identity
  gencrl      Generate a CRL
  gencsr      Generate a CSR
  getcacert   Get CA certificate chain
  identity    Manage identities
  reenroll    Reenroll an identity
  register    Register an identity
  revoke      Revoke an identity
  version     Prints Fabric CA Client version

Flags:
      --caname string                  Name of CA
      --csr.cn string                  The common name field of the certificate signing request
      --csr.hosts stringSlice          A list of space-separated host names in a certificate signing request
      --csr.names stringSlice          A list of comma-separated CSR names of the form <name>=<value> (e.g. C=CA,O=Org1)
      --csr.serialnumber string        The serial number in a certificate signing request
  -d, --debug                          Enable debug level logging
      --enrollment.attrs stringSlice   A list of comma-separated attribute requests of the form <name>[:opt] (e.g. foo,bar:opt)
      --enrollment.label string        Label to use in HSM operations
      --enrollment.profile string      Name of the signing profile to use in issuing the certificate
  -H, --home string                    Client's home directory (default "$HOME/.fabric-ca-client")
      --id.affiliation string          The identity's affiliation
      --id.attrs stringSlice           A list of comma-separated attributes of the form <name>=<value> (e.g. foo=foo1,bar=bar1)
      --id.maxenrollments int          The maximum number of times the secret can be reused to enroll (default CA's Max Enrollment)
      --id.name string                 Unique name of the identity
      --id.secret string               The enrollment secret for the identity being registered
      --id.type string                 Type of identity being registered (e.g. 'peer, app, user') (default "client")
  -M, --mspdir string                  Membership Service Provider directory (default "msp")
  -m, --myhost string                  Hostname to include in the certificate signing request during enrollment (default "$HOSTNAME")
  -a, --revoke.aki string              AKI (Authority Key Identifier) of the certificate to be revoked
  -e, --revoke.name string             Identity whose certificates should be revoked
  -r, --revoke.reason string           Reason for revocation
  -s, --revoke.serial string           Serial number of the certificate to be revoked
      --tls.certfiles stringSlice      A list of comma-separated PEM-encoded trusted certificate files (e.g. root1.pem,root2.pem)
      --tls.client.certfile string     PEM-encoded certificate file when mutual authenticate is enabled
      --tls.client.keyfile string      PEM-encoded key file when mutual authentication is enabled
  -u, --url string                     URL of fabric-ca-server (default "http://localhost:7054")

Use "fabric-ca-client [command] --help" for more information about a command.
```



## `Fabric-CA` 服务端

`fabric-ca-server` 命令允许初始化和启动一个服务器进程，该进程可以托管一个或多个证书颁发机构。

```sh
Hyperledger Fabric Certificate Authority Server

Usage:
  fabric-ca-server [command]

Available Commands:
  init        Initialize the fabric-ca server
  start       Start the fabric-ca server
  version     Prints Fabric CA Server version

Flags:
      --address string                            Listening address of fabric-ca-server (default "0.0.0.0")
  -b, --boot string                               The user:pass for bootstrap admin which is required to build default config file
      --ca.certfile string                        PEM-encoded CA certificate file (default "ca-cert.pem")
      --ca.chainfile string                       PEM-encoded CA chain file (default "ca-chain.pem")
      --ca.keyfile string                         PEM-encoded CA key file
  -n, --ca.name string                            Certificate Authority name
      --cacount int                               Number of non-default CA instances
      --cafiles stringSlice                       A list of comma-separated CA configuration files
      --cfg.affiliations.allowremove              Enables removal of affiliations dynamically
      --cfg.identities.allowremove                Enables removal of identities dynamically
      --crl.expiry duration                       Expiration for the CRL generated by the gencrl request (default 24h0m0s)
      --crlsizelimit int                          Size limit of an acceptable CRL in bytes (default 512000)
      --csr.cn string                             The common name field of the certificate signing request to a parent fabric-ca-server
      --csr.hosts stringSlice                     A list of space-separated host names in a certificate signing request to a parent fabric-ca-server
      --csr.serialnumber string                   The serial number in a certificate signing request to a parent fabric-ca-server
      --db.datasource string                      Data source which is database specific (default "fabric-ca-server.db")
      --db.tls.certfiles stringSlice              A list of comma-separated PEM-encoded trusted certificate files (e.g. root1.pem,root2.pem)
      --db.tls.client.certfile string             PEM-encoded certificate file when mutual authenticate is enabled
      --db.tls.client.keyfile string              PEM-encoded key file when mutual authentication is enabled
      --db.type string                            Type of database; one of: sqlite3, postgres, mysql (default "sqlite3")
  -d, --debug                                     Enable debug level logging
  -H, --home string                               Server's home directory (default "/etc/hyperledger/fabric-ca")
      --intermediate.enrollment.label string      Label to use in HSM operations
      --intermediate.enrollment.profile string    Name of the signing profile to use in issuing the certificate
      --intermediate.parentserver.caname string   Name of the CA to connect to on fabric-ca-server
  -u, --intermediate.parentserver.url string      URL of the parent fabric-ca-server (e.g. http://<username>:<password>@<address>:<port)
      --intermediate.tls.certfiles stringSlice    A list of comma-separated PEM-encoded trusted certificate files (e.g. root1.pem,root2.pem)
      --intermediate.tls.client.certfile string   PEM-encoded certificate file when mutual authenticate is enabled
      --intermediate.tls.client.keyfile string    PEM-encoded key file when mutual authentication is enabled
      --ldap.attribute.names stringSlice          The names of LDAP attributes to request on an LDAP search
      --ldap.enabled                              Enable the LDAP client for authentication and attributes
      --ldap.groupfilter string                   The LDAP group filter for a single affiliation group (default "(memberUid=%s)")
      --ldap.tls.certfiles stringSlice            A list of comma-separated PEM-encoded trusted certificate files (e.g. root1.pem,root2.pem)
      --ldap.tls.client.certfile string           PEM-encoded certificate file when mutual authenticate is enabled
      --ldap.tls.client.keyfile string            PEM-encoded key file when mutual authentication is enabled
      --ldap.url string                           LDAP client URL of form ldap://adminDN:adminPassword@host[:port]/base
      --ldap.userfilter string                    The LDAP user filter to use when searching for users (default "(uid=%s)")
  -p, --port int                                  Listening port of fabric-ca-server (default 7054)
      --registry.maxenrollments int               Maximum number of enrollments; valid if LDAP not enabled (default -1)
      --tls.certfile string                       PEM-encoded TLS certificate file for server's listening port (default "tls-cert.pem")
      --tls.clientauth.certfiles stringSlice      A list of comma-separated PEM-encoded trusted certificate files (e.g. root1.pem,root2.pem)
      --tls.clientauth.type string                Policy the server will follow for TLS Client Authentication. (default "noclientcert")
      --tls.enabled                               Enable TLS on the listening port
      --tls.keyfile string                        PEM-encoded TLS key for server's listening port

Use "fabric-ca-server [command] --help" for more information about a command.
```