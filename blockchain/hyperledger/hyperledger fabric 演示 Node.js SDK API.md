# `hyperledger fabric` 演示 `Node.js SDK API`

`NodeJS` 应用程序利用 `fabric-client` 和 `fabric-ca-client` 同时结合 `NodeJS SDK API` 接口调用智能合约 `chaincode`。

# 准备工作

+ `docker` 的安装， `v1.12 +`
+ `docker-compose` 的安装，`v1.8 +`
+ `Git` 的安装
+ `NodeJS` 的安装，`v8.9.4 +`  `npm v5.6.0+`
+ `fabric docker images`
  ```sh
  $ curl https://github.com/hyperledger/fabric/blob/release-1.1/scripts/bootstrap.sh
  $ /bin/bash bootstrap.sh
  ```

## `NodeJS` 和 `NPM` 安装与设置

### 升级版本

```sh
$ npm -v

$ node -v
v4.2.6

# 安装node 升级模块
$ npm install -g n

# n stable   
# 或者更新到指定版本  
$ n v8.9.4    
     install : node-v8.9.4 
       mkdir : /usr/local/n/versions/node/8.9.4 
       fetch : https://nodejs.org/dist/v8.9.4/node-v8.9.4-linux-x64.tar.gz
######################################################################## 100.0%
   installed : v8.9.4 
```

### 配置环境变量

```sh
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

# nodejs还是找不到node命令
$ apt install nodejs-legacy -y
```

### 配置 `NPM` 模块和缓存目录

```sh
# npm模块安装路径
/usr/local/n/versions/node/8.9.4/lib/node_modules/

# 设置 node_modules 的缓存目录和安装目录
$ mkdir -p $NODE_PATH/node_global
$ mkdir -p $NODE_PATH/node_cache
$ npm config set prefix "$NODE_PATH/node_global"
$ npm config set cache "$NODE_PATH/node_cache"

# 查看配置是否生效
$ npm config ls 

# 注销并重新登陆机器
```

### `NPM` 简单命令

```sh
# 模块 安装当前目录下
$ npm install 

# 升级npm
$ npm i npm@latest -g

# 更新 模块安装
$ npm update

# 卸载 当前安装包
$ npm uninstall

# 模块 安装在全局模块下
$ npm install -g

# 更新所有全局安装包
$ npm update -g .

# 初始化项目，生成 package.json
$ npm init -y

# 安装依赖包
$ sudo npm install --unsafe-perm --verbose -g speaker
$ sudo npm install -g PackageName --allow-root

# 删除安装包和npm缓存
$ rm -rf node_modules
$ rm -rf ~/.npm

# current user
$ npm cache clean -f
# clean root
$ sudo npm cache clean -f
$ npm cache verify

# upgrade npm
$ npm install -g npm@latest
$ sudo chown $USER:$USER ~/.npm/ -R

# upgrade node
$ sudo ln -s nodejs /usr/bin/node
$ sudo apt-get install nodejs-legacy
```

# 下载代码

```sh
$ cd /opt/gopath/src/github.com/hyperledger/
$ git clone https://github.com/hyperledger/fabric-samples.git

$ cd fabric-samples/balance-transfer/
```

# 运行示例

本示例有两个`chaincode`可以选择安装使用，你可以选择 使用`golang`或`node.js`编写的`chaincode`运行。确定已经进入目录 `fabric-samples/balance-transfer/`

## 手动命令行运行示例

+ 启动 `docker` 服务容器编排

  ```sh
  $ docker-compose -f artifacts/docker-compose.yaml up
  ```

+ 安装`fabric-client`和`fabric-ca-client`节点模块

  ```sh
  # 模块 安装当前目录下
  $ npm install 
  ```

+ 在`PORT 4000`上启动节点应用程序

  ```sh
  $ PORT=4000 node app
  
  /opt/gopath/src/github.com/hyperledger/fabric-samples/balance-transfer/node_modules/grpc/src/grpc_extension.js:45
      throw error;
      ^
  
  Error: The gRPC binary module was not installed. This may be fixed by running "npm rebuild"
  Original error: Cannot find module '/opt/gopath/src/github.com/hyperledger/fabric-samples/balance-transfer/node_modules/grpc/src/node/extension_binary/node-v57-linux-x64-glibc/grpc_node.node'
  
  # 出现错误，按照提示
  $ npm rebuild
  
  # 再次启动应用
  $ PORT=4000 node app
  [2018-06-05 08:43:10.807] [INFO] SampleWebApp - ****************** SERVER STARTED ************************
  [2018-06-05 08:43:10.809] [INFO] SampleWebApp - ***************  http://localhost:4000  ******************
  ```

## 脚本自动化运行示例

+ 运行脚本启动所需的网络，完成服务编排

  ```sh
  $ cd fabric-samples/balance-transfer
  $ ./runApp.sh
  ```

  脚本完成以下任务：

  - 这会在本地计算机上启动所需的网络
  - 安装`fabric-client`和`fabric-ca-client`节点模块
  - 在`PORT 4000`上启动节点应用程序

+ 为了让以下`shell`脚本正确解析`JSON`，必须安装`jq`

  ```sh
  $ whereis jq
  
  # 安装jq
  $ wget -O jq https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64
  $ chmod +x jq
  
  $ mv jq /usr/local/bin/jq
  ```

