[原文地址](https://docs.docker.com/network/host/)

如果你为容器使用 `host` 网络驱动程序，则该容器的网络堆栈不与 Docker 主机隔离。例如，如果运行绑定到端口 80 的容器并使用 `host` 网络，则可以直接通过宿主机 IP 地址的端口 80 使用该容器的应用程序。

在Docker 17.06 及更高版本中，通过将 `--network host` 传递给 `docker container create` 命令，还可以为 swarm 服务使用 `host` 网络。在这种情况下，控制流量（与管理 swarm 和服务相关的流量）仍然通过 `overlay` 网络发送，但各个 swarm 服务容器使用 Docker 守护进程的 `host` 网络和端口发送数据。这会产生一些额外的限制。例如，如果服务容器绑定到端口 80，则只有一个服务容器可以在给定的 swarm 节点上运行。

如果您的容器或服务没有发布端口，则 `host` 网络不起作用。