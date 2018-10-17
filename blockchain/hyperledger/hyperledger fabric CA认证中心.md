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
+ `Go` 1.11+
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
>
> **如果切换到`master`分支后，同时更新到最新版本还继续出现错误，需要将 `go` 更新到最新版本。**

## 启动服务器

以下内容使用默认设置启动 `fabric-ca-server`，`-b` 选项为引导程序管理员提供注册`ID`和密码，如果未使用`ldap.enabled` 设置启用`LDAP`，则必须执行此操作。

```sh
$ fabric-ca-server start -b admin:adminpw
```

启动后程序会在本地目录中创建名为 `fabric-ca-server-config.yaml` 的默认配置文件，该文件可以自定义。



## 通过Docker启动服务器

### 选择 docker 镜像

找到与要提取的`fabric-ca`的体系结构和版本相匹配的标记。选择适合环境的 docker 镜像：<https://hub.docker.com/r/hyperledger/fabric-ca/tags/>

进入工作目录到`$GOPATH/src/github.com/hyperledger/fabric-ca/docker/server`并在编辑器中打开`docker-compose.yml`。编辑文件中的 `docker` 镜像版本至自己需要的文件版本。

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

`Fabric CA`提供了3种方法来配置`Fabric CA`服务器和客户端上的设置。配置读取的优先顺序是：

+ `CLI`选项
+ 环境变量
+ 配置文件

**环境变量或`CLI`选项覆盖配置文件更改**。假定客户端配置文件内容如下：

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

同样的方法适用于`fabric-ca-server`，除了使用`FABRIC_CA_CLIENT`作为环境变量的前缀之外，还可以使用`FABRIC_CA_SERVER`。

## 关于文件配置路径

`Fabric CA`服务器和客户机配置文件中指定文件名的所有属性都支持**相对路径和绝对路径**。相对路径相对于配置文件所在的`config`目录。例如，如果`config`目录是`~/config`并且`tls`部分如下所示，则`Fabric CA`服务器或客户端将在`~/config`目录中查找`root.pem`文件，`~/config`中的`cert.pem`文件`/certs`目录和`/abs/path`目录中的`key.pem`文件。

```yaml
tls:
  enabled: true
  certfiles:
    - root.pem
  client:
    certfile: certs/cert.pem
    keyfile: /abs/path/key.pem
```

# Fabric CA Server

可以在启动之前初始化`Fabric CA`服务器。并提供了**生成默认配置文件**的机会，该文件可在启动服务器之前进行查看和自定义。

`Fabric CA`服务器的主目录确定如下：
+ 如果设置了`-home`命令行选项，请使用其值。
+ 否则，如果`FABRIC_CA_SERVER_HOME`设置了环境变量，请使用其值。
+ 否则，如果`FABRIC_CA_HOME`设置了环境变量，则使用其值。
+ 否则，如果`CA_CFG_PATH`设置了环境变量，请使用其值。
+ 否则，使用当前工作目录。

对于此服务器部分的其余部分，我们假设已将`FABRIC_CA_HOME`环境变量设置为`$HOME/fabric-ca/server`。

```sh
$ export FABRIC_CA_HOME=$HOME/fabric-ca/server
```

以下说明假定服务器配置文件存在于服务器的主目录中。

## 初始化服务器

按如下方式初始化`Fabric CA`服务器：

```sh
$ fabric-ca-server init -b admin:adminpw

2018/10/03 10:04:21 [INFO] Created default configuration file at /opt/gopath/src/github.com/hyperledger/fabric-ca/fabric-ca-server-config.yaml
2018/10/03 10:04:21 [INFO] Server Version: 1.3.0
2018/10/03 10:04:21 [INFO] Server Levels: &{Identity:1 Affiliation:1 Certificate:1 Credential:1 RAInfo:1 Nonce:1}
2018/10/03 10:04:21 [WARNING] &{69 The specified CA certificate file /opt/gopath/src/github.com/hyperledger/fabric-ca/ca-cert.pem does not exist}
2018/10/03 10:04:21 [INFO] generating key: &{A:ecdsa S:256}
2018/10/03 10:04:21 [INFO] encoded CSR
2018/10/03 10:04:21 [INFO] signed certificate with serial number 417095774184384307384614229539156324298726356837
2018/10/03 10:04:21 [INFO] The CA key and certificate were generated for CA
2018/10/03 10:04:21 [INFO] The key was stored by BCCSP provider 'SW'
2018/10/03 10:04:21 [INFO] The certificate is at: /opt/gopath/src/github.com/hyperledger/fabric-ca/ca-cert.pem
2018/10/03 10:04:22 [INFO] Initialized sqlite3 database at /opt/gopath/src/github.com/hyperledger/fabric-ca/fabric-ca-server.db
2018/10/03 10:04:22 [INFO] The issuer key was successfully stored. The public key is at: /opt/gopath/src/github.com/hyperledger/fabric-ca/IssuerPublicKey, secret key is at: /opt/gopath/src/github.com/hyperledger/fabric-ca/msp/keystore/IssuerSecretKey
2018/10/03 10:04:22 [INFO] Idemix issuer revocation public and secret keys were generated for CA ''
2018/10/03 10:04:22 [INFO] The revocation key was successfully stored. The public key is at: /opt/gopath/src/github.com/hyperledger/fabric-ca/IssuerRevocationPublicKey, private key is at: /opt/gopath/src/github.com/hyperledger/fabric-ca/msp/keystore/IssuerRevocationPrivateKey
2018/10/03 10:04:22 [INFO] Home directory for default CA: /opt/gopath/src/github.com/hyperledger/fabric-ca
2018/10/03 10:04:22 [INFO] Initialization was successful
```

执行命令后会生成如下文件和目录：

```sh
IssuerPublicKey
IssuerRevocationPublicKey
ca-cert.pem
docker/server/fabric-ca-server/
fabric-ca-server-config.yaml
fabric-ca-server.db
msp/
```

在初始化` ca `命令行中 `$ fabric-ca-server init -b admin:adminpw` ，**`-b`（引导身份）选项是必需的**，`LDAP`初始化时被禁用。启动`Fabric CA`服务器至少需要一个引导程序身份，此身份是**服务器管理员**。服务器配置文件`fabric-ca-server-config.yaml`包含可以配置的证书签名请求（`CSR`）部分。以下是`CSR`示例。

```yaml
csr:
   cn: fabric-ca-server
   names:
      - C: US
        ST: "North Carolina"
        L:
        O: Hyperledger
        OU: Fabric
   hosts:
     - 19b01d2ba729
     - localhost
   ca:
      expiry: 131400h
      pathlength: 1
```

上面的所有字段都与`X.509`签名密钥和由`fabric-ca-server init`生成的证书有关。这对应于服务器配置文件中的`ca.certfile`和`ca.keyfile`文件。字段如下：
+ `cn` 是通用名称
+ `O` 是组织名称
+ `OU` 是组织单位
+ `L` 是地点或城市
+ `ST` 是州
+ `C` 是国家

如果需要`CSR`的自定义值，可以**自定义配置文件**，删除`ca.certfile`和`ca.keyfile`配置项指定的文件，然后再次运行`fabric-ca-server init -b admin:adminpw`命令。

除非指定了`-u <parent-fabric-ca-server-URL>`选项，否则 `fabric-ca-server init` 命令会生成自签名`CA`证书。如果指定了`-u`，则服务器的`CA`证书由父`Fabric CA`服务器签名。要对父`Fabric CA`服务器进行身份验证，`URL`的格式必须为`<scheme>://<enrollmentID>:<secret>@<host>:<port>`，其中`<enrollmentID>`和`<secret>`对应于带有`'hf.IntermediateCA'`属性的身份，其值等于`'true'`。`fabric-ca-server init`命令还在服务器的主目录中生成名为`fabric-ca-server-config.yaml`的默认配置文件。

如果希望`Fabric CA`服务器使用您提供的CA签名证书和密钥文件，则必须将文件分别放在`ca.certfile`和`ca.keyfile`引用的位置。这两个文件都必须是**PEM编码**的，不得加密。更具体地说，CA证书文件的内容必须以`----- BEGIN CERTIFICATE -----`开头，并且密钥文件的内容必须以`----- BEGIN PRIVATE KEY -----`开头，而不是`-----BEGIN ENCRYPTED PRIVATE KEY-----`。

**算法和密钥大小**

可以自定义`CSR`以生成支持椭圆曲线（`ECDSA`）的`X.509`证书和密钥。以下设置是使用曲线`prime256v1`和签名算法`ecdsa-with-SHA256`实现椭圆曲线数字签名算法（`ECDSA`）的示例：

```yaml
key:
   algo: ecdsa
   size: 256
```

算法和密钥大小的选择基于安全需求。椭圆曲线（`ECDSA`）提供以下密钥大小选项：

| size | ASN1 OID     | Signature Algorithm |
| ---- | ------------ | ------------------- |
| 256  | `prime256v1` | `ecdsa-with-SHA256` |
| 384  | `secp384r1`  | `ecdsa-with-SHA384` |
| 521  | `secp521r1`  | `ecdsa-with-SHA512` |

## 启动服务器

启动`Fabric CA`服务器，如下所示：

```sh
$ fabric-ca-server start -b <admin>:<adminpw>
```

如果服务器之前没有被初始化，它将在**第一次启动时自行初始化**。在此初始化期间，服务器将生成`ca-cert.pem`和`ca-key.pem`文件（如果它们尚不存在），并且如果它不存在，还将创建默认配置文件。请参阅初始化`Fabric CA`服务器部分。

除非`Fabric CA`服务器配置为使用`LDAP`，否则**必须至少配置一个预先注册的引导程序身份**，以便注册和注册其他身份。**`-b`选项指定引导身份的名称和密码**。

要使`Fabric CA`服务器侦听`https`而不是`http`，请将`tls.enabled`设置为`true`。

> 安全警告：`Fabric CA`服务器应始终在启用`TLS`的情况下启动（`tls.enabled`设置为`true`）。如果不这样做，服务器就容易受到**访问网络流量的攻击者的攻击**。

要**限制可以使用相同密钥（或密码）进行注册的次数**，请将配置文件中的`registry.maxenrollments`设置为适当的值。如果将值设置为`1`，则`Fabric CA`服务器允许仅对特定注册ID使用一次密码。如果将值设置为`-1`，则`Fabric CA`服务器**不会限制**可以重新使用密钥进行注册的次数。**默认值为-1**。**将值设置为0，`Fabric CA`服务器将禁用所有身份的注册，并且不允许注册身份**。

`Fabric CA`服务器现在应该在端口**7054**上侦听。

## 配置数据库

本节介绍如何配置`Fabric CA`服务器以连接到`PostgreSQL`或`MySQL`数据库。默认数据库是`SQLite`，默认数据库文件是`Fabric CA`服务器主目录中的`fabric-ca-server.db`。

如果在群集中运行`Fabric CA`服务器，必须配置`PostgreSQL`或`MySQL`，如下所述。`Fabric CA`在群集设置中支持以下数据库版本：

- `PostgreSQL`： 9.5.5 or later
- `MySQL`：5.7 or later

## 配置LDAP

`Fabric CA`服务器可以配置为从`LDAP`服务器读取。 特别是，`Fabric CA`服务器可以连接到`LDAP`服务器以执行以下操作：

- 在注册之前验证身份
- 检索用于授权的身份属性值

修改`Fabric CA`服务器配置文件的`LDAP`部分，以将服务器配置为连接到`LDAP`服务器。