+ 运行脚本 **`testAPIs.sh`** 进行交易测试

  ```sh
  $ cd fabric-samples/balance-transfer
  
  ## To use golang chaincode execute the following command
  $ ./testAPIs.sh -l golang
  
  ## OR use node.js chaincode
  $ ./testAPIs.sh -l node
  
  ## nodejs 在安装实例化chaincode的时候出现超时异常
  ## POST instantiate chaincode on peer1 of Org1
  error: [client-utils.js]: sendPeersProposal - Promise is rejected: Error: REQUEST_TIMEOUT
  
  # 修改 xxx/fabric-samples/balance-transfer/node_modules/fabric-client/config/default.json文件下的45000的超时时间为更长, 改为100000便可以了
  ```

# 清理网络

网络仍然在运行。在再次手动启动网络之前，下面是清理容器和配置文件的命令。

```sh
$ docker rm -f $(docker ps -aq)
$ docker rmi -f $(docker images | grep dev | awk '{print $3}')
$ rm -rf fabric-client-kv-org[1-2]
```

# `REST API` 测试

## 登陆请求

在组织`Org1`中注册和注册新用户 ：

```sh
$ curl -s -X POST http://localhost:4000/users -H "content-type: application/x-www-form-urlencoded" -d 'username=Jim&orgName=Org1'

{"success":true,"secret":"","message":"Jim enrolled Successfully","token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MjgyMjQ2NTQsInVzZXJuYW1lIjoiSmltIiwib3JnTmFtZSI6Ik
```

响应包含 `成功/失败` 状态，**注册`secret`**和**`JSON Web Token（JWT）`**，它是请求标头中的后续请求所需的字符串的参数。

## 创建频道请求

```sh
$ curl -s -X POST \
  http://localhost:4000/channels \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json" \
  -d '{
	"channelName":"mychannel",
	"channelConfigPath":"../artifacts/channel/mychannel.tx"
}'
```

请注意，标头**授权**必须包含`POST /users`通话中返回的`JWT token`

## 加入频道请求

```sh
$ curl -s -X POST \
  http://localhost:4000/channels/mychannel/peers \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json" \
  -d '{
	"peers": ["peer0.org1.example.com","peer1.org1.example.com"]
}'
```

## 安装链码

```sh
$ curl -s -X POST \
  http://localhost:4000/chaincodes \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json" \
  -d '{
	"peers": ["peer0.org1.example.com","peer1.org1.example.com"],
	"chaincodeName":"mycc",
	"chaincodePath":"github.com/example_cc/go",
	"chaincodeType": "golang",
	"chaincodeVersion":"v0"
}'
```

**注意**： 当使用`node.js`链码并且*`chaincodePath`*必须设置为`node.js`链码的位置时，*`chaincodeType`*必须设置为**node**。

```sh
$ curl -s -X POST \
  http://localhost:4000/chaincodes \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json" \
  -d '{
	"peers": ["peer0.org1.example.com","peer1.org1.example.com"],
	"chaincodeName":"mycc",
	"chaincodePath":"$PWD/artifacts/src/github.com/example_cc/node",
	"chaincodeType": "node",
	"chaincodeVersion":"v0"
}'
```

## 实例化链码

```sh
$ curl -s -X POST \
  http://localhost:4000/channels/mychannel/chaincodes \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json" \
  -d '{
	"peers": ["peer0.org1.example.com","peer1.org1.example.com"],
	"chaincodeName":"mycc",
	"chaincodeVersion":"v0",
	"chaincodeType": "golang",
	"args":["a","100","b","200"]
}'
```

**注意**： 当使用`node.js`链码时，*`chaincodeType`*必须设置为**node**

##调用请求

```sh
$ curl -s -X POST \
  http://localhost:4000/channels/mychannel/chaincodes/mycc \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json" \
  -d '{
	"peers": ["peer0.org1.example.com","peer1.org1.example.com"],
	"fcn":"move",
	"args":["a","b","10"]
}'
```

**注意**：确保保存响应中的交易`ID`，以便在随后的查询交易中传递此字符串。

## `Chaincode`查询

```sh
$ curl -s -X GET \
  "http://localhost:4000/channels/mychannel/chaincodes/mycc?peer=peer0.org1.example.com&fcn=query&args=%5B%22a%22%5D" \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json"
```

## 通过 `BlockNumber` 查询区块

```sh
$ curl -s -X GET \
  "http://localhost:4000/channels/mychannel/blocks/1?peer=peer0.org1.example.com" \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json"
```

## 通过`TransactionID`查询交易

```sh
$ curl -s -X GET http://localhost:4000/channels/mychannel/transactions/<put transaction id here>?peer=peer0.org1.example.com \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json"
```

**注意**：交易`ID`可以来自任何先前的调用交易，查看调用请求的结果将看起来像`8a95b1794cb17e7772164c3f1292f8410fcfdc1943955a35c9764a21fcd1d1b3`。

## 查询 `ChainInfo`

```sh
$ curl -s -X GET \
  "http://localhost:4000/channels/mychannel?peer=peer0.org1.example.com" \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json"
```

## 查询已安装的连码

```sh
$ curl -s -X GET \
  "http://localhost:4000/chaincodes?peer=peer0.org1.example.com&type=installed" \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json"
```

## 查询实例化的链码

```sh
$ curl -s -X GET \
  "http://localhost:4000/chaincodes?peer=peer0.org1.example.com&type=instantiated" \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json"
```

## 查询频道

```sh
$ curl -s -X GET \
  "http://localhost:4000/channels?peer=peer0.org1.example.com" \
  -H "authorization: Bearer <put JSON Web Token here>" \
  -H "content-type: application/json"
```



