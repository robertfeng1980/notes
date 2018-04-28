# docker 网络配置

# 概述

Docker容器和服务如此强大的原因之一是可以将它们连接在一起，或将它们连接到非Docker工作环境进行负载。Docker容器和服务甚至不需要知道它们是否部署在Docker上，或者它们的对等端是否也是Docker工作负载。无论Docker主机是运行Linux、Windows还是两者的组合，都可以使用Docker以平台无关的方式管理它们。


## 网络驱动

Docker的网络子系统是可插拔的，使用驱动程序。默认情况下存在几个驱动程序，并提供核心网络功能：

- `bridge`：默认的网络驱动程序。如果您不指定驱动程序，则这是您正在创建的网络类型。**当您的应用程序运行在需要通信的独立容器中时，通常会使用桥接网络。**请参阅 [桥接网络](https://docs.docker.com/network/bridge/)。
- `host`：对于独立容器，删除容器和Docker主机之间的网络隔离，并直接使用主机的网络。`host` 仅适用于Docker 17.06及更高版本的群集服务。请参阅 [使用主机网络](https://docs.docker.com/network/host/)。
- `overlay`：覆盖网络将多个Docker守护进程连接在一起，并使群集服务能够相互通信。您还可以使用覆盖网络来促进swarm服务和独立容器之间的通信，或者不同Docker守护进程上的两个独立容器之间的通信。这种策略消除了在这些容器之间进行操作系统级路由的需求。请参阅[覆盖网络](https://docs.docker.com/network/overlay/)。
- `macvlan`：Macvlan网络允许您为容器分配MAC地址，使其显示为网络上的物理设备。Docker守护进程通过其MAC地址将流量路由到容器。使用`macvlan` 驱动程序有时是处理希望直接连接到物理网络的传统应用程序的最佳选择，而不是通过Docker主机的网络堆栈进行路由。请参阅 [Macvlan网络](https://docs.docker.com/network/macvlan/)。
- `none`：对于此容器，禁用所有网络。通常与自定义网络驱动程序一起使用。`none`不适用于群组服务。请参阅 [禁用容器联网](https://docs.docker.com/network/none/)。
- [网络插件](https://docs.docker.com/engine/extend/plugins_services/)：您可以在Docker上安装和使用第三方网络插件。这些插件可从 [Docker Store](https://store.docker.com/search?category=network&q=&type=plugin) 或第三方供应商处获得。有关安装和使用给定网络插件的信息，请参阅供应商的文档。

### 网络驱动程序总结

- 当您需要多个容器在同一个Docker主机上进行通信时，**用户定义的网桥**是最好的。
- 当网络堆栈不应该与Docker主机隔离时，**主机网络**是最好的，但您希望隔离容器的其他方面。
- 当需要运行在不同Docker主机上的容器进行通信时，或者当多个应用程序使用群集服务一起工作时，**覆盖网络**是最好的。
- **Macvlan网络**最适合从虚拟机设置迁移或需要容器看起来像网络上的物理主机，每个物理主机都有一个唯一的MAC地址。
- **第三方网络插件**允许您将Docker与专用网络堆栈集成。

## Docker EE网络功能

以下两个功能只有在使用Docker EE和使用通用控制平面（UCP）管理Docker服务时才有可能：

- 该[HTTP路由网格](https://docs.docker.com/datacenter/ucp/2.2/guides/admin/configure/use-domain-names-to-access-services/) 可以让你分享多个服务之间的相同的网络IP地址和端口。根据客户端的要求，UCP使用主机名和端口的组合将流量路由到相应的服务。
- [会话粘性](https://docs.docker.com/datacenter/ucp/2.2/guides/user/services/use-domain-names-to-access-services/#sticky-sessions)允许您指定UCP用于将后续请求路由到相同服务任务的HTTP标头中的信息，用于需要有状态会话的应用程序。

## 网络教程

本主题定义了一些基本的Docker网络概念，并为您准备设计和部署应用程序以充分利用这些功能。
大部分内容适用于所有Docker安装。但是， [一些高级功能](https://docs.docker.com/network/#docker-ee-networking-features)仅适用于Docker EE客户。

现在您已经了解了Docker网络的基本知识，使用以下教程加深了您的理解：

- [独立网络教程](https://docs.docker.com/network/network-tutorial-standalone/)
- [主机网络教程](https://docs.docker.com/network/network-tutorial-host/)
- [覆盖网络教程](https://docs.docker.com/network/network-tutorial-overlay/)
- [Macvlan网络教程](https://docs.docker.com/network/network-tutorial-macvlan/)