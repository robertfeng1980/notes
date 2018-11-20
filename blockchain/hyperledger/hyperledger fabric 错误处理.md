# `hyperledger fabric` 错误处理

`Hyperledger Fabric`代码应使用`vendored`包`github.com/pkg/errors`来代替`Go`提供的标准错误类型。该软件包**允许轻松生成和显示带有错误消息的堆栈跟踪**。

# 使用说明

应使用`github.com/pkg/errors`代替对`fmt.Errorf()`或`errors.New()`的所有调用。使用此包**将生成一个调用堆栈**，该堆栈将附加到**错误消息**中。

**使用这个包非常简单，只需要轻松调整代码即可**：

+ 首先，需要导入`github.com/pkg/errors`。

+ 接下来，更新代码生成的所有错误，以使用对应的错误创建函数：`errors.New()，errors.Errorf()，errors.WithMessage()，errors.Wrap()，errors.Wrapf()`。

> **注意**：有关可用错误创建功能的完整文档，请参阅https://godoc.org/github.com/pkg/errors。另外，有关使用`Fabric`代码包的更具体指导，请参阅下面的一般准则部分。

最后，将任何记录器或`fmt.Printf()`调用的格式化指令从`％s`更改为`％+v`，以**打印调用堆栈以及错误消息**。

# `Hyperledger Fabric`中错误处理的一般准则

+ 如果正在处理用户请求，则**应记录错误并将其返回**。
+ 如果错误来自外部源，例如`Go`库或`vendored`包，请**使用`errors.Wrap()`包装错误以生成错误的调用堆栈**。
+ 如果错误来自另一个`Fabric`函数，则**使用`errors.WithMessage()`将错误消息添加到错误消息上下文**（如果需要），同时保持调用堆栈不受影响。
+ 不应该让错误异常**传播到其他包**。