```yml
ldap:
   # Enables or disables the LDAP client (default: false)
   enabled: false
   # The URL of the LDAP server
   url: <scheme>://<adminDN>:<adminPassword>@<host>:<port>/<base>
   userfilter: <filter>
   attribute:
      # 'names' is an array of strings that identify the specific attributes
      # which are requested from the LDAP server.
      names: <LDAPAttrs>
      # The 'converters' section is used to convert LDAP attribute values
      # to fabric CA attribute values.
      #
      # For example, the following converts an LDAP 'uid' attribute
      # whose value begins with 'revoker' to a fabric CA attribute
      # named "hf.Revoker" with a value of "true" (because the expression
      # evaluates to true).
      #    converters:
      #       - name: hf.Revoker
      #         value: attr("uid") =~ "revoker*"
      #
      # As another example, assume a user has an LDAP attribute named
      # 'member' which has multiple values of "dn1", "dn2", and "dn3".
      # Further assume the following configuration.
      #    converters:
      #       - name: myAttr
      #         value: map(attr("member"),"groups")
      #    maps:
      #       groups:
      #          - name: dn1
      #            value: orderer
      #          - name: dn2
      #            value: peer
      # The value of the user's 'myAttr' attribute is then computed to be
      # "orderer,peer,dn3".  This is because the value of 'attr("member")' is
      # "dn1,dn2,dn3", and the call to 'map' with a 2nd argument of
      # "group" replaces "dn1" with "orderer" and "dn2" with "peer".
      converters:
        - name: <fcaAttrName>
          value: <fcaExpr>
      maps:
        <mapName>:
            - name: <from>
              value: <to>
```

配置介绍：

- `scheme` 是`ldap`或`ldaps`之一
- `adminDN` 是管理员用户的专有名称
- `pass` 是`admin`用户的密码
- `host` 是`LDAP`服务器的主机名或`IP`地址 
- `port` 是可选的端口号，其中`ldap`的默认值为`389`，`ldap`的默认值为`636`
- `base` 是用于搜索的`LDAP`树的可选根
- `filter` 是搜索将登录用户名转换为可分辨名称时使用的过滤器。例如，值（`uid=％s`）搜索具有`uid`属性值的`LDAP`条目，其值为**登录用户名**。同样，（`email=％s`）可用于使用电子邮件地址登录。
- `LDAPAttrs` 是一个`LDAP`属性名称数组，代表用户从`LDAP`服务器请求
- `attribute.converters`部分用于将`LDAP`属性转换为结构`CA`属性，其中`*fcaAttrName`是结构`CA`属性的名称；`*fcaExpr`是一个表达式，其评估值分配给结构`CA`属性。例如，假设`<LDAPAttrs>`是`['uid']`，`<fcaAttrName>`是`'hf.Revoker'`，而`<fcaExpr>`是`'attr('uid')=~'revoker*'`。这意味着代表用户从`LDAP`服务器请求名为`uid`的属性。如果用户的`'uid'`LDAP属性的值以`'revoker'`开头，则为`'hf.Revoker'`属性赋予用户`'true'`的值；否则，为`'hf.Revoker'`属性赋予用户`'false'`的值。
- `attribute.maps`部分用于映射`LDAP`响应值。典型的用例是将与`LDAP`组关联的可分辨名称映射到身份类型。

