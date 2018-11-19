# `hyperledger fabric` 可插拔交易认可和验证

在提交时验证事务时，对等体在应用事务本身随附的**状态更改之前执行各种检查**：

+ **验证**签署事务的**身份**。
+ 在交易中**验证参与者的签名**。
+ 确保事务满足相应链代码的命名空间的**认可策略**。

有些用例要求自定义事务验证规则与默认的`Fabric`验证**规则不同**，例如：

+ **`UTXO`（未使用的交易输出）**：当验证考虑到交易**是否不会花费**其投入。
+ **`Anonymous transactions` 匿名交易**：当认可**不包含对等体的身份**时，但共享的签名和公钥不能链接到对等体的身份。

# 可插拔的认可和验证逻辑

`Fabric`允许将**定制认可和验证逻辑实现和部署到对等体中**，以可插拔的方式与链码处理相关联。这个逻辑可以**编译到对等体中**，**内置于可选逻辑中**，也可以**作为[`Golang`插件](https://golang.org/pkg/plugin/)与对等体一起编译和部署**。

在链代码**实例化**时，每个链代码都与其**自己的认可和验证逻辑相关联**。如果用户未选择一个，则**隐式选择默认的内置逻辑**。对等管理员可以通过在对等启动时**加载和应用的认可/验证逻辑**的定制来**改变**通过扩展对等体的**本地配置**而选择的认可/验证逻辑。

# 配置

每个对等体都有一个本地配置（`core.yaml`），它**声明了认可/验证逻辑名称和要运行的实现之间的映射**。

**默认逻辑**称为`ESCC`（`E`代表认可）和`VSCC`（`V`代表验证），它们可以在处理程序部分的**对等本地配置**中找到：

```yaml
handlers:
    endorsers:
      escc:
        name: DefaultEndorsement
    validators:
      vscc:
        name: DefaultValidation
```

当**认可或验证**实现被编译到**对等体**中时，`name`属性表示要**运行的初始化函数**，以便获得**创建认可/验证逻辑实例的工厂**。

该函数是`core/handlers/library/library.go`下的`HandlerLibrary`构造的实例方法，为了添加自定义认可或验证逻辑，需要使用任何其他方法**扩展此构造**。

由于这很麻烦并且构成了部署挑战，因此可以通过在名为`library`的名称下**添加另一个属性**来将**自定义认可和验证部署为`Golang`插件**。

例如，如果有作为**插件实现的自定义认可和验证逻辑**，将在`core.yaml`的配置中包含以下条目：

```yaml
handlers:
    endorsers:
      escc:
        name: DefaultEndorsement
      custom:
        name: customEndorsement
        library: /etc/hyperledger/fabric/plugins/customEndorsement.so
    validators:
      vscc:
        name: DefaultValidation
      custom:
        name: customValidation
        library: /etc/hyperledger/fabric/plugins/customValidation.so
```

**必须将`.so`插件文件放在对等的本地文件系统中**。

> **注意**：此后，自定义认可或验证逻辑实现将被称为“插件”，即使它们被编译到对等体中。

