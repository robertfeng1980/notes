# `hyperledger fabric` 访问控制列表`ACL`

> **注意**：本文介绍**通道管理级别的访问控制和策略**。要了解链代码中的访问控制，请查看[开发链代码教程](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#Chaincode_API)。

`Fabric`使用访问控制列表（`ACL`）来**管理对资源的访问**，方法是将**策略（在给定一组身份的情况下指定评估为`true`或`false`的规则）与资源相关联**。`Fabric`包含许多默认`ACL`。在本文档中，将讨论如何格式化它们以及如何覆盖默认值。

# 资源

用户通过**定位[用户链代码](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html)，[系统链代码](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4noah.html)或[事件流源](https://hyperledger-fabric.readthedocs.io/en/latest/peer_event_services.html)**与`Fabric`进行交互。因此，这些端点被视为应在其上执行访问控制的**资源**。

应用程序开发人员需要了解这些资源以及与之**关联的默认策略**。这些资源的**完整列表可在`configtx.yaml`中找到**。可以[在此处查看示例`configtx.yaml`文件](http://github.com/hyperledger/fabric/blob/release-1.2/sampleconfig/configtx.yaml)。

`configtx.yaml`中指定的资源**是`Fabric`当前定义的所有内部资源的详尽列表**。那里采用的松散约定是`<component>/<resource>`。因此`cscc/GetConfigBlock`是`CSCC`组件中`GetConfigBlock`调用的资源。





