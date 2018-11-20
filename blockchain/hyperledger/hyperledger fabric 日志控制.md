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

使用从中选择的**不区分大小写**的字符串指定日志记录**严重性级别**记录级别本身被视为整体默认值。

```sh
FATAL | PANIC | ERROR | WARNING | INFO | DEBUG
```

否则，可以使用以下命令指定**个人或记录器组**的覆盖句法。

```sh
<logger>[,<logger>...]=<level>
```

日志规格示例：

```sh
info                                        - Set default to INFO
warning:msp,gossip=warning:chaincode=info   - Default WARNING; Override for msp, gossip, and chaincode
chaincode=info:msp,gossip=warning:warning   - Same as above
```

# 日志格式

`peer`和`orderer`命令的日志记录格式是**通过`FABRIC_LOGGING_FORMAT`环境变量控制**的。这可以设置为格式字符串，例如默认值：

```go
"%{color}%{time:2006-01-02 15:04:05.000 MST} [%{module}] %{shortfunc} -> %{level:.4s} %{id:03x}%{color:reset} %{message}"
```

以人类可读的控制台格式打印日志。它也可以设置为`json`以`JSON`格式输出日志。

# `Go` 合约

`Go`链代码应用程序中日志的标准机制是**通过对等方与每个链代码实例公开的日志传输集成**。`chaincode shim`包提供了`API`，**允许链代码创建和管理日志对象**，其日志将与`shim`日志一致地格式化和交错。

作为独立执行的程序，**用户提供的链代码在技术上也可以在`stdout/stderr`上产生输出**。虽然对于`devmode`自然有用，但这些**通道通常在生产网络上被禁用**，以减少来自**破坏或恶意代码**的滥用。但是，通过`CORE_VM_DOCKER_ATTACHSTDOUT=true`配置选项，甚至可以在**每个对等体**的基础上为对等管理的容器（例如`netmode`）**启用此输出**。

一旦启用，每个链代码将接收由其`container-id`键入的自己的**日志记录通道**。写入`stdout`或`stderr`的任何输出都将以**每行为基础与对等的日志集成**。**建议不要将其用于生产**。

# API

+ `NewLogger(name string) *ChaincodeLogger` 
  创建一个供链代码使用的**日志记录对象**

+ `(c *ChaincodeLogger) SetLevel(level LoggingLevel)`  

  设置记录器的**日志记录级别**

+ `(c *ChaincodeLogger) IsEnabledFor(level LoggingLevel) bool` 

  如果将**在给定级别生成日志**，则返回`true`

+ `LogLevel(levelString string) (LoggingLevel, error)` 

  将字符串转换为`LoggingLevel`

+ `LoggingLevel` 是枚举的成员

    ```sh
    LogDebug, LogInfo, LogNotice, LogWarning, LogError, LogCritical
    ```

    可以**直接使用**，也可以通过传递**不区分大小写的字符串来生成**到`LogLevel API`。

+ `SetLoggingLevel（LoggingLevel level）`

    **控制`shim`程序**的日志记录级别

## 提供函数

各种严重性级别的格式化日志记录由函数提供：

```go
(c *ChaincodeLogger) Debug(args ...interface{})
(c *ChaincodeLogger) Info(args ...interface{})
(c *ChaincodeLogger) Notice(args ...interface{})
(c *ChaincodeLogger) Warning(args ...interface{})
(c *ChaincodeLogger) Error(args ...interface{})
(c *ChaincodeLogger) Critical(args ...interface{})

(c *ChaincodeLogger) Debugf(format string, args ...interface{})
(c *ChaincodeLogger) Infof(format string, args ...interface{})
(c *ChaincodeLogger) Noticef(format string, args ...interface{})
(c *ChaincodeLogger) Warningf(format string, args ...interface{})
(c *ChaincodeLogger) Errorf(format string, args ...interface{})
(c *ChaincodeLogger) Criticalf(format string, args ...interface{})
```

日志`API`的`f`形式可以**精确控制日志的格式**。`API`的非`f`形式**当前在参数的打印表示之间插入空格**，并且任意选择要使用的格式。

在当前实现中，由`shim`和`ChaincodeLogger`生成的日志**带有时间戳**，标记有日志器**名称和严重性级别**，并写入`stderr`。请注意，日志记录级别控制当前**基于创建`ChaincodeLogger`时提供的名称**。为了避免歧义，所有`ChaincodeLogger`都应该被**赋予除`shim`之外的唯一名称**。记录器名称将出现在记录器创建的所有日志消息中，`shim`记录名称为`shim`。

## 日志级别设置

可以在`core.yaml`文件中设置`Chaincode`容器中记录器的**默认日志记录级别**。键`chaincode.logging.level`为`Chaincode`容器中的**所有记录器设置默认级别**。键`chaincode.logging.shim`会**覆盖`shim`记录器的默认级别**。

```yml
# Logging section for the chaincode container
logging:
  # Default level for all loggers within the chaincode container
  level:  info
  # Override default level for the 'shim' logger
  shim:   warning
```

可以使用**环境变量覆盖默认日志记录级别**。`CORE_CHAINCODE_LOGGING_LEVEL`设置**所有记录器的默认日志记录级别**。`CORE_CHAINCODE_LOGGING_SHIM`会**覆盖`shim`记录器的级别**。

`Go`语言链代码还可以**通过`SetLoggingLevel API`控制链代码`shim`程序接口的日志记录级别**。

`SetLoggingLevel(LoggingLevel level)`  控制`shim`的记录级别

下面是一个简单的示例，说明链代码如何创建`LogInfo`级别的**私有日志记录**对象。

```go
var logger = shim.NewLogger("myChaincode")

func main() {

    logger.SetLevel(shim.LogInfo)
    ...
}
```