`LDAP`表达式语言使用`govaluate`包，如[https://github.com/Knetic/govaluate/blob/master/MANUAL.md](https://github.com/Knetic/govaluate/blob/master/MANUAL.md)。这定义了诸如`=~`之类的运算符和诸如`revoker*`之类的文字，这是一个正则表达式。扩展基本`govaluate`语言的特定于`LDAP`的变量和函数如下：

- `DN` 是一个等于用户专有名称的变量。
- `affiliation` 是一个等于用户所属关系的变量。
- `attr`是一个带有`1`或`2`个参数的函数。第一个参数是`LDAP`属性名称。第二个参数是一个分隔符字符串，用于将多个值连接成一个字符串; 默认的分隔符字符串是`“，”`。该`attr`函数始终返回`string`类型的值。
- `map`是一个带有`2`个参数的函数。第一个参数是任何字符串。第二个参数是映射的名称，用于对第一个参数的字符串执行字符串替换。
- `if`是一个函数，它接受3个参数，其中第一个参数必须解析为布尔值。如果它的计算结果为`true`，则返回第二个参数；否则，返回第三个参数。

例如，如果用户具有以`O=org1，C=US`结尾的可分辨名称，或者如果用户具有以`org1.dept2`开头的关系（`affiliation `），并且还具有`admin`属性，则以下表达式的计算结果为`true`。

```sh
DN =~'*O=org1,C=US' || (affiliation =~ 'org1.dept2.*' && attr('admin') = 'true')
```

注意：由于`attr`函数始终返回`string`类型的值，因此数字运算符不能用于构造表达式。例如，以下内容**不是有效表达式**：

```yml
value: attr("gidNumber") >= 10000 && attr("gidNumber") < 10006
```

或者，如下所示用引号括起来的正则表达式可用于返回等效结果：

```yml
value: attr("gidNumber") =~ "1000[0-5]$" || attr("mail") == "root@example.com"
```

以下是`OpenLDAP`服务器的默认设置的示例配置部分，其`Docker`镜像位于https://github.com/osixia/docker-openldap。

```yml
ldap:
   enabled: true
   url: ldap://cn=admin,dc=example,dc=org:admin@localhost:10389/dc=example,dc=org
   userfilter: (uid=%s)
```

有关启动`OpenLDAP` docker 映像，配置它，在`FABRIC_CA/cli/server/ldap/ldap_test.go`中运行`LDAP`测试并停止`OpenLDAP`服务器的脚本，请参阅`FABRIC_CA/scripts/run-ldap-tests`。

配置`LDAP`时，注册的工作方式如下：

+ `Fabric CA`客户端或客户端`SDK`使用基本授权标头发送注册请求。
+ `Fabric CA`服务器接收注册请求，解码授权头中的身份名称和密码，使用配置文件中的`userfilter`查找与身份名称关联的`DN`（专有名称），然后尝试`LDAP`绑定身份的密码。如果`LDAP`绑定成功，则注册处理已获得授权并可继续。

## 设置群集

可以使用任何`IP sprayer`将负载平衡到`Fabric CA`服务器群集。本节提供了如何设置`Haproxy`以路由到`Fabric CA`服务器群集的示例。请务必更改主机名和端口以反映`Fabric CA`服务器的设置。

`haproxy.conf` 配置如下：

```nginx
global
      maxconn 4096
      daemon

defaults
      mode http
      maxconn 2000
      timeout connect 5000
      timeout client 50000
      timeout server 50000

listen http-in
      bind *:7054
      balance roundrobin
      server server1 hostname1:port
      server server2 hostname2:port
      server server3 hostname3:port
```

注意：如果使用`TLS`，需要使用模式`tcp`。

## 设置多个CA

默认情况下，`fabric-ca`服务器由单个默认CA组成。但是，可以使用`cafiles`或`cacount`配置选项将其他`CA`添加到单个服务器。每个额外的CA都**有自己的主目录**。

**cacount：**

`cacount`提供了一种快速启动`X`个默认附加`CA`的方法。主目录将相对于服务器目录。使用此选项，目录结构如下：

```sh
--<Server Home>
  |--ca
    |--ca1
    |--ca2
```

每个额外的CA都将获得在其主目录中生成的默认配置文件，在配置文件中它将包含唯一的CA名称。例如，以下命令将启动2个默认CA实例：

```sh
$ fabric-ca-server start -b admin:adminpw --cacount 2
```

**cafiles:**

如果使用`cafiles`配置选项时未提供绝对路径，则CA主目录将相对于服务器目录。要使用此选项，**必须已为要启动的每个CA生成并配置CA配置文件**。每个配置文件必须具有**唯一的CA名称和公用名（CN）**，否则服务器将无法启动，因为这些名称必须是唯一的。CA配置文件将覆盖任何默认CA配置，CA配置文件中的任何缺少的选项将替换为默认CA中的值。

配置文件中优先顺序如下：

1. `CA`配置文件
2. 默认`CA` 的命令行选项
3. 默认`CA`环境变量
4. 默认`CA`配置文件

CA配置文件必须至少包含以下内容：

```yaml
ca:
# Name of this CA
name: <CANAME>

csr:
  cn: <COMMONNAME>
```

可以按如下方式配置目录结构：

```sh
--<Server Home>
  |--ca
    |--ca1
      |-- fabric-ca-config.yaml
    |--ca2
      |-- fabric-ca-config.yaml
```

例如，以下命令将启动两个自定义CA实例：

```sh
$ fabric-ca-server start -b admin:adminpw --cafiles ca/ca1/fabric-ca-config.yaml --cafiles ca/ca2/fabric-ca-config.yaml
```

## 注册中间CA

为了为中间CA创建CA签名证书，中间CA必须以与`fabric-ca-client`注册**CA相同的方式向父CA注册**。这是通过使用`-u`选项指定父CA的`URL`以及注册ID和秘钥来完成的，如下所示。与此注册ID关联的身份必须具有名称为`hf.IntermediateCA`且值为`true`的属性。已颁发证书的`CN`（通用名称）将设置为注册`ID`。如果中间CA尝试显式指定`CN`值，则会发生错误。

```sh
$ fabric-ca-server start -b admin:adminpw -u http://<enrollmentID>:<secret>@<parentserver>:<parentport>
```

对于其他中间CA标志，请参阅[Fabric CA服务器的配置文件格式](https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#fabric-ca-server-s-configuration-file-format)部分。

## 升级服务器

在升级`Fabric CA`客户端之前，必须升级`Fabric CA`服务器。在升级之前，建议**备份当前数据库**：

+ 如果使用`sqlite3`，则备份当前数据库文件（默认情况下名为`fabric-ca-server.db`）。
+ 对于其他数据库类型，请使用适当的`备份/复制`机制。

要升级`Fabric CA`服务器的单个实例，需要完成以下步骤：

1. 停止`fabric-ca-server`进程。

2. 确保备份当前数据库。

3. 将先前的`fabric-ca-server`二进制文件替换为升级版本。

4. 启动`fabric-ca-server`进程。

5. 使用以下命令验证`fabric-ca-server`进程是否可用，其中`<host>`是启动服务器的主机名：

   ```sh
   $ fabric-ca-client getcainfo -u http://<host>:7054
   ```

### **升级集群**：

要使用`MySQL`或`Postgres`数据库升级`fabric-ca-server`实例集群，请执行以下过程。我们假设正在使用`haproxy`分别对`host1`和`host2`上的两个`fabric-ca-server`集群成员进行负载均衡，同时监听端口`7054`。在此过程之后，将负载平衡到升级的`fabric-ca-server`集群成员在`host3`和`host4`上分别监听端口`7054`。

要使用`haproxy`统计信息监视更改，请启用统计信息收集。将以下行添加到`haproxy`配置文件的全局部分：

```nginx
stats socket /var/run/haproxy.sock mode 666 level operator
stats timeout 2m
```

重新启动`haproxy`以获取更改：

```sh
$ haproxy -f <configfile> -st $(pgrep haproxy)
```

要显示`haproxy` `“show stat”`命令的摘要信息，以下函数可能对解析返回的大量`CSV`数据非常有用：

```sh
haProxyShowStats() {
   echo "show stat" | nc -U /var/run/haproxy.sock |sed '1s/^# *//'|
      awk -F',' -v fmt="%4s %12s %10s %6s %6s %4s %4s\n" '
         { if (NR==1) for (i=1;i<=NF;i++) f[tolower($i)]=i }
         { printf fmt, $f["sid"],$f["pxname"],$f["svname"],$f["status"],
                       $f["weight"],$f["act"],$f["bck"] }'
}
```

1、最初的`haproxy`配置文件类似于以下内容：

```nginx
server server1 host1:7054 check
server server2 host2:7054 check
```

将此配置更改为以下内容：

```nginx
server server1 host1:7054 check backup
server server2 host2:7054 check backup
server server3 host3:7054 check
server server4 host4:7054 check
```

2、使用新配置重新启动HA代理，如下所示：

```sh
$ haproxy -f <configfile> -st $(pgrep haproxy)
```

`haProxyShowStats`现在将反映修改后的配置，包括两个活动的旧版本备份服务器和两个（尚未启动）升级服务器：

```sh
sid   pxname      svname  status  weig  act  bck
  1   fabric-cas  server3   DOWN     1    1    0
  2   fabric-cas  server4   DOWN     1    1    0
  3   fabric-cas  server1     UP     1    0    1
  4   fabric-cas  server2     UP     1    0    1
```

3、在`host3`和`host4`上安装`fabric-ca-server`的升级二进制文件。`host3`和`host4`上新升级的服务器应配置为使用与`host1`和`host2`上的旧版本相同的数据库。启动升级后的服务器后，将自动迁移数据库。`haproxy`将所有新流量转发到升级后的服务器，因为它们未配置为备份服务器。在继续之前，使用`fabric-ca-client getcainfo`命令验证群集是否仍在正常运行。此外，`haProxyShowStats`现在应该反映所有服务器都处于活动状态，类似于以下内容：

```sh
sid   pxname      svname  status  weig  act  bck
  1   fabric-cas  server3    UP     1    1    0
  2   fabric-cas  server4    UP     1    1    0
  3   fabric-cas  server1    UP     1    0    1
  4   fabric-cas  server2    UP     1    0    1
```

4、停止`host1`和`host2`上的旧服务器。在继续之前，使用`fabric-ca-client getcainfo`命令验证新群集是否仍在正常运行。然后从`haproxy`配置文件中删除旧的服务器备份配置，使其看起来类似于以下内容：

```nginx
server server3 host3:7054 check
server server4 host4:7054 check
```

5、使用新配置重新启动HA代理，如下所示：

```sh
$ haproxy -f <configfile> -st $(pgrep haproxy)
```

`haProxyShowStats` 现在将反映已修改的配置，其中两个活动服务器已升级到新版本：

```sh
sid   pxname      svname  status  weig  act  bck
  1   fabric-cas  server3   UP       1    1    0
  2   fabric-cas  server4   UP       1    1    0
```

# Fabric CA Client

`Fabric CA`客户端的主目录确定如下：
+ 如果设置了`-home`命令行选项，使用其值
+ 否则，如果设置了`FABRIC_CA_CLIENT_HOME`环境变量，使用其值
+ 否则，如果设置了`FABRIC_CA_HOME`环境变量，使用其值
+ 否则，如果设置了`CA_CFG_PATH`环境变量，使用其值
+ 否则，使用`$HOME/.fabric-ca-client`

以下说明假定客户端配置文件存在于客户端的主目录中。

## 注册bootstrap身份

首先，如果需要，请在客户端配置文件中自定义`CSR`（证书签名请求）部分。请注意，必须将`csr.cn`字段设置为引导身份的`ID`。默认`CSR`值如下所示：
```yml
csr:
  cn: <<enrollment ID>>
  key:
    algo: ecdsa
    size: 256
  names:
    - C: US
      ST: North Carolina
      L:
      O: Hyperledger Fabric
      OU: Fabric CA
  hosts:
   - <<hostname of the fabric-ca-client>>
  ca:
    pathlen:
    pathlenzero:
    expiry:
```

然后运行`fabric-ca-client enroll`命令以注册身份。例如，以下命令通过调用在`7054`端口本地运行的`Fabric CA`服务器来注册`ID`为`admin`且密码为`adminpw`的身份。

```sh
$ export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin 
$ fabric-ca-client enroll -u http://admin:adminpw@localhost:7054
```

`enroll`命令在`Fabric CA`客户端的`msp`目录的子目录中存储注册证书（`ECert`）相应的私钥和`CA`证书链`PEM`文件。将看到指示`PEM`文件存储位置的消息。


## 注册新身份

**执行注册请求的身份必须是已注册的，并且还必须具有注册身份类型的适当权限**。特别是，`Fabric CA`服务器在注册期间进行了**三次授权检查**，如下所示：

1、注册服务（即调用者）必须具有`hf.Registrar.Roles`属性，其中包含逗号分隔的值列表，其中一个值等于要注册的身份类型。例如，如果注册商具有值为`peer，app，user`的`hf.Registrar.Roles`属性，则注册商可以注册`peer，app`和`user`类型的身份，但不能注册`orderer`。

2、注册服务的从属关系必须等于所注册身份的从属关系或前缀。例如，具有`“a.b”`附属关系的注册商可以注册具有`“a.b.c”`附属关系的身份，但不能注册具有`“a.c”`附属关系的身份。如果身份需要`root`联属，则联属请求应为点（`“.”`），并且注册商也必须具有`root`隶属关系。如果注册请求中未指定任何从属关系，则注册的身份将被授予注册服务的关联。

3、如果满足以下所有条件，注册服务可以向用户注册属性：

+ 只有在注册商拥有该属性并且它是`hf.Registrar.Attributes`属性值的一部分时，注册商才能注册具有前缀`'hf.'`的`Fabric CA`保留属性。此外，如果属性是类型列表，则注册的属性值必须等于注册器具有的值的一个子集。如果属性的类型为`boolean`，则只有当注册商的属性值为`true`时，注册器才能注册该属性。
+ 注册自定义属性（即名称不以`'hf.'`开头的任何属性）要求注册商具有`'hf.Registar.Attributes'`属性，并且要注册属性或模式的值。唯一支持的模式是末尾带有`“*”`的字符串。例如，`“a.b.*”`是匹配以`“a.b.”`开头的所有属性名称的模式。例如，如果注册商具有`hf.Registrar.Attributes=orgAdmin`，则注册商可以在身份中添加或删除的唯一属性是`“orgAdmin”`属性。
+ 如果请求的属性名称为`“hf.Registrar.Attributes”`，则执行附加检查以查看此属性的请求值是否等于`“hf.Registrar.Attributes”`的注册商值的子集。为此，每个请求的值必须与注册商的`'hf.Registrar.Attributes'`属性值相匹配。例如，如果注册商的`'hf.Registrar.Attributes'`的值是`'a.b.*，x.y.z'`并且所请求的属性值是`'a.b.c，x.y.z'`，则它是有效的，因为`'a.b.c'`匹配`'a.b.*'`并且`'x.y.z'`与注册商的`'x.y.z'`值相匹配。

**例子：**

+ **有效方案**：
  + 如果注册商具有属性`'hf.Registrar.Attributes=a.b.*，x.y.z'`并且正在注册属性`'a.b.c'`，则有效`'a.b.c'`匹配`'a.b.*'`。
  + 如果注册商具有属性`'hf.Registrar.Attributes = a.b.*，x.y.z'`并且正在注册属性`'x.y.z'`，则它是有效的，因为`'x.y.z'`与注册商的`'x.y.z'`值匹配。
  + 如果注册商具有属性`'hf.Registrar.Attributes = a.b.*，x.y.z'`且请求的属性值为`'a.b.c,x.y.z'`，则它是有效的，因为`'a.b.c'`匹配`'a.b.*'`和`'x.y.z'`匹配注册商的`'x.y.z'`值。
  + 如果注册商具有属性`'hf.Registrar.Roles=peer,client'`并且所请求的属性值是`'peer'`或`'peer,client'`，则它是有效的，因为请求的值等于或者是注册服务器的值的子集。

+ **无效方案**
  + 如果注册商具有属性`'hf.Registrar.Attributes = a.b.*，x.y.z'`并且正在注册属性`'hf.Registar.Attributes = a.b.c，x.y.*'`，则它无效，因为请求的属性`'x.y.*'`不是模式由注册商拥有。值`'x.y.*'`是`'x.y.z'`的超集。
  + 如果注册商具有属性`'hf.Registrar.Attributes = a.b.*，x.y.z'`并且正在注册属性`'hf.Registar.Attributes = a.b.c，x.y.z，attr1'`，则它无效，因为注册商的`'hf.Registrar.Attributes'`属性值不包含`'attr1'`。
  + 如果注册商具有属性`'hf.Registrar.Attributes = a.b.*，x.y.z'`并且正在注册属性`'a.b'`，则它是无效的，因为`'a.b.*'`中不包含值`'a.b'`。
  + 如果注册商具有属性`'hf.Registrar.Attributes = a.b.*，x.y.z'`并且正在注册属性`'x.y'`，则它是无效的，因为`'x.y.z'`不包含`'x.y'`。
  + 如果注册商具有属性`'hf.Registrar.Roles = peer，client'`并且所请求的属性值是`'peer，client，orderer'`，则它是无效的，因为注册商在其值中没有`orderer`角色`hf.Registrar.Roles`属性。
  + 如果注册商具有属性`'hf.Revoker = false'`且请求的属性值为`'true'`，则它无效，因为`hf.Revoker`属性是布尔属性，并且注册商的属性值不是`'true'`。

下表列出了可以为身份注册的所有属性。属性的名称区分大小写。

| 名称                         | 类型 | 描述                                                         |
| ---------------------------- | ---- | ------------------------------------------------------------ |
| `hf.Registrar.Roles`         | 名单 | 允许注册服务管理的角色列表                                   |
| `hf.Registrar.DelegateRoles` | 名单 | 注册服务允许注册服务为其“hf.Registrar.Roles”属性提供的角色列表 |
| `hf.Registrar.Attributes`    | 名单 | 允许注册服务注册的属性列表                                   |
| `hf.GenCRL`                  | 布尔 | 如果属性值为true，则Identity可以生成CRL                      |
| `hf.Revoker`                 | 布尔 | 如果属性值为true，则身份可以撤消用户和/或证书                |
| `hf.AffiliationMgr`          | 布尔 | 如果属性值为true，则身份可以管理从属关系                     |
| `hf.IntermediateCA`          | 布尔 | 如果属性值为true，则身份可以注册为中间CA.                    |

> 注意：注册身份时，请指定属性名称和值的数组。如果数组指定具有相同名称的多个数组元素，则当前仅使用最后一个元素。换句话说，目前不支持多值属性。

以下命令使用管理员身份的凭据注册，注册ID为`“admin2”`的新用户，`“org1.department1”`的从属关系，名为`“hf.Revoker”`的属性，值为`“true”`，以及属性名为`“admin”`，值为`“true”`。`“:ecert”`后缀默认情况下，`“admin”`属性及其值将插入用户的注册证书中，然后可用于制定访问控制决策。

```sh
$ export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
$ fabric-ca-client register --id.name admin2 --id.affiliation org1.department1 --id.attrs 'hf.Revoker=true,admin=true:ecert'
```

将会生成密码并打印密码在终端上，也称为注册密码。后续注册身份需要此密码，这允许管理员注册身份并将注册ID和秘钥提供给其他人以注册身份。

可以将多个属性指定为`-id.attrs`标志的一部分，每个属性必须以逗号分隔。对于包含逗号的属性值，必须将该属性封装在双引号中。见下面的例子：

```sh
$ fabric-ca-client register -d --id.name admin2 --id.affiliation org1.department1 --id.attrs '"hf.Registrar.Roles=peer,user",hf.Revoker=true'

# 或者
$ fabric-ca-client register -d --id.name admin2 --id.affiliation org1.department1 --id.attrs '"hf.Registrar.Roles=peer,user"' --id.attrs hf.Revoker=true
```

通过编辑客户端的配置文件为`register`命令中使用的任何字段设置默认值。例如，假设配置文件包含以下内容：

```yml
id:
  name:
  type: user
  affiliation: org1.department1
  maxenrollments: -1
  attributes:
    - name: hf.Revoker
      value: true
    - name: anotherAttrName
      value: anotherAttrValue
```

然后，使用以下命令行注册一个`ID`为`“admin3”`的新身份，其余部分将从包含身份类型的配置文件中获取：`“user”`，`affiliation：“org1.department1”`，以及两个属性：`“hf.Revoker”`和`“anotherAttrName”`。

```sh
$ export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
$ fabric-ca-client register --id.name admin3
```

要注册具有多个属性的身份，需要在配置文件中指定所有属性名称和值，如上所示。

将`maxenrollments`设置为`0`或将其从配置中删除，将导致注册的身份使用CA的**最大注册值**。此外，正在注册的身份的最大注册值不能超过CA的最大注册值。例如，如果CA的最大注册值为`5`，任何新身份的值必须小于或等于`5`，并且也不能将其设置为`-1`（**无限注册**）。

接下来，让我们注册一个对等服务的身份，用于在下一节中注册对等体。以下命令注册`peer1`身份。请注意，我们选择指定自己的密码，而不是让服务器为我们生成一个密码。

```sh
$ export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
$ fabric-ca-client register --id.name peer1 --id.type peer --id.affiliation org1.department1 --id.secret peer1pw
```

请注意，除了服务器配置文件中指定的非叶子节点关联外，关联是区分大小写的，这些关联始终以小写形式存储。例如，如果服务器配置文件的从属关系部分如下所示：

```yaml
affiliations:
  BU1:
    Department1:
      - Team1
  BU2:
    - Department2
    - Department3
```

`BU1`，`Department1`，`BU2`以小写形式存储。这是因为`Fabric CA`使用`Viper`读取配置。`Viper`将映射键视为**不区分大小写，并始终返回小写值**。要向`Team1`联盟注册身份，需要将`bu1.department1.Team1`指定给`-id.affiliation`标志，如下所示：

```sh
$ export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
$ fabric-ca-client register --id.name client1 --id.type client --id.affiliation bu1.department1.Team1
```

## 认证Peer身份

既然已成功注册了对等身份，现在可以在给定注册`ID`和密码的情况下认证对等体。这与注册引导（`bootstrap`）身份类似，不同之处在于我们还演示了如何使用`“-M”`选项填充`Hyperledger Fabric MSP`（成员资格服务提供程序）目录结构。

以下命令认证`peer1`，请务必将`“-M”`选项的值替换为对等方`MSP`目录的路径，目录是对等方`core.yaml`文件中的`“mspConfigPath”`设置。也可以将`FABRIC_CA_CLIENT_HOME`设置为对等体的主目录。

```sh
export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/peer1
fabric-ca-client enroll -u http://peer1:peer1pw@localhost:7054 -M $FABRIC_CA_CLIENT_HOME/msp
```

注册`Orderer`是相同的，除了`MSP`目录的路径是`orderer`的`orderer.yaml`文件中的`“LocalMSPDir”`设置。`fabric-ca-server`颁发的所有注册证书都有组织单位（或简称`OU`），如下所示：

+ `OU`层次结构的根等于身份类型
+ 为身份的所属关系的每个组件添加`OU`

例如，如果身份是`peer`类型且其隶属关系是`department1.team1`，则身份的`OU`层次结构（从`leaf`到`root`）是`OU = team1，OU = department1，OU = peer`。

## 从另一个`Fabric CA`服务器获取`CA`证书链

通常，`MSP`目录的`cacerts`目录必须包含其他证书颁发机构的证书颁发机构链，代表对等方的所有信任根。`fabric-ca-client getcainfo`命令用于从其他`Fabric CA`服务器实例检索这些证书链。

例如，以下内容将在`localhost`上启动第二个`Fabric CA`服务器，侦听端口`7055`，名称为`“CA2”`。这代表完全独立的信任根，并由区块链上的其他成员管理。

```sh
export FABRIC_CA_SERVER_HOME=$HOME/ca2
fabric-ca-server start -b admin:ca2pw -p 7055 -n CA2
```

以下命令将`CA2`的证书链安装到`peer1`的`MSP`目录中

```sh
export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/peer1
fabric-ca-client getcainfo -u http://localhost:7055 -M $FABRIC_CA_CLIENT_HOME/msp
```

默认情况下，`Fabric CA`服务器以子优先顺序返回CA链。这意味着链中的每个CA证书后面都是其颁发者的CA证书。如果需要`Fabric CA`服务器以相反的顺序返回CA链，请将环境变量`CA_CHAIN_PARENT_FIRST`设置为`true`并重新启动`Fabric CA`服务器。`Fabric CA`客户端将适当地处理任一顺序。

## 获取用户的`Identity Mixer`凭据

`Identity Mixer`（`Idemix`）是**一种加密协议套件，用于保护身份验证和传输认证属性**。`Idemix`允许用户在没有发行者（CA）参与的情况下使用验证者进行身份验证，并且仅选择性地仅披露验证者所需的那些属性，并且可以在不跨越其事务的情况下进行链接。

除`X509`证书外，`Fabric CA`服务器还可以颁发`Idemix`凭据。可以通过将请求发送到`/api/v1/idemix/credential` API端点来请求`Idemix`凭证。有关此和其他`Fabric CA`服务器`API`端点的更多信息，请参阅`swagger-fabric-ca.json`。

`Idemix` 凭证颁发是一个两步过程。首先，将带有空主体的请求发送到`/api/v1/idemix/credential` API端点以获取随机数和CA的`Idemix`公钥。其次，使用`nonce`和CA的`Idemix`公钥创建凭证请求，并将正文中的凭证请求发送另一个请求到`/api/v1/idemix/credential` API端点，以获取`Idemix`凭证，凭证撤销信息（`CRI`），和属性名称和值。目前，仅支持三个属性：

- **OU**：用户的组织单位。此属性的值设置为用户的从属关系。例如，如果用户的`affiliaton`是`dept1.unit1`，则`OU`属性设置为`dept1.unit1`
- **IsAdmin**：如果用户是管理员。此属性的值设置为`isAdmin`注册属性的值 
- **EnrollmentID**： 用户的注册ID

可以参考https://github.com/hyperledger/fabric-ca/blob/master/lib/client.go中的`handleIdemixEnroll`函数，以获取获取`Idemix`凭据的两步流程的参考实现。

`/api/v1/idemix/credential` API端点接受基本和令牌授权标头。基本授权标头应包含用户的注册ID和密码。如果用户已具有`X509`注册证书，则还可以使用它来创建令牌授权标头。

> 注意，`Hyperledger Fabric`将支持`客户端/用户`使用`X509`和`Idemix`凭据对事务进行签名，但仅支持`peer`和`orderer`身份的`X509`凭据。和以前一样，应用程序可以使用`Fabric SDK`将请求发送到`Fabric CA`服务器。`SDK`隐藏了与创建授权标头和请求有效负载以及处理响应相关的复杂性。

# 获取`Idemix CRI`（证书撤销信息）

`Idemix CRI`（凭证撤销信息）的目的与`X509 CRL`（证书撤销列表）类似：撤销先前发布的内容。但是，存在一些差异。

在`X509`中，颁发者撤销最终用户的证书，其`ID`包含在`CRL`中。验证程序检查用户的证书是否在`CRL`中，如果是，则返回授权失败。除了从验证者接收授权错误之外，最终用户不参与此撤销过程。

在`Idemix`中，最终用户参与其中。发行人撤销类似于`X509`的最终用户凭证，并且撤销的证据记录在`CRI`中。`CRI`给予最终用户（又名“证明者”）。然后，最终用户根据`CRI`生成证明其凭证未被撤销的证明。最终用户将此证据提供给验证者，验证者根据CRI验证证明。为了验证成功，最终用户和验证者使用的`CRI`版本（称为“纪元”）必须相同。可以通过向`/api/v1/idemix/cri` API端点发送请求来请求最新的`CRI`。

当`fabric-ca-server`收到注册请求并且撤销句柄池中没有剩余撤销句柄时，`CRI`的版本会递增。在这种情况下，`fabric-ca-server`必须生成一个新的撤销句柄池，它会增加`CRI`的纪元。可以通过`idemix.rhpoolsize`服务器配置属性配置吊销句柄池中的吊销句柄数。

## 重新注册身份

假设您的注册证书即将过期或已被盗用。可以按如下方式发出`reenroll`命令以续订注册证书。

```sh
export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/peer1
fabric-ca-client reenroll
```

## 撤销证书或身份

身份或证书可以被撤销。撤消身份将撤销身份所拥有的所有证书，并且还将阻止身份获取任何新证书。撤销证书将使单个证书无效。

为了撤销证书或身份，调用身份必须具有`hf.Revoker`和`hf.Registrar.Roles`属性。撤销身份只能撤销证书或具有与撤销身份的从属关系相同或前缀的关联的身份。此外，`revoker`只能撤销`revoker`的`hf.Registrar.Roles`属性中列出的类型的身份。

例如，具有从属关系`orgs.org1`和`'hf.Registrar.Roles = peer，client'`属性的`revoker`可以撤销与`orgs.org1`或`orgs.org1.department1`关联的对等或客户端类型身份，但不能撤销身份附属于`orgs.org2`或任何其他类型。

以下命令**禁用身份并撤消与身份关联的所有证书**。`Fabric CA`服务器从此身份收到的所有未来请求都将被拒绝。

```sh
$ fabric-ca-client revoke -e <enrollment_id> -r <reason>
```

以下是可以使用`-r`标志指定的受支持原因：

+ `unspecified` 不明
+ `keycompromise`  密钥泄漏
+ `cacompromise` 证书泄露
+ `affiliationchange` 从属关系变更
+ `superseded` 取代
+ `cessationofoperation` 停止操作
+ `certificatehold` 证书持有
+ `removefromcrl` 从 `crl` 删除
+ `privilegewithdrawn` 特权剔除
+ `aacompromise`  泄露

例如，与关联树的根关联的引导管理员可以撤消`peer1`的身份，如下所示：

```sh
export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
fabric-ca-client revoke -e peer1
```

可以通过指定其`AKI`（授权密钥身份符）和序列号来撤销属于身份的注册证书，如下所示：

```sh
$ fabric-ca-client revoke -a xxx -s yyy -r <reason>
```

例如，可以使用`openssl`命令获取证书的`AKI`和序列号，并将它们传递给`revoke`命令以撤销所述证书，如下所示：

```sh
serial=$(openssl x509 -in userecert.pem -serial -noout | cut -d "=" -f 2)
aki=$(openssl x509 -in userecert.pem -text | awk '/keyid/ {gsub(/ *keyid:|:/,"",$1);print tolower($0)}')
$ fabric-ca-client revoke -s $serial -a $aki -r affiliationchange
```

`-gencrl` 标志可用于生成包含所有已撤销证书的`CRL`（证书吊销列表）。例如，以下命令将撤消身份`peer1`，生成`CRL`并将其存储在` <msp folder>/crls/crl.pem `文件中。

```sh
$ fabric-ca-client revoke -e peer1 --gencrl
```

也可以使用`gencrl`命令生成`CRL`。有关`gencrl`命令的更多信息，请参阅生成`CRL`（证书吊销列表）部分。

## 生成CRL（证书吊销列表）

在`Fabric CA`服务器中撤消证书后，还必须更新`Hyperledger Fabric`中的相应`MSP`。这包括对等体的本地`MSP`以及相应通道配置块中的`MSP`。为此，`PEM`编码的`CRL`（证书撤销列表）文件必须放在`MSP`的`crls`文件夹中。`fabric-ca-client gencrl`命令可用于生成`CRL`。具有`hf.GenCRL`属性的任何身份都可以创建一个`CRL`，其中包含在特定时间段内已撤消的所有证书的序列号。创建的`CRL`存储在`<msp folder>/crls/crl.pem`文件中。

以下命令将创建一个包含所有已撤销的证书（已过期和未过期）的`CRL`，并将`CRL`存储在`~/msp/crls/crl.pem`文件中。

```sh
$ export FABRIC_CA_CLIENT_HOME=~/clientconfig
$ fabric-ca-client gencrl -M ~/msp
```

下一个命令将创建一个`CRL`，其中包含在`2017-09-13T16:39:57-08:00`（由`-revokedafter`标志指定）之后和`2017-09-21T16:39:57-08:00 `之前（由`-revokedbefore`标志指定）被撤销的所有证书（已过期和未过期）并将`CRL`存储在`~/msp/crls/crl.pem`文件中。

```sh
$ export FABRIC_CA_CLIENT_HOME=~/clientconfig
$ fabric-ca-client gencrl --caname "" --revokedafter 2017-09-13T16:39:57-08:00 --revokedbefore 2017-09-21T16:39:57-08:00 -M ~/msp
```

`-caname` 标志指定将请求发送到的CA的名称。在此示例中，`gencrl`请求被发送到默认`CA`。

`-revokedafter`和`-revokedbefore`标志指定时间段的下边界和上边界。生成的`CRL`将包含在此时间段内被撤销的证书。值必须是`RFC3339`格式中指定的`UTC`时间戳。`-revokedafter`时间戳不能大于`-revokedbefore`时间戳。

默认情况下，`CRL`的“下次更新”日期设置为第二天。`crl.expiry` CA配置属性可用于指定自定义值。

`gencrl`命令还将接受`-expireafter`和`-expirebefore`标志，这些标志可用于生成具有在这些标志指定的时间段内到期的已撤销证书的`CRL`。例如，以下命令将生成一个`CRL`，其中包含在`2017-09-13T16:39:57-08:00`之后和`2017-09-21T16:39:57-08:00`之前被撤销的证书，并且该证书在之后到期`2017-09-13T16:39:57-08:00`和`2018-09-13T16:39:57-08:00`之前。

```sh
$ export FABRIC_CA_CLIENT_HOME=~/clientconfig
$ fabric-ca-client gencrl --caname "" --expireafter 2017-09-13T16:39:57-08:00 --expirebefore 2018-09-13T16:39:57-08:00  --revokedafter 2017-09-13T16:39:57-08:00 --revokedbefore 2017-09-21T16:39:57-08:00 -M ~/msp
```

`fabric-samples/fabric-ca`示例演示了如何生成包含已撤销用户证书的`CRL`并更新通道`msp`。然后，它将演示使用已撤销的用户凭据查询通道将导致授权错误。

## 启用`TLS`

本节更详细地介绍如何为`Fabric CA`客户端配置`TLS`。可以在`fabric-ca-client-config.yaml`中配置以下部分。

```yml
tls:
  # Enable TLS (default: false)
  enabled: true
  certfiles:
    - root.pem
  client:
    certfile: tls_client-cert.pem
    keyfile: tls_client-key.pem
```

`certfiles`选项是客户端信任的一组根证书。这通常只是在`ca-cert.pem`文件中服务器主目录中找到的根`Fabric CA`服务器证书。仅当在服务器上配置了相互`TLS`时，才需要**客户端**选项。

## 基于属性的访问控制

可以通过链代码（以及`Hyperledger Fabric`运行时）基于身份的属性来做出访问控制决策。这称为基于属性的访问控制，简称`ABAC`。

为了实现这一点，身份的注册证书（`ECert`）可以包含一个或多个属性名称和值。然后，链码提取属性的值以进行访问控制决策。

例如，假设正在开发应用程序`app1`并希望只有`app1`管理员才能访问特定的链代码操作。链代码可以验证调用者的证书（由`CA`信任的`CA`颁发）包含名为`app1Admin`且值为`true`的属性。当然，属性的名称可以是任何值，值不必是布尔值。

那么如何获得具有属性的注册证书？有两种方法：

+ 注册身份时，可以指定为身份颁发的注册证书默认情况下应包含属性。在注册时可以覆盖此行为，但这对于建立默认行为很有用，并且假设注册发生在应用程序之外，则不需要更改任何应用程序。

  以下显示如何使用两个属性注册`user1:app1Admin`和`email`。默认情况下，`“：ecert”`后缀会导致`appAdmin`属性插入到`user1`的注册证书中，此时用户在注册时未明确请求属性。默认情况下，电子邮件属性不会添加到注册证书中。

+ 注册身份时，可以明确请求将一个或多个属性添加到证书中。对于请求的每个属性，可以指定属性是否可选。如果未选择性地请求并且身份不具有该属性，则将发生错误。

  下面显示了如何使用`email`属性注册`user1`，不使用`app1Admin`属性，也可以选择使用`phone`属性（如果用户拥有`phone`属性）。

```sh
$ fabric-ca-client enroll -u http://user1:user1pw@localhost:7054 --enrollment.attrs "email,phone:opt"
```

下表显示了为每个身份自动注册的三个属性。

| 属性名称          | 属性值         |
| ----------------- | -------------- |
| `hf.EnrollmentID` | 身份的注册ID   |
| `hf.Type`         | 身份的类型     |
| `hf.Affiliation`  | 身份的隶属关系 |

要将任何上述属性默认添加到证书，必须使用`“：ecert”`规范显式注册该属性。例如，以下注册身份`'user1'`，以便如果在注册时未请求特定属性，则将`'hf.Affiliation'`属性添加到注册证书。请注意，`'-id.affiliation'`和`'-id.attrs'`标志中的隶属关系（`'org1'`）的值必须相同。

```sh
$ fabric-ca-client register --id.name user1 --id.secret user1pw --id.type user --id.affiliation org1 --id.attrs 'hf.Affiliation=org1:ecert'
```

有关基于属性的访问控制的链代码库API的信息，请参阅https://github.com/hyperledger/fabric/tree/release-1.1/core/chaincode/lib/cid/README.md

有关演示基于属性的访问控制的端到端示例等，请参阅https://github.com/hyperledger/fabric-samples/tree/release-1.1/fabric-ca/README.md

## 动态服务器配置更新

本节介绍如何使用`fabric-ca-client`动态更新`fabric-ca-server`的部分配置，而无需重新启动服务器。本节中的所有命令都要求您首先通过执行`fabric-ca-client enroll`命令注册。

### 动态更新身份

本节介绍如何使用`fabric-ca-client`动态更新身份。如果客户端身份不满足以下所有条件，则会发生授权失败：

+ 客户端身份必须具有`“hf.Registrar.Roles”`属性，其中包含逗号分隔的值列表，其中一个值等于要更新的身份的类型；例如，如果客户端的身份具有值为`“client，peer”`的`“hf.Registrar.Roles”`属性，则客户端可以更新`“client”`和`“peer”`类型的身份，但不能更新`“orderer”`。
+ 客户身份的从属关系必须等于或更新身份的从属关系的前缀。例如，具有`“a.b”`附属关系的客户端可以更新具有`“a.b.c”`附属关系的身份，但不能更新具有`“a.c”`附属关系的身份。如果身份需要`root`联属，则更新请求应为联盟指定点（`“.”`），并且客户端还必须具有`root`隶属关系。

以下显示如何添加，修改和删除联属关系。

### 获取身份信息

只要调用者满足上一节中突出显示的授权要求，调用者就可以从`fabric-ca`服务器检索身份信息。以下命令显示如何获取身份。

```sh
$ fabric-ca-client identity list --id user1
```

调用者还可以通过发出以下命令来请求检索有权查看的所有身份的信息。

```sh
$ fabric-ca-client identity list
```

### 添加身份

以下内容为`“user1”`添加了新身份。添加新身份与通过`“fabric-ca-client register”`命令注册身份执行相同的操作。添加新身份有两种可用的方法：第一种方法是通过`-json`标志，您可以在其中描述`JSON`字符串中的身份：

```sh
$ fabric-ca-client identity add user1 --json '{"secret": "user1pw", "type": "user", "affiliation": "org1", "max_enrollments": 1, "attrs": [{"name": "hf.Revoker", "value": "true"}]}'
```

以下添加具有`root affiliation`的用户。请注意，从属关系名称`“.”`表示根关联。

```sh
$ fabric-ca-client identity add user1 --json '{"secret": "user1pw", "type": "user", "affiliation": ".", "max_enrollments": 1, "attrs": [{"name": "hf.Revoker", "value": "true"}]}'
```

添加身份的第二种方法是使用直接标志，请参阅下面的示例以添加`“user1”`：

```sh
$ fabric-ca-client identity add user1 --secret user1pw --type user --affiliation . --maxenrollments 1 --attrs hf.Revoker=true
```

下表列出了身份的所有字段，以及它们是必需的还是可选的，以及它们可能具有的任何默认值：

| 字段             | 必需 | 默认值               |
| ---------------- | ---- | -------------------- |
| `ID`             | 是   |                      |
| `Secret`         | 没有 |                      |
| `Affiliation`    | 没有 | Caller’s Affiliation |
| `Type`           | 没有 | client               |
| `Maxenrollments` | 没有 | 0                    |
| `Attributes`     | 没有 |                      |

### 修改身份

有两种可用于修改现有身份的方法。第一种方法是通过`-json`标志，可以在其中描述对`JSON`字符串中的身份的修改。可以在单个请求中进行多项修改，未修改的身份的任何元素将保留其原始值。

> 注意：`maxenrollments`值`“-2”`指定要使用CA的最大注册设置。

以下命令使用`-json`标志对身份进行多次修改。

```sh
$ fabric-ca-client identity modify user1 --json '{"secret": "newPassword", "affiliation": ".", "attrs": [{"name": "hf.Regisrar.Roles", "value": "peer,client"},{"name": "hf.Revoker", "value": "true"}]}'
```

以下命令使用直接标志进行修改。以下内容将身份`'user1'`的注册密码更新为`'newsecret'`

```sh
$ fabric-ca-client identity modify user1 --secret newsecret
```

以下内容更新了身份`'user1'`与`'org2'`的关系

```sh
$ fabric-ca-client identity modify user1 --affiliation org2
```

以下内容将身份`'user1'`的类型更新为`'peer'`

```sh
$ fabric-ca-client identity modify user1 --type peer
```

以下内容将身份`'user1'`的`maxenrollments`更新为5

```sh
$ fabric-ca-client identity modify user1 --maxenrollments 5
```

通过指定`maxenrollments`值为`“-2”`，以下内容会导致身份`“user1”`使用CA的最大注册设置

```sh
$ fabric-ca-client identity modify user1 --maxenrollments -2
```

以下将身份`'user1'`的`'hf.Revoker'`属性的值设置为`'false'`。如果身份具有其他属性，则不会更改它们。如果身份先前没有`'hf.Revoker'`属性，则该属性将添加到身份中。也可以通过不为属性指定值来删除属性。

```sh
$ fabric-ca-client identity modify user1 --attrs hf.Revoker=false
```

以下删除了用户`'user1'`的`'hf.Revoker'`属性

```sh
$ fabric-ca-client identity modify user1 --attrs hf.Revoker=
```

以下演示了在单个`fabric-ca-client`身份修改命令中可以使用多个选项。在这种情况下，为用户`'user1'`更新秘钥和类型。

```sh
$ fabric-ca-client identity modify user1 --secret newpass --type peer
```

### 删除身份

以下删除身份`'user1'`并撤消与`'user1'`身份关联的任何证书。

```sh
$ fabric-ca-client identity remove user1
```

> 注意：默认情况下，在`fabric-ca-server`中禁用身份的删除，但可以通过使用`-cfg.identities.allowremove`选项启动`fabric-ca-server`来启用。

## 动态更新附属关系

本节介绍如何使用`fabric-ca-client`动态更新从属关系。以下显示如何添加，修改，删除和列出联属关系。

### 添加关系

如果客户端身份不满足以下所有条件，则会发生授权失败：

+ 客户端身份必须具有值为`'true'`的属性`'hf.AffiliationMgr'`
+ 客户身份的从属关系必须在层级上高于正在更新的关联。例如，如果客户的隶属关系是`“a.b”`，则客户端可以添加从属关系`“a.b.c”`而不是`“a”`或`“a.b”`。

以下添加了一个名为`“org1.dept1”`的新关系：

```sh
$ fabric-ca-client affiliation add org1.dept1
```

### 修改关系

如果客户端身份不满足以下所有条件，则会发生授权失败：

+ 客户端身份必须具有值为`'true'`的属性`'hf.AffiliationMgr'`
+ 客户身份的从属关系必须在层级上高于正在更新的关联。例如，如果客户的隶属关系是`“a.b”`，则客户端可以添加从属关系`“a.b.c”`而不是`“a”`或`“a.b”`
+ 如果`'-force'`选项为`true`且存在必须修改的身份，则还必须授权客户身份修改身份。

以下将`'org2'`从属关系重命名为`'org3'`。它还重命名任何子隶属关系（例如`'org2.department 1'`被重命名为`'org3.department 1'`）。

```sh
$ fabric-ca-client affiliation modify org2 --name org3
```

如果存在受关系重命名影响的身份，除非使用`'-force'`选项，否则将导致错误。使用`'-force'`选项将更新受影响的身份的从属关系以使用新的从属关系名称：

```sh
$ fabric-ca-client affiliation modify org1 --name org2 --force
```

### 删除关系

如果客户端身份不满足以下所有条件，则会发生授权失败：

+ 客户端身份必须具有值为`'true'`的属性`'hf.AffiliationMgr'`。
+ 客户身份的从属关系必须在层级上高于正在更新的关联。例如，如果客户的隶属关系是`“a.b”`，则客户端可以删除从属关系`“a.b.c”`而不是`“a”`或`“a.b”`。
+ 如果`'-force'`选项为`true`且存在必须修改的身份，则还必须授权客户身份修改身份。

以下删除了从属关系`'org2'`以及任何子关联。例如，如果`'org2.dept1'`是`'org2'`下面的联盟，它也会被删除。

```sh
$ fabric-ca-client affiliation remove org2
```

如果存在受删除联盟影响的身份，则除非使用`“-force”`选项，否则将导致错误。使用`'-force'`选项还将删除与该关联关联的所有身份，以及与这些身份中的任何身份相关联的证书。

> 注意：默认情况下，在`fabric-ca-server`中禁用删除附属关系，但可以通过使用`-cfg.affiliations.allowremove`选项启动`fabric-ca-server`来启用。

## 列出关系信息

如果客户端身份不满足以下所有条件，则会发生授权失败：

+ 客户端身份必须具有值为`'true'`的属性`'hf.AffiliationMgr'`。
+ 客户身份的关联必须等于或者等级高于正在更新的关联。例如，如果客户的隶属关系是`“a.b”`，则客户端可以获得关于`“a.b”`或`“a.b.c”`但不是`“a”`或`“a.c”`的关联信息。

以下命令显示如何获取特定的关联：

```sh
$ fabric-ca-client affiliation list --affiliation org2.dept1
```

调用者还可以通过发出以下命令来请求检索其有权查看的所有从属关系的信息。

```sh
$ fabric-ca-client affiliation list
```

## 管理证书

本节介绍如何使用`fabric-ca-client`管理证书

### 列出证书信息

调用者可见的证书包括：

+ 那些属于调用者的证书
+ 如果调用者拥有`hf.Registrar.Roles`属性或值为`true`的`hf.Revoker`属性，则属于调用者所属关系中及其下方的身份的所有证书。例如，如果客户的联盟是`a.b`，则客户可以获得身份证明的证书，其身份是`a.b`或`a.b.c`但不是`a`或`b`。

如果执行请求多个身份证书的列表命令，则只会列出联系人等于或等级低于调用者所属关系的身份证书。可以基于`ID`，`AKI`，序列号，到期时间，撤销时间，未撤销和`notexpired`标志来过滤将列出的证书。

- `id`：列出此注册ID的证书
- `serial`：列出具有此序列号的证书
- `aki`：列出具有此`AKI`的证书
- `expiration`：列出到期日期在此到期时间内的证书
- `revocation`：列出在此吊销时间内撤消的证书
- `notrevoked`：列出尚未撤销的证书
- `notexpired`：列出尚未过期的证书

可以使用标记`notexpired`和`notrevoked`作为过滤器来从结果集中排除已撤销的证书和过期的证书。例如，如果只关心已过期但尚未撤销的证书，则可以使用`expiration`和`notrevoked`标志来获取此类结果。下面提供了这种情况的一个例子。

应根据`RFC3339`指定时间。例如，要列出2018年3月1日下午1:00和2018年6月15日凌晨2:00之间到期的证书，输入时间字符串将显示为`2018-03-01T13:00:00z`和`2018-06-15T02:00:00Z`。如果时间不是问题，只有日期很重要，那么时间部分可以保持关闭，然后字符串变为2018-03-01和2018-06-15。

`now`字符串可用于表示当前时间，空字符串可用于表示任何时间。例如，`now::`表示从现在到未来任何时间的时间范围，`::now`表示从过去到现在的任何时间的时间范围。

以下命令显示如何使用各种筛选器列出证书，列出所有证书：

```sh
$ fabric-ca-client certificate list
```

按ID列出所有证书：

```sh
$ fabric-ca-client certificate list --id admin
```

按序列和`aki`列出证书：

```sh
$ fabric-ca-client certificate list --serial 1234 --aki 1234
```

按ID和`serial/aki`列出证书：

```sh
$ fabric-ca-client certificate list --id admin --serial 1234 --aki 1234
```

列出既不是`revoker`也不是`id`过期的证书：

```sh
$ fabric-ca-client certificate list --id admin --notrevoked --notexpired
```

列出尚未针对`id`（`admin`）撤消的所有证书：

`“-notexpired”`标志相当于`“-expiration now::”`，这意味着证书将在未来某个时间到期。

```sh
$ fabric-ca-client certificate list --id admin --notexpired
```

列出在时间范围内已撤销但尚未为id（`admin`）过期的所有证书：

```sh
$ fabric-ca-client certificate list --id admin --revocation 2018-01-01T01:30:00z::2018-01-30T05:00:00z
```

列出在时间范围内已撤销但尚未为id（`admin`）过期的所有证书：

```sh
$ fabric-ca-client certificate list --id admin --revocation 2018-01-01::2018-01-30 --notexpired
```

使用持续时间（在`30`天和`15`天前撤销）列出所有已撤销的证书（`ID`）（`admin`）：

```sh
$ fabric-ca-client certificate list --id admin --revocation -30d::-15d
```

在一段时间之前列出所有被撤销的证书：

```sh
$ fabric-ca-client certificate list --revocation ::2018-01-30
```

在此之前和特定日期之后列出所有已撤销的证书：

```sh
$ fabric-ca-client certificate list --id admin --revocation 2018-01-30::now
```

列出所有在时间范围之间过期但尚未针对id（`admin`）撤销的证书：

```sh
$ fabric-ca-client certificate list --id admin --expiration 2018-01-01::2018-01-30 --notrevoked
```

列出所有过期的证书使用持续时间（在30天和15天前过期）为id（`admin`）：

```sh
$ fabric-ca-client certificate list --expiration -30d::-15d
```

列出所有已过期或将在特定时间之前过期的证书：

```sh
$ fabric-ca-client certificate list --expiration ::2058-01-30
```

列出所有已过期或将在特定时间后过期的证书：

```sh
$ fabric-ca-client certificate list --expiration 2018-01-30::
```

在此之前和特定日期之后列出所有过期的证书：

```sh
$ fabric-ca-client certificate list --expiration 2018-01-30::now
```

列出未来10天到期的证书：

```sh
$ fabric-ca-client certificate list --id admin --expiration ::+10d --notrevoked
```

`list certificate`命令还可用于在文件系统上存储证书。这是在`MSP`中填充`admins`文件夹的便捷方式，`“-store”`标志指向文件系统上用于存储证书的位置。

通过在`MSP`中存储身份证书，将身份配置为管理员：

```sh
$ export FABRIC_CA_CLIENT_HOME=/tmp/clientHome
$ fabric-ca-client certificate list --id admin --store msp/admincerts
```

## 关联特定的CA实例

当服务器运行多个`CA`实例时，可以将请求定向到特定CA。默认情况下，如果客户端请求中未指定CA名称，则请求将定向到`fabric-ca`服务器上的默认CA。可以使用`caname`过滤器在客户端命令的命令行上指定CA名称，如下所示：

```sh
$ fabric-ca-client enroll -u http://admin:adminpw@localhost:7054 --caname <caname>
```

# HSM

默认情况下，`Fabric CA`服务器和客户端将私钥存储在`PEM`编码的文件中，但它们也可以配置为通过`PKCS11 API`在`HSM`（硬件安全模块）中存储私钥。此行为在服务器或客户端配置文件的`BCCSP`（`BlockChain`加密服务提供程序）部分中配置。

## 配置`Fabric CA`服务器以使用`softhsm2`

本节介绍如何配置`Fabric CA`服务器或客户端以使用名为`softhsm`的`PKCS11`软件版本（请参阅https://github.com/opendnssec/SoftHSMv2）。

安装`softhsm`后，创建一个令牌，将其标记为`“ForFabric”`，将引脚设置为`“98765432”`（请参阅`softhsm`文档）。

可以使用配置文件和环境变量来配置`BCCSP`。例如，如下所示设置`Fabric CA`服务器配置文件的`bccsp`部分。请注意，默认字段的值为`PKCS11`。

```yml
#############################################################################
# BCCSP (BlockChain Crypto Service Provider) section is used to select which
# crypto library implementation to use
#############################################################################
bccsp:
  default: PKCS11
  pkcs11:
    Library: /usr/local/Cellar/softhsm/2.1.0/lib/softhsm/libsofthsm2.so
    Pin: 98765432
    Label: ForFabric
    hash: SHA2
    security: 256
    filekeystore:
      # The directory used for the software file-based keystore
      keystore: msp/keystore
```

可以通过环境变量覆盖相关字段，如下所示：

```sh
FABRIC_CA_SERVER_BCCSP_DEFAULT=PKCS11 FABRIC_CA_SERVER_BCCSP_PKCS11_LIBRARY=/usr/local/Cellar/softhsm/2.1.0/lib/softhsm/libsofthsm2.so 
FABRIC_CA_SERVER_BCCSP_PKCS11_PIN=98765432 FABRIC_CA_SERVER_BCCSP_PKCS11_LABEL=ForFabric
```

# `Fabric-CA` 客户端

`fabric-ca-client` 命令允许管理身份（包括属性管理）和证书（包括续订和撤销）。

## 基本命令行

```sh
Hyperledger Fabric Certificate Authority Client

Usage:
  fabric-ca-client [command]

Available Commands:
  affiliation # 管理从属关系
  certificate # 管理证书
  enroll      # 注册身份
  gencrl      # 生成CRL
  gencsr      # 生成CSR
  getcainfo   # 获取CA证书链和Idemix公钥
  identity    # 管理身份
  reenroll    # 注册重新注册身份
  register    # 注册身份
  revoke      # 撤销身份
  version     # 打印Fabric CA Client版本

Flags:
      --caname string                  # CA的名称
      --csr.cn string                  # 证书签名请求的公用名字段
      --csr.hosts stringSlice          # Slice证书签名请求中以空格分隔的主机名列表
      --csr.keyrequest.algo string     # 指定密钥算法
      --csr.keyrequest.size int        # 指定密钥大小
      --csr.names stringSlice          # Slice格式为<name> = <value>的逗号分隔的CSR名称列表(e.g. C=CA,O=Org1)
      --csr.serialnumber string        # 证书签名请求中的序列号
  -d, --debug                          # 启用调试级别日志记录
      --enrollment.attrs string        # 格式的逗号分隔属性请求列表 <name>[:opt] (e.g. foo,bar:opt)
      --enrollment.label string        # 在HSM操作中使用的标签
      --enrollment.profile string      # 用于颁发证书的签名配置文件的名称
      --enrollment.type string         # 注册请求的类型：'x509'或'idemix'(default "x509")
  -H, --home string                    # 客户端的主目录(default "/home/vagrant/.fabric-ca-client")
      --id.affiliation string          # 身份的隶属关系
      --id.attrs string                # Slice <name> = <value>形式的逗号分隔属性列表 (e.g. foo=foo1,bar=bar1)
      --id.maxenrollments int          # 可以重用密钥以注册的最大次数(default CA's Max Enrollment)
      --id.name string                 # 身份的唯一名称
      --id.secret string               # 正在注册的身份的注册密钥
      --id.type string                 # 正在注册的身份类型(e.g. 'peer, app, user') (default "client")
  -M, --mspdir string                  # 成员资格服务提供程序目录(default "msp")
  -m, --myhost string                  # 注册期间包含在证书签名请求中的主机名(default "ubuntu-xenial")
  -a, --revoke.aki string              # 要撤销的证书的字符串AKI（Authority Key Identifier）
  -e, --revoke.name string             # 应撤销其证书的身份
  -r, --revoke.reason string           # 撤销原因
  -s, --revoke.serial string           # 要撤销的证书的序列号
      --tls.certfiles string           # Slice逗号分隔的PEM编码的可信证书文件列表(e.g. root1.pem,root2.pem)
      --tls.client.certfile string     # 启用相互身份验证时的PEM编码证书文件
      --tls.client.keyfile string      # 字符串启用相互身份验证时的PEM编码密钥文件
  -u, --url string                     # fabric-ca-server的字符串URL(default "http://localhost:7054")
```

# `Fabric-CA` 服务端

`fabric-ca-server` 命令允许初始化和启动一个服务器进程，该进程可以托管一个或多个证书颁发机构。

## 基本命令行

```sh
Hyperledger Fabric 证书颁发机构服务器

Usage:
  fabric-ca-server [command]

Available Commands:
  init        #初始化fabric-ca服务器
  start       #启动fabric-ca服务器
  version     #打印Fabric CA Server版本

Flags:
      --address string   #fabric-ca-server的侦听地址 (default "0.0.0.0")
  -b, --boot string      #用户：传递bootstrap admin，这是构建默认配置文件所必需的
      --ca.certfile string      # PEM编码的CA证书文件(default "ca-cert.pem")
      --ca.chainfile string     # PEM编码的CA链文件(default "ca-chain.pem")
      --ca.keyfile string       # PEM编码的CA密钥文件
  -n, --ca.name string          # 证书颁发机构名称
      --cacount int             # 非默认，CA实例的数量
      --cafiles stringSlice     # Slice以逗号分隔的CA配置文件列表
      --cfg.affiliations.allowremove     # 允许动态删除从属关系
      --cfg.identities.allowremove       # 允许动态删除身份
      --crl.expiry duration              # gencrl请求生成的CRL的到期时间（默认为24h0m0s）
      --crlsizelimit int         # 可接受的CRL的大小限制（以字节为单位）（默认为512000）
      --csr.cn string            # 对父fabric-ca-server的证书签名请求的公用名字段
      --csr.hosts string         # 对父fabric-ca-server的证书签名请求中以空格分隔的主机名列表
      --csr.keyrequest.algo string       # 指定密钥算法
      --csr.keyrequest.size int          # 指定密钥大小
      --csr.serialnumber string  # 对父fabric-ca-server的证书签名请求中的序列号
      --db.datasource string     # 数据库特定的数据源(default "fabric-ca-server.db")
      --db.tls.certfiles string  # Slice以逗号分隔的PEM编码的可信证书文件列表(e.g. root1.pem,root2.pem)
      --db.tls.client.certfile string    # 启用相互身份验证时的PEM编码证书文件
      --db.tls.client.keyfile string     # 启用相互身份验证时的PEM编码密钥文件
      --db.type string                   # 数据库的类型;之一：sqlite3，postgres，mysql (default "sqlite3")
  -d, --debug                            # 启用调试级别日志记录
  -H, --home string                      # Server的主目录(default ".")
      --idemix.nonceexpiration string    # nonce过期的持续时间 (default "15s")
      --idemix.noncesweepinterval string # 删除过期的nonce的间隔 (default "15m")
      --idemix.rhpoolsize int            # 指定吊销句柄池大小(default 100)
      --intermediate.enrollment.label string      # 在HSM操作中使用的标签
      --intermediate.enrollment.profile string    # 用于颁发证书的签名配置文件的名称
      --intermediate.enrollment.type string       # 注册请求的类型：'x509'或'idemix'(default "x509")
      --intermediate.parentserver.caname string   # 要在fabric-ca-server上连接的CA的名称
  -u, --intermediate.parentserver.url string      # l父fabric-ca-server的字符串URL(e.g. http://<username>:<password>@<address>:<port)
      --intermediate.tls.certfiles stringSlice    # Slice逗号分隔的PEM编码的可信证书文件列表(e.g. root1.pem,root2.pem)
      --intermediate.tls.client.certfile string   # 启用相互身份验证时的PEM编码证书文件
      --intermediate.tls.client.keyfile string    # 启用相互身份验证时的PEM编码密钥文件
      --ldap.attribute.names stringSlice          # Slice要在LDAP搜索上请求的LDAP属性的名称
      --ldap.enabled                              # 启用LDAP客户端以进行身份验证和属性
      --ldap.groupfilter string                   # 单个联属组的LDAP组过滤器(default "(memberUid=%s)")
      --ldap.tls.certfiles string                 # Slice逗号分隔的PEM编码的可信证书文件列表(e.g. root1.pem,root2.pem)
      --ldap.tls.client.certfile string           # 启用相互身份验证时的PEM编码证书文件
      --ldap.tls.client.keyfile string            # 启用相互身份验证时的PEM编码密钥文件
      --ldap.url string                           LDAP client URL of form ldap://adminDN:adminPassword@host[:port]/base
      --ldap.userfilter string                    # 搜索用户时使用的LDAP用户过滤器 (default "(uid=%s)")
  -p, --port int                                  # fabric-ca-server的侦听端口 (default 7054)
      --registry.maxenrollments int               # 最大注册人数; 如果未启用LDAP，则有效(default -1)
      --tls.certfile string                       # 服务器侦听端口的PEM编码TLS证书文件(default "tls-cert.pem")
      --tls.clientauth.certfiles stringSlice      # Slice以逗号分隔的PEM编码的可信证书文件列表(e.g. root1.pem,root2.pem)
      --tls.clientauth.type string                # 服务器将遵循TLS客户端身份验证的策略(default "noclientcert")
      --tls.enabled                               # 在侦听端口上启用TLS
      --tls.keyfile string                        # 服务器侦听端口的PEM编码TLS密钥
```

## `fabric-ca-server-config.yaml` 配置详解

```yaml
#############################################################################
#   This is a configuration file for the fabric-ca-server command.
#
#   COMMAND LINE ARGUMENTS AND ENVIRONMENT VARIABLES
#   ------------------------------------------------
#   Each configuration element can be overridden via command line
#   arguments or environment variables.  The precedence for determining
#   the value of each element is as follows:
#   1) command line argument
#      Examples:
#      a) --port 443
#         To set the listening port
#      b) --ca.keyfile ../mykey.pem
#         To set the "keyfile" element in the "ca" section below;
#         note the '.' separator character.
#   2) environment variable
#      Examples:
#      a) FABRIC_CA_SERVER_PORT=443
#         To set the listening port
#      b) FABRIC_CA_SERVER_CA_KEYFILE="../mykey.pem"
#         To set the "keyfile" element in the "ca" section below;
#         note the '_' separator character.
#   3) configuration file
#   4) default value (if there is one)
#      All default values are shown beside each element below.
#
#   FILE NAME ELEMENTS
#   ------------------
#  名称以“file”或“files”结尾的所有字段的值是其他文件的名称或名称。 
#  例如，请参阅“tls.certfile”和“tls.clientauth.certfiles”。 
#  每个字段的值可以是简单文件名，相对路径或绝对路径。 
#  如果该值不是绝对路径，则将其解释为相对于此配置文件的位置。
#############################################################################

# Version of config file
version: 1.3.0

# Server's listening port (default: 7054)
port: 7054

# Enables debug logging (default: false)
debug: false

# Size limit of an acceptable CRL in bytes (default: 512000)
crlsizelimit: 512000

#############################################################################
# 服务器侦听端口的TLS部分
# 客户端身份验证支持以下类型：NoClientCert，
# RequestClientCert，RequireAnyClientCert，VerifyClientCertIfGiven和RequireAndVerifyClientCert。
# Certfiles是服务器在验证客户端证书时使用的根证书颁发机构的列表。
#############################################################################
tls:
  # Enable TLS (default: false)
  enabled: false
  # TLS for the server's listening port
  certfile:
  keyfile:
  clientauth:
    type: noclientcert
    certfiles:

#############################################################################
# CA部分包含与证书颁发机构相关的信息，包括CA的名称，对于区块链网络的所有成员，CA的名称应该是唯一的。 
# 它还包括颁发注册证书（ECerts）和交易证书（TCerts）时使用的密钥和证书文件。 
# 链文件（如果存在）包含应该为此CA信任的证书链，其中链中的第1个始终是根CA证书。
#############################################################################
ca:
  # Name of this CA
  name:
  # Key file (is only used to import a private key into BCCSP)
  keyfile:
  # Certificate file (default: ca-cert.pem)
  certfile:
  # Chain file
  chainfile:

#############################################################################
# gencrl REST端点用于生成包含已吊销证书的CRL。 
# 本节包含gencrl请求处理期间使用的配置选项。
#############################################################################
crl:
  # 指定生成的CRL的到期时间。 
  # 此属性指定的小时数将添加到UTC时间，结果时间用于设置CRL的“下次更新”日期
  expiry: 24h

#############################################################################
# 注册表部分控制fabric-ca-server如何执行两项操作：
# 1）验证包含用户名和密码（也称为注册ID和密码）的注册请求。
# 2）一旦经过身份验证，就会检索fabric-ca-server可选择放入TCerts的身份属性名称和值，然后在TCerts上为Hyperledger Fabric区块链进行交易。
# 这些属性对于在链代码中进行访问控制决策很有用。
# 有两个主要配置选项：
# 1）fabric-ca-server是注册表。 如果下面的ldap部分中的“ldap.enabled”为false，则为true。
# 2）LDAP服务器是注册表，在这种情况下，fabric-ca-server调用LDAP服务器来执行这些任务。
# 如果下面的ldap部分中的“ldap.enabled”为true，则为true，这意味着将忽略此“注册表”部分。
#############################################################################
registry:
  # Maximum number of times a password/secret can be reused for enrollment
  # (default: -1, which means there is no limit)
  maxenrollments: -1

  # Contains identity information which is used when LDAP is disabled
  identities:
     - name: admin
       pass: adminpw
       type: client
       affiliation: ""
       attrs:
          hf.Registrar.Roles: "*"
          hf.Registrar.DelegateRoles: "*"
          hf.Revoker: true
          hf.IntermediateCA: true
          hf.GenCRL: true
          hf.Registrar.Attributes: "*"
          hf.AffiliationMgr: true

#############################################################################
#  Database section
#  支持的类型有：“sqlite3”，“postgres”和“mysql”。
#  dataasource值取决于类型。
#  如果类型为“sqlite3”，则数据源值是用作数据库存储的文件名。 
#  由于“sqlite3”是嵌入式数据库，因此如果要在群集中运行fabric-ca-server，则可能无法使用它。
#  要在群集中运行fabric-ca-server，必须选择“postgres”或“mysql”。
#############################################################################
db:
  type: sqlite3
  datasource: fabric-ca-server.db
  tls:
      enabled: false
      certfiles:
      client:
        certfile:
        keyfile:

#############################################################################
#  LDAP section
#  如果启用了LDAP，则fabric-ca-server将LDAP调用到：
#  1）验证注册请求的注册ID和秘密（即用户名和密码）;
#  2）检索身份属性
#############################################################################
ldap:
   # Enables or disables the LDAP client (default: false)
   # If this is set to true, the "registry" section is ignored.
   enabled: false
   # The URL of the LDAP server
   url: ldap://<adminDN>:<adminPassword>@<host>:<port>/<base>
   # TLS configuration for the client connection to the LDAP server
   tls:
      certfiles:
      client:
         certfile:
         keyfile:
   # Attribute related configuration for mapping from LDAP entries to Fabric CA attributes
   attribute:
      # 'names' is an array of strings containing the LDAP attribute names which are
      # requested from the LDAP server for an LDAP identity's entry
      names: ['uid','member']
      # The 'converters' section is used to convert an LDAP entry to the value of
      # a fabric CA attribute.
      # For example, the following converts an LDAP 'uid' attribute
      # whose value begins with 'revoker' to a fabric CA attribute
      # named "hf.Revoker" with a value of "true" (because the boolean expression
      # evaluates to true).
      #    converters:
      #       - name: hf.Revoker
      #         value: attr("uid") =~ "revoker*"
      converters:
         - name:
           value:
      # The 'maps' section contains named maps which may be referenced by the 'map'
      # function in the 'converters' section to map LDAP responses to arbitrary values.
      # For example, assume a user has an LDAP attribute named 'member' which has multiple
      # values which are each a distinguished name (i.e. a DN). For simplicity, assume the
      # values of the 'member' attribute are 'dn1', 'dn2', and 'dn3'.
      # Further assume the following configuration.
      #    converters:
      #       - name: hf.Registrar.Roles
      #         value: map(attr("member"),"groups")
      #    maps:
      #       groups:
      #          - name: dn1
      #            value: peer
      #          - name: dn2
      #            value: client
      # The value of the user's 'hf.Registrar.Roles' attribute is then computed to be
      # "peer,client,dn3".  This is because the value of 'attr("member")' is
      # "dn1,dn2,dn3", and the call to 'map' with a 2nd argument of
      # "group" replaces "dn1" with "peer" and "dn2" with "client".
      maps:
         groups:
            - name:
              value:

#############################################################################
# Affiliations section. 
# Fabric CA 服务器可以使用本节中指定的附件进行引导。 附属关系被指定为地图。
#
# For example:
#   businessunit1:
#     department1:
#       - team1
#   businessunit2:
#     - department2
#     - department3
#
# Affiliations本质上是分层的。 
# 在上面的例子中，department1（用作businessunit1.department1）是businessunit1的子节点。 
# team1（用作businessunit1.department1.team1）是department1的子级。 
# department2（用作businessunit2.department2）和department3（businessunit2.department3）是businessunit2的子项。
# 注意：除了在配置文件中指定的非叶关联（如businessunit1，department1，businessunit2）之外，
# 关联是区分大小写的，它们始终以小写形式存储。
#############################################################################
affiliations:
   org1:
      - department1
      - department2
   org2:
      - department1

#############################################################################
#  Signing section
#
#  “默认”子部分用于签署注册证书; 
#  默认到期时间（“到期”字段）为“8760h”，即1小时。
#
#  “ca”配置文件子部分用于签署中间CA证书; 默认到期时间（“到期”字段）为“43800h”，即5小时。
#  请注意，“isca”为true，表示它颁发CA证书。 
#  maxpathlen为0表示中间CA无法颁发其他中间CA证书，但仍可以颁发最终实体证书。（参见RFC 5280，第4.2.1.9节）
#
# “tls”配置文件子节用于签署TLS证书请求; 默认到期时间（“到期”字段）为“8760h”，即1小时。
#############################################################################
signing:
    default:
      usage:
        - digital signature
      expiry: 8760h
    profiles:
      ca:
         usage:
           - cert sign
           - crl sign
         expiry: 43800h
         caconstraint:
           isca: true
           maxpathlen: 0
      tls:
         usage:
            - signing
            - key encipherment
            - server auth
            - client auth
            - key agreement
         expiry: 8760h

###########################################################################
# 证书签名请求（CSR）部分。
#
# 这将控制根CA证书的创建。
# 根CA证书的到期时间配置为下面的“ca.expiry”字段，其默认值为“131400h”，即15小时。
# pathlength 字段用于限制CA证书层次结构，如RFC 5280的4.2.1.9节所述。
# 例子：
# 1）没有路径长度值意味着没有请求限制。
# 2）pathlength == 1表示请求的限制为1，这是根CA的默认值。 
# 这意味着根CA可以颁发中间CA证书，但这些中间CA可能不会发出其他CA证书，尽管它们仍然可以颁发最终实体证书。
# 3）pathlength == 0表示请求限制为0; 
# 这是中间CA的默认值，这意味着它仍然无法颁发CA证书，尽管它仍然可以颁发最终实体证书。
###########################################################################
csr:
   cn: fabric-ca-server
   keyrequest:
     algo: ecdsa
     size: 256
   names:
      - C: US
        ST: "North Carolina"
        L:
        O: Hyperledger
        OU: Fabric
   hosts:
     - ubuntu-xenial
     - localhost
   ca:
      expiry: 131400h
      pathlength: 1

###########################################################################
# 每个CA都可以发出X509注册证书以及Idemix Credential。 
# 本节指定负责颁发Idemix凭据的颁发者组件的配置。
###########################################################################
idemix:
  # 指定吊销句柄的池大小。 
  # 撤销句柄是Idemix凭证的唯一身份符。 
  # 颁发者将创建此指定大小的池撤销句柄。
  # 当请求凭证时，发行者将从池中获得处理并将其分配给凭证。 
  # 当使用池中的最后一个句柄时，Issuer将使用新句柄重新填充池。
  # 撤销句柄和凭证撤销信息（CRI）用于由证明者创建非撤销证明，以向验证者证明她的凭证未被撤销。
  rhpoolsize: 1000

  # Idemix凭证颁发是一个两步过程。 
  # 第一步是从发行者获取一个nonce，第二步是使用nonce向isuser发送凭证请求以请求凭证。
  # 此配置属性指定随机数的到期时间。 默认情况下，nonce在15秒后过期。
  # 该值以time.Duration格式表示(see https://golang.org/pkg/time/#ParseDuration).
  nonceexpiration: 15s

  # 指定从数据存储中删除过期的nonce的时间间隔。 默认值为15分钟。
  # 该值以time.Duration格式表示 (see https://golang.org/pkg/time/#ParseDuration)
  noncesweepinterval: 15m

#############################################################################
# BCCSP（BlockChain Crypto Service Provider）部分用于选择要使用的加密库实现
#############################################################################
bccsp:
    default: SW
    sw:
        hash: SHA2
        security: 256
        filekeystore:
            # 用于基于软件文件的密钥库的目录
            keystore: msp/keystore

#############################################################################
# Multi CA section
#
# 默认情况下，每个Fabric CA服务器都包含一个CA. 此部分用于在单个服务器中配置多个CA.
#
# 1) --cacount <number-of-CAs>
# 自动生成<number-of-CAs>非默认CA. 
# 这些额外CA的名称是“ca1”，“ca2”，...“caN”，其中“N”是<CA的数量>
# 这在快速设置多个CA的开发环境中特别有用。 
# 请注意，此配置选项不适用于中间CA服务器
# i.e., 使用intermediate.parentserver.url配置选项启动的结构CA服务器 (-u command line option)
#
# 2) --cafiles <CA-config-files>
# 对于列表中的每个CA配置文件，生成单独的签名CA. 
# 此列表中的每个CA配置文件可以包含除port，debug和tls部分之外的服务器配置文件中的所有相同元素。
#
# Examples:
# fabric-ca-server start -b admin:adminpw --cacount 2
#
# fabric-ca-server start -b admin:adminpw --cafiles ca/ca1/fabric-ca-server-config.yaml
# --cafiles ca/ca2/fabric-ca-server-config.yaml
#
#############################################################################

cacount:

cafiles:

#############################################################################
# Intermediate CA section
#
# 服务器和CA之间的关系如下：
#    1）单个服务器进程可以包含或充当一个或多个CA. 这由上面的“多CA部分”配置。
#    2）每个CA是根CA或中间CA.
#    3）每个中间CA具有父CA，其是根CA或另一中间CA.
#
# 本节涉及＃2和＃3的配置。
# 如果设置了“intermediate.parentserver.url”属性，那么这是具有指定父CA的中间CA.
#
# parentserver section
#    url - 父服务器的URL
#    caname - 要在服务器中注册的CA的名称
#
# enrollment section 用于向父CA注册中间CA.
#    profile - 用于颁发证书的签名配置文件的名称
#    label - 用于HSM操作的标签
#
# tls section for secure socket connection
#   certfiles - PEM编码的受信任根证书文件列表
#   client:
#     certfile - PEM编码的证书文件，用于在服务器上启用客户端身份验证
#     keyfile - 用于在服务器上启用客户端身份验证的PEM编码密钥文件
#############################################################################
intermediate:
  parentserver:
    url:
    caname:

  enrollment:
    hosts:
    profile:
    label:

  tls:
    certfiles:
    client:
      certfile:
      keyfile:
```

