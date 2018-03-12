[原文地址](https://docs.docker.com/network/)

Docker 容器和服务如此强大的原因之一是可以将它们连接到一起，或将它们连接到非 Docker 工作负载。Docker 容器和服务甚至都不需要知道它们是否部署在 Docker 上，或者它们的对等端是否也是 Docker 工作负载。无论你的 Docker 主机运行的是 Linux、Windows 还是两者都有，都可以使用 Docker 以平台无关的方式管理它们。

本主题定义了一些基本的 Docker 网络概念，并使得你可以设计和部署应用程序以充分利用这些功能。

大部分内容适用于所有 Docker。但是，一些高级功能仅适用于 Docker EE 客户。
#1. 本主题的范围
本主题不讨论在操作系统层面 Docker 网络如何工作，所以这里不会出现 Docker 如何操作 Linux 上的 iptable 规则或如何操作 Windows 服务器上的路由规则，同样也不会出现 Docker 如何构造和封装数据包或处理加密。查看 [Docker and iptables](https://docs.docker.com/network/iptables/)  和 [Docker Reference Architecture: Designing Scalable, Portable Docker Container Networks](https://success.docker.com/Architecture/Docker_Reference_Architecture%3A_Designing_Scalable%2C_Portable_Docker_Container_Networks) 来了解更深层的技术。

此外，本主题不会讲解如何创建、管理并使用 Docker 网络。每个章节都包含相关文档和命令参考。
#2. 网络驱动程序
##2.1 概述
Docker 的网络子系统是可插拔的，使用驱动程序。默认情况下存在几个驱动程序，并提供核心网络功能：

- bridge：桥接网络，**默认的网络驱动程序**。不指定驱动程序时用的就是这个。当应用程序运行在需要通信的独立容器中时，通常会使用 bridge 网络模式。更多信息参考 [bridge networks](https://docs.docker.com/network/bridge/)。

- host：对于独立容器，删除容器和 Docker 主机之间的网络隔离，使得**容器可以直接使用主机的网络**。host 网络模式仅适用于 Docker （17.06 或更高版本）上运行的 swarm 服务。更多信息参考 [host networks](https://docs.docker.com/network/host/)。

- overlay：overlay **将多个 Docker 守护进程连接在一起并使 swarm 服务能够相互通信**。还可以使用 overlay 网络模式来实现 swarm 服务和独立容器之间的通信，或者不同 Docker 守护进程上的两个独立容器之间的通信。这种策略消除了在这些容器之间进行操作系统级路由的需求。更多信息参考 [overlay networks](https://docs.docker.com/network/overlay/)。

- macvlan：macvlan 网络模式将 MAC 地址分配给容器，使其显示为网络上的物理设备。Docker 守护进程通过其 MAC 地址将流量路由到容器。在处理**希望直接连接到物理网络的遗留应用程序**时，最佳选择是使用 macvlan 网络模式，而不是通过 Docker 主机的网络堆栈（Docker host’s network stack）进行路由。更多信息参考 [macvlan networks](https://docs.docker.com/network/macvlan/)。

- none：。关闭容器的所有网络。通常与自定义网络驱动程序一起使用。 none 不适用于 swarm 服务。更多信息参考 [none networks](https://docs.docker.com/network/none/)。

- [Network plugins](https://docs.docker.com/engine/extend/plugins_services/)：。可以在 Docker 中安装和使用第三方网络插件。可以从 [Docker Store](https://store.docker.com/search?category=network&q=&type=plugin) 或第三方供应商那里获取插件。
##2.2 网络驱动程序总结

- 同一个 Docker 主机上运行的容器需要通信时，用户定义的 bridge 是最佳选择。
- 不同 Docker 主机上运行的容器需要通信或 swarm 服务中的多个应用需要协同工作时，overlay 是最佳选择。
- 当你正在从虚拟机迁移，或者需要使你网络上的容器看起来像物理机一样都有一个独立的 MAC 地址，macvlan 是最佳选择。
- 第三方网络插件允许你将 Docker 与专用网络堆栈集成。
#3. Docker EE 网络特性
下面的特性仅仅在使用 Docker EE 并且通过 Universal Control Plane (UCP) 管理 Docker 服务时有效：

- The HTTP routing mesh allows you to share the same network IP address and port among multiple services. UCP routes the traffic to the appropriate service using the combination of hostname and port, as requested from the client.

- Session stickiness allows you to specify information in the HTTP header which UCP uses to route subsequent requests to the same service task, for applications which require stateful sessions.
#4. 网络教程
现在，你已经理解了 Docker 的网络基础，下面的教程可以加深理解：

- [bridge 网络教程](https://docs.docker.com/network/network-tutorial-standalone/)
- [Host 网络教程](https://docs.docker.com/network/network-tutorial-host/)
- [Overlay 网络教程](https://docs.docker.com/network/network-tutorial-overlay/)
- [Macvlan 网络教程](https://docs.docker.com/network/network-tutorial-macvlan/)