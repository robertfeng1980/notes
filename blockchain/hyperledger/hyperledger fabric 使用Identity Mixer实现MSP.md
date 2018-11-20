# `fabric`使用`Identity Mixer`实现`MSP`

`Idemix`是一个**加密协议套件**，它提供强大的**身份验证以及隐私保护**功能，如**匿名**，**无需披露交易者身份**即可进行交易，以及**不可链接性**，即**单个身份发送多个交易的能力**，而不会泄露交易是通过相同的身份发送的。

`Idemix`流中涉及三个参与者：**用户、发行者和验证者**。

![_images/idemix-overview.png](https://hyperledger-fabric.readthedocs.io/en/latest/_images/idemix-overview.png)

+ 发行者证明一组用户的属性以**数字证书**的形式发布，以下称为“**凭证**”。
+ 用户稍后生成拥有凭证的“[**零知识证明**](https://en.wikipedia.org/wiki/Zero-knowledge_proof)”，并且还**选择性地仅公开用户选择显示的属性**。证明，因为它是零知识，**不会向验证者，发行者或任何其他人显示其他信息**。

例如，假设`Alice`需要向`Bob`（店员）证明她拥有由`DMV`发给她的驾驶执照。

在这种情况下，`Alice`是用户，`DMV`是发行者，`Bob`是验证者。为了向`Bob`证明`Alice`有驾驶执照，她可以向他展示。然而，`Bob`可以看到`Alice`的姓名，地址，确切的年龄等等，这就比`Bob`需要了解的更多信息。

相反，`Alice`可以使用`Idemix`为`Bob`生成“**零知识证明**”，**只显示她拥有有效的驾驶执照而没有其他任何内容**。

所以从证据来看：

+ 除了拥有**有效许可证（匿名）之外**，`Bob`不会了解有关`Alice`的任何其他信息。
+ 如果`Alice`多次访问商店并且**每次为`Bob`生成证据**，`Bob`将**无法从证据中判断出它是同一个人**（**不可链接性**）。

`Idemix`身份验证技术提供的**信任模型和安全保证**类似于标准`X.509`证书所确保的信任模型和安全保证，但具有有效**提供高级隐私**功能的基础加密算法，包括上述隐私功能。在下面的技术部分详细比较`Idemix`和`X.509`技术。

# 如何使用`Idemix`

要了解如何将`Idemix`与`Hyperledger Fabric`一起使用，需要查看哪些`Fabric`组件与`Idemix`中的用户，颁发者和验证者相对应。

+ `Fabric Java SDK`是用户的`API`。将来，其他`Fabric SDK`也将支持`Idemix`。
+ `Fabric`提供了两个可能的`Idemix`发行者：
  + `Fabric CA`适用于生产或开发环境
  + 用于开发环境的`idemixgen`工具
+ 验证者是`Fabric`中的`Idemix MSP`

要在`Hyperledger Fabric`中使用`Idemix`，需要以下三个基本步骤：

![_images/idemix-three-steps.png](https://hyperledger-fabric.readthedocs.io/en/latest/_images/idemix-three-steps.png)

比较此图像中的角色与上面的角色。

1. 研究**发行人**。

   `Fabric CA`（版本`1.3`或更高版本）已得到增强，可**自动充当`Idemix`颁发者**。当`fabric-ca-server`启动（或通过`fabric-ca-server init`命令初始化）时，将在`fabric-ca-server`的主目录中**自动创建以下两个文件**：`IssuerPublicKey`和`IssuerRevocationPublicKey`。这些文件**在步骤2**中是必需的。

   对于开发环境，如果**不使用`Fabric CA`，可以使用`idemixgen`来创建这些文件**。

2. 研究**验证者**。

   需要**使用步骤1**中的`IssuerPublicKey`和`IssuerRevocationPublicKey`创建`Idemix MSP`。

   例如，请研究以下摘录自`Hyperledger Java SDK`示例中的`configtx.yaml`：

   ```yml
   - &Org1Idemix
       # defaultorg定义sampleconfig中使用的组织
       # fabric.git开发环境
       name: idemixMSP1
   
       # id将msp定义加载为
       id: idemixMSPID1
   
       msptype: idemix
       mspdir: crypto-config/peerOrganizations/org3.example.com
   ```

   `msptype`设置为`idemix`，`mspdir`目录的内容（本例中为`crypto-config/peerOrganizations/org3.example.com/msp`）包含`IssuerPublicKey`和`IssuerRevocationPublicKey`文件。

   > 请注意，在此示例中，`Org1Idemix`表示`Org1`的`Idemix MSP`（未显示），它也具有`X509 MSP`。

3. 研究**用户**。`Java SDK`是用户的`API`。

   为了将`Idemix`与`Java SDK`一起使用，只需要一个额外的`API`调用：`org.hyperledger.fabric_ca.sdk.HFCAClient`类的`idemixEnroll`方法。例如，假设`hfcaClient`是您的`HFCAClient`对象，`x509Enrollment`是与您的`X509`证书关联的`org.hyperledger.fabric.sdk.Enrollment`。

   以下调用将返回与您的`Idemix`凭据关联的`org.hyperledger.fabric.sdk.Enrollment`对象。

   ```go
   IdemixEnrollment idemixEnrollment = hfcaClient.idemixEnroll(x509enrollment, "idemixMSPID1");
   ```

# `Idemix`和链码

从**验证者的角度**来看，还有一个角色：**链码**。使用`Idemix`凭证时，链码可以了解有关交易者的信息？

当使用`Idemix`凭证时，[`cid`（客户端标识）库](https://github.com/hyperledger/fabric/tree/master/core/chaincode/shim/ext/cid)（仅用于`golang`）已扩展为支持`GetAttributeValue`函数。但是，正如下面“当前限制”部分所述，`Idemix`案例中只公开了两个属性：`ou`和`role`。

如果`Fabric CA`是凭证颁发者：

+ `ou`属性的值是**身份的隶属关系**（例如“`org1.department1`”）;
+ `role`属性的值将**是`member`或`admin`**。值`admin`表示该**身份标识是`MSP`管理员**。默认情况下，`Fabric CA`创建的身份标识将返回`member`角色。要创建`admin`身份标识，请使用`role`属性和值`2`注册身份标识。

有关使用`cid`库检索这些属性的示例，请[参阅此`Java SDK`示例](https://github.com/hyperledger/fabric-sdk-java/blob/master/src/test/fixture/sdkintegration/gocc/sampleIdemix/src/github.com/example_cc/example_cc.go)。

# 目前的局限

当前版本的`Idemix`确实有一些限制。

+ **固定的属性集**

  尚**无法使用自定义属性发布或使用`Idemix`凭据**。将来的版本将支持自定义属性。

  目前支持以下四个属性：

  + **组织单位属性**（`ou`）：
    + 用法：与X.509相同
    + `Type`：`String`
    + `Revealed`: `always`
  + **角色属性**（`role`）：
    + 用法：与X.509相同
    + `Type`：`Integer`
    + `Revealed`: `always`
  + **注册`ID`属性**：
    + `Usage`：**唯一标识用户**，在属于同一用户的所有注册凭据中相同（将在未来版本中用于审核）
    + `Type`：`BIG`
    + `Revealed`：从不在签名中，仅在**为`Fabric CA`生成身份验证令牌**时
  + **撤销句柄属性**：
    - `Usage`：**唯一标识凭证**（将在未来版本中用于撤销）
    - `Type`：`integer`
    - `Revealed`：`never`

+ **撤销尚不支持**

  尽管上面提到的撤销句柄属性的存在可以看到很多撤销框架已经到位，但尚不支持撤销`Idemix`凭据。

+ **同行身份不使用`Idemix`进行认可**

  目前，**对等体仅使用`Idemix MSP`进行签名验证**。使用`Idemix`进行**签名只能通过`Client SDK`完成**。`Idemix MSP`将支持更多角色（包括`peer`角色）。