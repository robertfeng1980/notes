# 如何发布JAR包到仓库

在实际开发中，大量拷贝重复的代码。在代码需要修改后，之前拷贝的地方都需要重新拷贝。如果将自己的代码写在一个通用的项目中，或者定义自己通用风格的简单框架、工具类打包成JAR，然后发布到网上或公司内部仓库，在后期需要修改的情况下，只需用更新JAR包重新发布一个版本就可以了。这样方便了自己的JAR包维护和管理，也非常方便使用。

## 发布JAR包到本地仓库

### 修改maven的`settings.xml`配置

在本地`maven`的`C:\Users\Administrator\.m2\settings.xml`中添加管理员密码配置如下：

```xml
<servers>
    <server>
        <id>releases</id> <!-- 这里的id和后面的配置对应 -->
        <username>admin</username>
        <password>123456</password>
    </server>
    <server>
        <id>Snapshots</id> <!-- 这里的id和后面的配置对应 -->
        <username>admin</username>
        <password>123456</password>
    </server>
</servers>
```

这里设置的用户名和密码都是`nexus`仓库的登陆账号和密码。

### 修改项目 `pom.xml`配置

在当前项目工程中的`maven`的 `pom` 中增加配置，填写要发布的仓库位置信息如下：

```xml
<distributionManagement>
    <repository>
        <id>releases</id> <!-- 这里的id和上面server的配置对应 -->
        <url>http://maven.test.com:7078/nexus/content/repositories/releases/</url>
    </repository>

    <snapshotRepository>
        <id>Snapshots</id> <!-- 这里的id和上面server的配置对应 -->
        <url>http://maven.test.com:7078/nexus/content/repositories/snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

这里的`url`和 `nexus`仓库的地址对应，配置完成后就可以执行发布命令完成发布动作。

### 发布JAR包到仓库

```sh
# 跳过编译单元测试和执行单元测试
$ mvn clean deploy -Dmaven.test.skip=true

# 跳过执行单元测试
$ mvn clean deploy -DskipTests
```

`skipTests` 和 `maven.test.skip` 区别

```sh
 -DskipTests，不执行测试用例，但编译测试用例类生成相应的class文件至target/test-classes下
 -Dmaven.test.skip=true，不执行测试用例，也不编译测试用例类。
```

### JAR包版本和名称设置

在发布JAR包的时候，`maven`会判断版本后面是否带了`-SNAPSHOT`，如果带了就发布到`snapshots`仓库，否则发布到`release`仓库。这里我们可以在`pom.xml`文件中，设置版本和名称信息

```xml
<groupId>org.springframework.data</groupId>
<artifactId>spring-data-commons</artifactId>
<packaging>jar</packaging>
<version>${project.release.version}</version>

<properties>
    <java.version>1.8</java.version>
    <project.release.version>1.0-SNAPSHOT</project.release.version>
</properties>

<profiles>
    <profile>
        <id>release</id>
        <properties>
            <project.release.version>1.0</project.release.version>
        </properties>
    </profile>
</profiles>
```

表达式 `${project.release.version}` 在 ` properties`节点中查找配置，找到版本信息 `1.0-SNAPSHOT`填充到 `version`节点。

那么在直接使用 `mvn deploy` 发布的时候，发布的JAR包名称为 `spring-data-commons-1.0-SNAPSHOT.jar`，并且发布到`snapshots `的仓库位置。

如果使用 `mvn deploy -P release` 发布的时候，其中`-P release`参数和`profile`中的`id`对应，发布的JAR包名称为 `spring-data-commons-1.0-release.jar`，并且发布到`releases `的仓库位置。

## 发布JAR包到Maven中央仓库

