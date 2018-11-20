# `Identity Mixer MSP`配置生成器`idemixgen`

本文档描述了`idemixgen`实用程序的用法，该**实用程序可用于为基于`MSP`的身份混合器创建配置文件**。有两个命令可用，一个用于**创建新的`CA`密钥对**，另一个**用于使用以前生成的`CA`密钥创建`MSP`配置**。

# 目录结构

`idemixgen`工具将创建具有以下结构的目录：

```yml
- /ca/
    IssuerSecretKey
    IssuerPublicKey
    RevocationKey
- /msp/
    IssuerPublicKey
    RevocationPublicKey
- /user/
    SignerConfig
```

`ca`目录包含**颁发者密钥（包括撤销密钥）**，并且只应存在于`CA`。`msp`目录包含**设置`MSP`验证`idemix`签名所需的信息**。`user`用户目录指定**默认签名者**。

# `CA`密钥生成

使用`idemixgen ca-keygen`生成`ca`和`msp`目录后，可以使用`idemixgen signerconfig`将**用户目录中指定的默认签名者添加到配置中**。

```sh
$ idemixgen signerconfig -h
usage: idemixgen signerconfig [<flags>]

Generate a default signer for this Idemix MSP

Flags:
    -h, --help               Show context-sensitive help (also try --help-long and --help-man).
    -u, --org-unit=ORG-UNIT  The Organizational Unit of the default signer
    -a, --admin              Make the default signer admin
    -e, --enrollment-id=ENROLLMENT-ID
                             The enrollment id of the default signer
    -r, --revocation-handle=REVOCATION-HANDLE
                             The handle used to revoke this signer
```

例如，可以使用以下命令**创建一个默认签名者**，该签名者是**组织**单元`OrgUnit1`的成员，注册**身份**标识为`johndoe`，**撤销句柄**为`1234`，这是一个**管理员**：

```sh
$ idemixgen signerconfig -u OrgUnit1 --admin -e "johndoe" -r 1234
```



