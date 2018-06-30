### fabric 学习中，历史遗留问题清单

+ 在[configtxupdate](https://github.com/hyperledger/fabric/tree/release-1.1/examples/configtxupdate) 示例中最后一步操作 `update org` 更新组织时，出现版本不一致无法更新BUG

  ```sh
  Error: got unexpected status: BAD_REQUEST -- error authorizing update: error validating DeltaSet: policy for [Group]  /Channel/Application not satisfied: Failed to reach implicit threshold of 2 sub-policies, required 1 remaining
  ```

  解决方案：

  

+ 在 `balance-transfer ` 示例中，无法安装 `nodejs`的 `chaincode` 问题

  ```sh
  error: [client-utils.js]: sendPeersProposal - Promise is rejected: Error: REQUEST_TIMEOUT
  ```

  解决方案：

  

+ 待增加

