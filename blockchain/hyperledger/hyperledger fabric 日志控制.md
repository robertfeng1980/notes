# `hyperledger fabric` 日志控制

日志对等体和订购者**由`common/flogging`包提供**。如果**使用`shim`程序提供的日志记录方法**，使用`Go`编写的`Chaincodes`也会使用此包。这个包支持：

+ 根据消息的**严重性**记录日志
+ 基于**生成消息的软件记录器**的记录日志
+ 基于消息严重性的不同**打印漂亮日志**选项

**所有日志目前都定向到`stderr`**。为用户和开发人员提供了按严重性进行的**全局和记录器级别**的日志记录控制。目前没有针对每个**严重性级别**提供的信息类型的正式规则。提交错误报告时，开发人员可能希望查看完整日志，直至`DEBUG`级别。

在漂亮打印的日志中，**日志级别由颜色和四字符代码指示**，例如，`ERROR为“ERRO”`，`DEBUG为“DEBU”`等。在日志记录上下文中，记录器是任意名称（字符串）由开发人员提供给相关消息组。在下面的漂亮打印示例中，记录器`ledgermgmt`，`kvledger`和`peer`正在生成日志。

```sh
2018-11-01 15:32:38.268 UTC [ledgermgmt] initialize -> INFO 002 Initializing ledger mgmt 
2018-11-01 15:32:38.268 UTC [kvledger] NewProvider -> INFO 003 Initializing ledger provider 
2018-11-01 15:32:38.342 UTC [kvledger] NewProvider -> INFO 004 ledger provider Initialized 
2018-11-01 15:32:38.357 UTC [ledgermgmt] initialize -> INFO 005 ledger mgmt initialized 2018-11-01 15:32:38.357 UTC [peer] func1 -> INFO 006 Auto-detected peer address: 172.24.0.3:7051 
2018-11-01 15:32:38.357 UTC [peer] func1 -> INFO 007 Returning peer0.org1.example.com:7051
```

可以在运行时**创建任意数量的记录器**，因此没有记录器的“主列表”，并且记录控制构造**无法检查**记录记录器**是否实际存在或将存在**。

# 日志规范

`peer`和`orderer`命令的日志记录级别由**日志记录规范控制**，该规范通过`FABRIC_LOGGING_SPEC`环境变量设置。

完整的日志记录级别规范是表单：

```sh
[<logger>[,<logger>...]=]<level>[:[<logger>[,<logger>...]=]<level>...]
```

使用从中选择的**不区分大小写**的字符串指定日志记录严重性级别：

