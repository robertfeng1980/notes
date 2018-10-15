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
> 如果切换到`master`分支后，同时更新到最新版本还继续出现错误，需要将 `go` 更新到最新版本。

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
+ `CLI`选项
+ 环境变量
+ 配置文件

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

`Fabric CA`服务器和客户机配置文件中指定文件名的所有属性都支持**相对路径和绝对路径**。相对路径相对于配置文件所在的`config`目录。例如，如果`config`目录是`~/config`并且`tls`部分如下所示，则`Fabric CA`服务器或客户端将在`~/config`目录中查找`root.pem`文件，`~/config`中的`cert.pem`文件`/certs`目录和`/abs/path`目录中的`key.pem`文件。

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

`Fabric CA`服务器的主目录确定如下：
+ 如果设置了`-home`命令行选项，请使用其值。
+ 否则，如果`FABRIC_CA_SERVER_HOME`设置了环境变量，请使用其值。
+ 否则，如果`FABRIC_CA_HOME`设置了环境变量，则使用其值。
+ 否则，如果`CA_CFG_PATH`设置了环境变量，请使用其值。
+ 否则，使用当前工作目录。

对于此服务器部分的其余部分，我们假设已将`FABRIC_CA_HOME`环境变量设置为`$HOME/fabric-ca/server`。

以下说明假定服务器配置文件存在于服务器的主目录中。

## 初始化服务器

按如下方式初始化Fabric CA服务器：

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

在初始化` ca `命令行中 `$ fabric-ca-server init -b admin:adminpw` ，`-b`（引导标识）选项是必需的，`LDAP`初始化时被禁用。启动Fabric CA服务器至少需要一个引导程序标识。此身份是服务器管理员。

服务器配置文件`fabric-ca-server-config.yaml`包含可以配置的证书签名请求（`CSR`）部分。以下是`CSR`示例。

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

除非指定了`-u <parent-fabric-ca-server-URL>`选项，否则 `fabric-ca-server init` 命令会生成自签名`CA`证书。如果指定了`-u`，则服务器的`CA`证书由父`Fabric CA`服务器签名。要对父`Fabric CA`服务器进行身份验证，`URL`的格式必须为`<scheme>://<enrollmentID>:<secret>@<host>:<port>`，其中`<enrollmentID>`和`<secret>`对应于带有`'hf.IntermediateCA'`属性的标识，其值等于`'true'`。`fabric-ca-server init`命令还在服务器的主目录中生成名为`fabric-ca-server-config.yaml`的默认配置文件。

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

除非`Fabric CA`服务器配置为使用`LDAP`，否则必须至少配置一个预先注册的引导程序标识，以便注册和注册其他标识。**`-b`选项指定引导标识的名称和密码**。

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
- 检索用于授权的标识属性值

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
- `attribute.maps`部分用于映射`LDAP`响应值。典型的用例是将与`LDAP`组关联的可分辨名称映射到标识类型。

`LDAP`表达式语言使用`govaluate`包，如[https://github.com/Knetic/govaluate/blob/master/MANUAL.md](https://github.com/Knetic/govaluate/blob/master/MANUAL.md)。这定义了诸如`=〜`之类的运算符和诸如`revoker*`之类的文字，这是一个正则表达式。扩展基本`govaluate`语言的特定于`LDAP`的变量和函数如下：

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

每个额外的CA都将获得在其主目录中生成的默认配置文件，在配置文件中它将包含唯一的CA名称。

例如，以下命令将启动2个默认CA实例：

```sh
$ fabric-ca-server start -b admin:adminpw --cacount 2
```

**cafiles:**

如果使用`cafiles`配置选项时未提供绝对路径，则CA主目录将相对于服务器目录。

要使用此选项，**必须已为要启动的每个CA生成并配置CA配置文件**。每个配置文件必须具有**唯一的CA名称和公用名（CN）**，否则服务器将无法启动，因为这些名称必须是唯一的。CA配置文件将覆盖任何默认CA配置，CA配置文件中的任何缺少的选项将替换为默认CA中的值。

配置文件中优先顺序如下：

1. `CA`配置文件
2. 默认`CA` CLI标志
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
$ fabric-ca-server start -b admin:adminpw --cafiles ca/ca1/fabric-ca-config.yaml
--cafiles ca/ca2/fabric-ca-config.yaml
```

## 注册中间CA

为了为中间CA创建CA签名证书，中间CA必须以与`Fabric-ca-client`注册**CA相同的方式向父CA注册**。这是通过使用`-u`选项指定父CA的`URL`以及注册ID和秘钥来完成的，如下所示。与此注册ID关联的标识必须具有名称为`hf.IntermediateCA`且值为`true`的属性。已颁发证书的`CN`（或通用名称）将设置为注册`ID`。如果中间CA尝试显式指定`CN`值，则会发生错误。

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

## 注册bootstrap标识

首先，如果需要，请在客户端配置文件中自定义`CSR`（证书签名请求）部分。请注意，必须将`csr.cn`字段设置为引导标识的`ID`。默认`CSR`值如下所示：
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

然后运行`fabric-ca-client enroll`命令以注册身份。例如，以下命令通过调用在`7054`端口本地运行的`Fabric CA`服务器来注册`ID`为`admin`且密码为`adminpw`的标识。

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

注意：注册身份时，请指定属性名称和值的数组。如果数组指定具有相同名称的多个数组元素，则当前仅使用最后一个元素。换句话说，目前不支持多值属性。

以下命令使用管理员标识的凭据注册，注册ID为`“admin2”`的新用户，`“org1.department1”`的从属关系，名为`“hf.Revoker”`的属性，值为`“true”`，以及属性名为`“admin”`，值为`“true”`。`“:ecert”`后缀默认情况下，`“admin”`属性及其值将插入用户的注册证书中，然后可用于制定访问控制决策。

```sh
$ export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
$ fabric-ca-client register --id.name admin2 --id.affiliation org1.department1 --id.attrs 'hf.Revoker=true,admin=true:ecert'
```

将打印密码，也称为注册密码。注册身份需要此密码，这允许管理员注册身份并将注册ID和秘钥提供给其他人以注册身份。

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

然后，使用以下命令行注册一个`ID`为`“admin3”`的新标识，其余部分将从包含标识类型的配置文件中获取：`“user”`，`affiliation：“org1.department1”`，以及两个属性：`“hf.Revoker”`和`“anotherAttrName”`。

```sh
$ export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
$ fabric-ca-client register --id.name admin3
```

要注册具有多个属性的标识，需要在配置文件中指定所有属性名称和值，如上所示。

将`maxenrollments`设置为0或将其从配置中删除，将导致注册的标识使用CA的最大注册值。此外，正在注册的身份的最大注册值不能超过CA的最大注册值。例如，如果CA的最大注册值为5，任何新标识的值必须小于或等于5，并且也不能将其设置为-1（无限注册）。

接下来，让我们注册一个对等服务的身份，用于在下一节中注册对等体。以下命令注册`peer1`标识。请注意，我们选择指定自己的密码，而不是让服务器为我们生成一个密码。

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