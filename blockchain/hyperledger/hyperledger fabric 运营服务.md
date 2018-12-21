# `hyperledger fabric` 运营服务

对等方和订货方托管提供`RESTful` 操作 `API`的`HTTP`服务器。**此`API`与`Fabric`网络服务无关**，旨在**供运营商使用**，而不是网络的管理员或用户。

`API`公开了以下功能：

+ 日志级别管理
+ 健康检查
+ `Prometheus`针对运营指标的目标（配置时）

