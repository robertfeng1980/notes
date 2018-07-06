### fabric 学习中，历史遗留问题清单

+ 在[configtxupdate](https://github.com/hyperledger/fabric/tree/release-1.1/examples/configtxupdate) 示例中最后一步操作 `update org` 更新组织时，出现版本不一致无法更新BUG

  ```sh
  Error: got unexpected status: BAD_REQUEST -- error authorizing update: error validating DeltaSet: policy for [Group]  /Channel/Application not satisfied: Failed to reach implicit threshold of 2 sub-policies, required 1 remaining
  ```

  **解决方案**：

  

+ 在 `balance-transfer ` 示例中，`/fabric-samples/balance-transfer# ./testAPIs.sh -l node`无法安装 `nodejs`的 `chaincode` 问题

  ```sh
  error: [client-utils.js]: sendPeersProposal - Promise is rejected: Error: REQUEST_TIMEOUT
  
  error: [client-utils.js]: sendPeersProposal - Promise is rejected: Error: 2 UNKNOWN: error starting container: Failed to generate platform-specific docker build: Error returned from build: 1 "
  > x509@0.3.3 install /chaincode/output/node_modules/x509
  > node-gyp rebuild
  
  gyp WARN install got an error, rolling back install
  gyp ERR! configure error
  gyp ERR! stack Error: getaddrinfo EAI_AGAIN nodejs.org:443
  gyp ERR! stack     at Object._errnoException (util.js:1022:11)
  gyp ERR! stack     at errnoException (dns.js:55:15)
  gyp ERR! stack     at GetAddrInfoReqWrap.onlookup [as oncomplete] (dns.js:92:26)
  gyp ERR! System Linux 4.4.0-128-generic
  gyp ERR! command "/usr/local/bin/node" "/usr/local/lib/node_modules/npm/node_modules/node-gyp/bin/node-gyp.js" "rebuild"
  gyp ERR! cwd /chaincode/output/node_modules/x509
  gyp ERR! node -v v8.9.4
  gyp ERR! node-gyp -v v3.6.2
  gyp ERR! not ok
  npm WARN example_cc@1.0.0 No repository field.
  
  [2018-06-29 08:38:08.778] [ERROR] instantiate-chaincode - instantiate proposal was bad
  [2018-06-29 08:38:08.778] [INFO] instantiate-chaincode - instantiate proposal was good
  [2018-06-29 08:38:08.778] [DEBUG] instantiate-chaincode - Failed to send Proposal and receive all good ProposalResponse
  [2018-06-29 08:38:08.778] [ERROR] instantiate-chaincode - Failed to instantiate. cause:Failed to send Proposal and receive all good ProposalResponse
  (node:22368) UnhandledPromiseRejectionWarning: Error: Failed to instantiate. cause:Failed to send Proposal and receive all good ProposalResponse
  ```

  **解决方案**：

  由于国内被墙网络问题，NPM无法下载 `fabric-shim`组件。上面的下载地址也就是 https://registry.npmjs.org/fabric-shim

  可以开启VPN，让程序自动下载 `fabric-shim`组件。

  或者设置NPM仓库镜像指向国内镜像

  ```sh
  $ npm config set registry https://registry.npm.taobao.org
  $ npm config get registry
  $ npm config list
  ```

+ 在`fabric-sdk-java` 的`End2endNodeIT`示例中利用 `JavaSDK API` 部署`NodeJS chaincode`后，进行初始化操作发生异常

  ```sh
  peer0.org1.example.com    | 2018-06-29 07:52:21.816 UTC [chaincode] Launch -> ERRO 43c launchAndWaitForRegister failed: Failed to generate platform-specific docker build: Error returned from build: 1 "npm ERR! code EAI_AGAIN
  peer0.org1.example.com    | npm ERR! errno EAI_AGAIN
  peer0.org1.example.com    | npm ERR! request to https://registry.npmjs.org/fabric-shim failed, reason: getaddrinfo EAI_AGAIN registry.npmjs.org:443
  peer0.org1.example.com    |
  peer0.org1.example.com    | npm ERR! A complete log of this run can be found in:
  peer0.org1.example.com    | npm ERR!     /root/.npm/_logs/2018-06-29T07_52_21_516Z-debug.log
  peer0.org1.example.com    | "
  peer0.org1.example.com    | error starting container
  ```

  **解决方案**：

  由于国内被墙网络问题，NPM无法下载 `fabric-shim`组件。上面的下载地址也就是 https://registry.npmjs.org/fabric-shim

  可以开启VPN，让程序自动下载 `fabric-shim`组件。

  或者设置NPM仓库镜像指向国内镜像

  ```sh
  $ npm config set registry https://registry.npm.taobao.org
  $ npm config get registry
  $ npm config list
  ```

  

+ 

