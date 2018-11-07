# `hyperledger fabric` 第一个示例

首先，如果还不熟悉`Fabric`网络的基本架构，则可能需要在继续之前访问 [简介](https://hyperledger-fabric.readthedocs.io/en/latest/blockchain.html) 和 [构建您的第一个网络](https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html) 文档。

其次，在文将介绍一些示例程序，**以了解`Fabric`应用程序的工作原理**。这些应用程序（以及他们使用的智能合约）统称为 `fabcar`提供了`Fabric`功能的广泛演示。值得注意的是，我们将展示**与证书颁发机构进行交互并生成注册证书**的过程，之后我们将**利用这些身份来查询和更新分类帐**。

将通过三个主要步骤，完成示例演示：

**1、建立开发环境。** 应用程序**需要一个网络**进行交互，因此需要下载一个**仅限于注册/认证，查询和更新所需的组件**：

![_images / AppConceptsOverview.png](https://hyperledger-fabric.readthedocs.io/en/latest/_images/AppConceptsOverview.png)

**2、学习应用程序将使用的示例中智能合约的参数。**智能合约包含各种功能，能够以不同的方式**与分类帐**进行交互。将**进入并检查该智能合约**，以了解应用程序将使用的功能。

**3、开发应用程序以便能够在分类帐上查询和更新资产。** 将进入应用程序代码本身（应用程序已用`Javascript`编写）并**手动操作变量**以运行不同类型的**查询和更新**。

完成本教程后，应该基本了解**应用程序如何与智能合约一起编程**，以便与`Fabric`网络上的分类帐（即对等方）进行交互。

# 建立开发环境

如果已经完成了[构建第一个网络](hyperledger%20fabric%20建立你的第一个网络.md)，那么应该设置开发环境并下载`fabric-samples`以及附带的配置文件。要运行本教程，现在需要做的就是**拆除（停止）现有的任何网络**，可以通过发出以下命令来执行此操作：

```sh
$ ./byfn.sh down
```

如果没有开发环境以及网络和应用程序的附带配置工件，请访问[环境条件](hyperledger%20fabric%20入门.md)页面，并确保在计算机上安装了必要的依赖项。

接下来，如果尚未执行此操作，请访问[安装示例，二进制文件和Docker镜像](hyperledger%20fabric%20入门.md)页面，然后按照提供的说明进行操作。克隆`fabric-samples`存储库后，返回本教程 ，并**下载最新的稳定`Fabric`映像和可用应用程序**。

所有准备完成后，**导航到`fabric-samples`存储库中的`fabcar`子目录**，并查看内部的内容：

```sh
$ cd fabric-samples/fabcar  && ls

enrollAdmin.js  invoke.js  package.json  query.js  registerUser.js  startFabric.sh
```

在开始之前，我们还需要做一些**清理工作**。运行以下命令以**终止任何陈旧或活动容器**：

```sh
$ docker rm -f $(docker ps -aq)
```

**清除所有缓存的网络**：

```sh
# Press 'y' when prompted by the command
$ docker network prune
```

最后，如果已经完成了本教程，那么还需要**删除`fabcar`智能合约的基础链码图像**。如果您是第一次浏览此内容的用户，那么您的系统上将不会显示此链代码图像：

```sh
$ docker rmi dev-peer0.org1.example.com-fabcar-1.0-5c906e402ed29f20260ae42283216aa75549c571e2e380f3615826365d8269ba

# or
$ docker rmi $(docker images dev-* -aq)
```

## 安装客户端并启动网络

> **注意**：以下说明要求操作位于`fabric-samples` `repo`的本地克隆中的`fabcar`子目录中。在本教程的其余部分中，保留在此子目录的根目录下。

运行以下命令以安装应用程序的`Fabric`依赖项。需要关注的是`fabric-ca-client`，它允许**应用程序与`CA`服务器通信并检索身份资料**，以及`fabric-client`，它允许**加载身份资料并与同行和订购服务交互**。

如果`node`或`npm`版本过旧，可以先安装或升级相关程序

```sh
$ sudo npm -v
# 安装node版本安装工具
$ sudo npm install -g n
# 安装指定版本node
$ n v8.9.4

# 配置NODE_HOME，进入profile编辑环境变量
$ vim /etc/profile
#set for nodejs
export NODE_HOME=/usr/local/n/versions/node/8.9.4
export PATH=$NODE_HOME/bin:$PATH
# set npm module path
export NODE_PATH=/usr/local/lib/node_modules

# 立即执行生效
$ source /etc/profile

$ node -v
v8.9.4 

$ npm -v
```

安装依赖的 `nodejs` 程序库

```sh
$ npm install
```

使用`startFabric.sh` 脚本启动网络。此命令将启动各种`Fabric`实体并为`Golang`编写的链码启动智能合约容器：

```sh
$ ./startFabric.sh
```

还可以选择针对`Node.js`中编写的链码运行本教程。如果想继续这条路线，请发出以下命令：

```sh
$ ./startFabric.sh node
```

> **注意**：`Node.js`链码场景大约**需要90秒**才能完成，也许更长。脚本没有挂起，而是增加的时间是在**构建链代码镜像时安装`fabric-shim`的结果**。

好了，现在已经有了一个示例网络和一些代码，来看看不同的部分是如何组合在一起的。

# 应用程序如何与网络交互

要更深入地了解`fabcar`网络中的组件（它们的**部署方式**）以及**应用程序如何在更精细的级别上与这些组件进行交互**，请参阅[了解Fabcar网络](https://hyperledger-fabric.readthedocs.io/en/latest/understand_fabcar_network.html)。

开发人员更有兴趣了解应用程序的作用，以及查看代码本身以查看**应用程序的构建方式**应该继续。目前，最重要的是要知道应用程序**使用软件开发工具包（`SDK`）来访问允许查询和更新分类帐的`API`**。

# 注册管理员用户

> **注意**：以下两节涉及与**证书颁发机构的通信**。可能会发现在运行即将推出的程序时流式传输`CA`日志很有用。

要流式查看`CA`日志，请拆分终端或打开新`shell`并发出以下命令：

```sh
$ docker logs -f ca.example.com
```

当启动网络时，**管理员用户 `admin` 已在证书颁发机构注册**。现在，需要向`CA`服务器发送注册请求，并为该用户检索注册证书（`eCert`）。不会在这里深入研究注册详细信息，但只需说明`SDK`和扩展的应用程序需要此证书才能形成管理员的用户对象。然后，将**使用此管理对象随后注册并注册新用户**。将管理员注册请求发送到`CA`服务器：

```sh
$ node enrollAdmin.js
```

程序将调用证书签名请求（`CSR`），并最终将`eCert`和密钥材料输出到此项目根目录中新创建的文件夹 `hfc-key-store`。然后，当应用需要为各种用户创建或加载身份对象时，他们会查看此位置。

# 注册并认证 `user1`

使用新生成的管理`eCert`，现在将再次与`CA`服务器通信以注册和认证新用户。此**用户 `user1`将是在查询和更新分类帐时使用的身份标识**。这里需要注意的是，**管理员身份是为我们的新用户发出注册和注册请求**（即该用户扮演注册商的角色）。发送注册并认证`user1`的电话：

```sh
$ node registerUser.js
```

与管理员注册类似，该程序调用`CSR`并将密钥和`eCert`输出到`hfc-key-store`子目录中。所以现在有两个独立用户的身份资料 `admin`＆`user1`。

# 查询分类帐

**查询是从分类帐中读取数据的方式**。此数据存储为一系列键值对，可以查询单个键，多个键的值。如果分类帐是以`JSON`等丰富的数据存储格式编写的，对其执行复杂的搜索（寻找包含某些关键字的所有资产）。

查询如何工作的表示：

![_images/QueryingtheLedger.png](https://hyperledger-fabric.readthedocs.io/en/latest/_images/QueryingtheLedger.png)

首先，运行`query.js`程序，返回分类帐中所有汽车的清单。**使用第二个身份`user1`作为此应用程序的签名实体**。程序中的以下行将`user1`指定为签名者：

```javascript
fabric_client.getUserContext('user1', true);
```

**`user1`注册资料已经放入`hfc-key-store`子目录**中，所以只需要告诉应用程序获取该身份。通过定义用户对象，现在可以继续从分类帐中读取。将查询所有汽车`queryAllCars`的函数预先加载到应用程序中，因此可以按原样运行程序：

```sh
$ node query.js
```

它应该返回这样的东西：

```sh
Successfully loaded user1 from persistence
Query has completed, checking results
Response is  [{"Key":"CAR0", "Record":{"colour":"blue","make":"Toyota","model":"Prius","owner":"Tomoko"}},
{"Key":"CAR1",   "Record":{"colour":"red","make":"Ford","model":"Mustang","owner":"Brad"}},
{"Key":"CAR2", "Record":{"colour":"green","make":"Hyundai","model":"Tucson","owner":"Jin Soo"}},
{"Key":"CAR3", "Record":{"colour":"yellow","make":"Volkswagen","model":"Passat","owner":"Max"}},
{"Key":"CAR4", "Record":{"colour":"black","make":"Tesla","model":"S","owner":"Adriana"}},
{"Key":"CAR5", "Record":{"colour":"purple","make":"Peugeot","model":"205","owner":"Michel"}},
{"Key":"CAR6", "Record":{"colour":"white","make":"Chery","model":"S22L","owner":"Aarav"}},
{"Key":"CAR7", "Record":{"colour":"violet","make":"Fiat","model":"Punto","owner":"Pari"}},
{"Key":"CAR8", "Record":{"colour":"indigo","make":"Tata","model":"Nano","owner":"Valeria"}},
{"Key":"CAR9", "Record":{"colour":"brown","make":"Holden","model":"Barina","owner":"Shotaro"}}]
```

这些是`10`辆车。由`Adriana`拥有的黑色特斯拉`Model S`，由`Brad`拥有的红色`Ford Mustang`，`Pari`拥有的紫色菲亚特`Punto`等等。分类账是基于键值的，在我们的实现中，**关键是`CAR0`到`CAR9`**。这将在一瞬间变得特别重要。

仔细看看这个程序，使用编辑器并打开`query.js`。应用程序的**初始部分定义了某些变量**，例如通道名称，证书存储位置和网络端点。在示例应用程序中，这些变量已经被引用，但在实际应用程序中，这些变量必须由`app dev`指定。

```js
var channel = fabric_client.newChannel('mychannel');
var peer = fabric_client.newPeer('grpc://localhost:7051');
channel.addPeer(peer);

var member_user = null;
var store_path = path.join(__dirname, 'hfc-key-store');
console.log('Store path:'+store_path);
var tx_id = null;
```

这是构造查询的代码块：

```js
// queryCar chaincode function - requires 1 argument, ex: args: ['CAR4'],
// queryAllCars chaincode function - requires no arguments , ex: args: [''],
const request = {
  //targets : --- letting this default to the peers assigned to the channel
  chaincodeId: 'fabcar',
  fcn: 'queryAllCars',
  args: ['']
};
```

当应用程序运行时，它调用对等体上的`fabcar`链代码，在其中运行`queryAllCars`函数，并且不传递任何参数。

要查看智能合约中的可用功能，请导航到`fabric-samples`根目录下的`chaincode/fabcar/go`子目录，并在编辑器中打开`fabcar.go`。

> **注意**：这些相同的功能在`fabcar`链码的`Node.js`版本中定义。

在代码中将看到可以调用以下函数：`initLedger`，`queryCar`，`queryAllCars`，`createCar`和`changeCarOwner`。让我们**仔细看看`queryAllCars`函数，看看它如何与分类帐交互**。

```go
func (s *SmartContract) queryAllCars(APIstub shim.ChaincodeStubInterface) sc.Response {

      startKey := "CAR0"
      endKey := "CAR999"

      resultsIterator, err := APIstub.GetStateByRange(startKey, endKey)
```

这定义了`queryAllCars`的范围。`CAR0`和`CAR999`之间的每辆车 总共`1,000`辆车，假设每个`key`都已正确标记将由查询返回。

下面是**应用程序如何在链代码中调用不同函数**的演示。必须根据链码`shim`中的可用`API`对每个函数进行编码，这反过来允许智能合约容器与对等分类帐正确连接。

![_images/RunningtheSample.png](https://hyperledger-fabric.readthedocs.io/en/latest/_images/RunningtheSample.png)

可以看到`queryAllCars`函数，以及一个名为`createCar`的函数，它将允许更新分类帐并最终在一瞬间将新块附加到链中。

但首先，返回`query.js`程序并编辑构造函数请求以查询`CAR4`。我们通过将`query.js`中的函数从`queryAllCars`更改为`queryCar`并将`CAR4`作为特定键传递来完成此操作。

`query.js`程序现在应该如下所示：

```sh
const request = {
  //targets : --- letting this default to the peers assigned to the channel
  chaincodeId: 'fabcar',
  fcn: 'queryCar',
  args: ['CAR4']
};
```

保存程序并返回`fabcar`目录。现在再次运行程序：

```sh
$ node query.js

{"colour":"black","make":"Tesla","model":"S","owner":"Adriana"}
```

如果你回过头来查看我们之前查询过每辆车的结果，你可以看到`CAR4`是`Adriana`的黑色特斯拉型号`S`，这是在这里返回的结果。

使用`queryCar`函数，我们可以查询任何键（例如`CAR0`）并获得与该汽车相对应的任何品牌，型号，颜色和所有者。

# 更新分类帐

现在已经完成了一些分类帐查询并添加了一些代码，已准备好更新分类帐。可以做很多潜在的更新，但让我们从创建汽车开始吧。

下面可以看到这个过程如何运作。建议更新，签署，然后返回给应用程序，然后应用程序将其发送给每个对等方的分类帐进行排序和写入：

![_images/UpdatingtheLedger.png](https://hyperledger-fabric.readthedocs.io/en/latest/_images/UpdatingtheLedger.png)

我们对分类账的第一次更新将是创建一辆新车。有一个单独的`Javascript`程序  `invoke.js`  我们将用它来进行更新。与查询一样，使用编辑器打开程序并导航到构建调用的代码块：

```js
// createCar chaincode function - requires 5 args, ex: args: ['CAR12', 'Honda', 'Accord', 'Black', 'Tom'],
// changeCarOwner chaincode function - requires 2 args , ex: args: ['CAR10', 'Barry'],
// must send the proposal to endorsing peers
var request = {
  //targets: let default to the peer assigned to the client
  chaincodeId: 'fabcar',
  fcn: '',
  args: [''],
  txId: tx_id
};
```

将看到可以调用两个函数 `createCar`或`changeCarOwner`。首先，创建一个红色雪佛兰`Volt`并将其交给名为`Nick`的老板。我们的分类账上有`CAR9`，因此我们将`CAR10`作为识别密钥。编辑此代码块如下所示：

```go
var request = {
  //targets: let default to the peer assigned to the client
  chaincodeId: 'fabcar',
  fcn: 'createCar',
  args: ['CAR10', 'Chevy', 'Volt', 'Red', 'Nick'],
  txId: tx_id
};
```

保存并运行程序：

```sh
$ node invoke.js
```

终端中会有一些关于`ProposalResponse`和`promises`的输出。但是，我们所关注的是这条消息：

```sh
The transaction has been committed on peer localhost:7053
```

要查看此交易已写入，请返回`query.js`并将参数从`CAR4`更改为`CAR10`。

再次改变下代码：

```js
const request = {
  //targets : --- letting this default to the peers assigned to the channel
  chaincodeId: 'fabcar',
  fcn: 'queryCar',
  args: ['CAR4']
};

const request = {
  //targets : --- letting this default to the peers assigned to the channel
  chaincodeId: 'fabcar',
  fcn: 'queryCar',
  args: ['CAR10']
};
```

再次保存，然后查询：

```sh
$ node query.js
```

将返回如下结果：

```sh
Response is  {"colour":"Red","make":"Chevy","model":"Volt","owner":"Nick"}
```

要做到这一点，请返回`invoke.js`并将函数从`createCar`更改为`changeCarOwner`并输入如下参数：

```js
var request = {
  //targets: let default to the peer assigned to the client
  chaincodeId: 'fabcar',
  fcn: 'changeCarOwner',
  args: ['CAR10', 'Dave'],
  txId: tx_id
};
```

第一个参数 `CAR10` 反映了将改变的汽车。第二个参数 `Dave` 定义了汽车的新主人。

再次保存并执行程序：

```sh
$ node invoke.js
```

现在让我们再次查询分类帐并确保`Dave`现在与`CAR10`密钥相关联：

```sh
$ node query.js
```

它应该返回这个结果：

```sh
Response is  {"colour":"Red","make":"Chevy","model":"Volt","owner":"Dave"}
```

`CAR10`的所有权已从`Nick`改为`Dave`。

