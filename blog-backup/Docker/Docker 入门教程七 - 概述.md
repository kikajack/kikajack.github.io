[原文地址](https://docs.docker.com/engine/docker-overview/)

Docker 是开发、传输和运行应用程序的开放平台。Docker 使你可以将应用程序与基础架构分离开来，从而可以快速交付软件。借助 Docker，可以像管理应用程序一样管理基础架构。通过利用 Docker 的方法快速进行传输、测试和部署代码，可以显着缩短编写代码和在生产环境中运行代码之间的延迟。
#1. Docker 平台
Docker 提供在称为容器 container 的松散隔离的环境中打包运行应用程序的能力。隔离和安全性允许你在给定主机上同时运行多个容器。容器是轻量级的，因为容器不需要监控程序 hypervisor，而是直接在宿主机的内核上运行。这意味着相比使用虚拟机，可以在一台给定硬件的主机上运行更多的容器。甚至可以在虚拟机中运行 Docker 容器。

Docker 提供工具和平台来管理容器的整个生命周期：

- 使用容器开发应用及其支持组件。
- 容器成为分发和测试应用程序的单元。
- 准备好后，可以将应用程序部署到生产环境中，作为容器或协调服务。 无论生产环境是本地数据中心，云提供商还是两者的混合，一样工作。
#2. Docker Engine
Docker Engine 是 client-server 架构的应用程序，主要组件如下：

- 服务端守护进程，常驻内存（`dockerd` 命令）

- 程序用来指挥守护进程工作的 REST API 接口。

- 使用命令行接口（CLI）的客户端（ `docker` 命令）

![engine-components-flow](http://img.blog.csdn.net/20180224161243164?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

CLI 使用 Docker 的 REST API 接口通过脚本或命令来控制守护进程或者和守护进程交互。许多其他的 Docker 应用使用底层 API 和 CLI。

守护进程创建和管理 Docker 对象，比如镜像，容器，网络，卷（volume）。

注意：Docker 授权协议是 open source Apache 2.0 license。

更多信息参考 [Docker Architecture](https://docs.docker.com/engine/docker-overview/#docker-architecture) 或下面的第 4 部分。
#3. Docker 可以用来干啥？
##3.1 快速，一致的交付应用
Docker 通过使开发人员在使用本地容器提供应用程序和服务的标准化环境中工作，简化了开发生命周期。容器非常适合持续集成和持续交付（CI/CD）工作流程。

考虑下面的示例场景：

- 开发人员在本地写代码，通过 Docker 容器和同事共享。
- 通过 Docker 把应用程序上传到测试环境，执行自动和人工测试。
- 开发人员发现 bug 后，可以在开发环境修复，然后重新部署到测试环境来测试和验证。
- 测试完成后，向客户提供补丁程序时只要简单的把更新后的镜像推送到生产环境即可。
##3.2 响应式部署和缩放
Docker 的基于容器的平台支持高度可移植的工作负载（workloads）。 Docker 容器可以在开发人员的本地笔记本电脑上，数据中心的物理机或虚拟机上，云服务提供商上或混合环境中运行。

Docker 的可移植性和轻量级特性也使得动态管理工作负载变得非常容易，可以近乎实时地按业务需求扩展或拆分应用程序和服务。
##3.3 同样硬件上运行更多工作负载
Docker 轻量且速度快。和基于虚拟机管理程序 hypervisor 的虚拟机相比，Docker 可行且经济高效，因此你可以使用更多的计算能力来实现业务目标。Docker 非常适合需要用更少资源做更多事情的高密度环境和中小型环境的部署。
#4. Docker 架构
Docker 使用 client-server 架构。Docker 客户端可以和 Docker 守护进程通信，Docker 守护进程负责构建、运行和分发 Docker 容器。 Docker 客户端和守护进程可以在同一个系统上运行，也可以将 Docker 客户端连接到远程 Docker 守护进程。 Docker 客户端和守护进程使用 REST API 通过 UNIX 套接字或网络接口进行通信。
![architecture](http://img.blog.csdn.net/20180224164059136?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
##4.1 Docker daemon 守护进程
Docker 守护进程（`dockerd`）监听来自 Docker API 的请求，管理 Docker 对象（image、container、network、volume）。守护进程之间可以通信，实现对 Docker service 的管理。
##4.2 Docker 客户端
Docker 客户端（`docker`）是 Docker 用户与 Docker 交互的主要方式。当你使用类似 `docker run` 这样的命令时，客户端把这些命令使用 Docker API 发送到 `dockerd` 来执行命令。Docker 客户端可以和多个 Docker 守护进程通信。
##4.3 Docker registries 注册处
Docker registry 用于存储 Docker 镜像。Docker Hub 和 Docker Cloud 是所有人都可以使用的公共 registry，默认情况下，Docker 会在 Docker Hub 中寻找镜像。你可以运行自己的 registry。如果你使用了 Docker Datacenter (DDC)，它包含了 Docker Trusted Registry (DTR)。

当使用 `docker pull` 或 `docker run` 命令时，会从你配置的 registry 中获取所需镜像。当使用 `docker push` 命令时，镜像会上传到你配置的 registry 中。

Docker 商店允许你购买和销售 Docker 镜像或免费发布镜像。例如，可以购买包含软件供应商的应用程序或服务的 Docker 镜像，并使用该映像将应用程序部署到测试、暂存和生产环境中。可以通过获取新版本的镜像并重新部署容器来升级应用程序。
##4.4 Docker objects 对象
使用 Docker 时会创建和使用 image、container、network、volume、plugin 和其他对象。下面大概介绍一下部分对象：

###4.4.1 IMAGES 镜像
镜像是包含创建 Docker 容器的指令的只读模板。通常，一个镜像会基于另一个镜像做额外的定制。例如，可以构建一个基于 ubuntu 的镜像，但是安装了 Apache web 服务器和自己的应用程序，并且进行配置以使应用程序运行。

可以创建自己的镜像，也可以只使用别人创建并发布到 registry 中的镜像。要自己构建镜像，需要创建一个指定了每个步骤 Dockerfile 文件并运行。Dockerfile 文件中的每个指令在镜像中创建一个层 layer。当编辑完成 Dockerfile 文件并重构镜像，只有变更过的层才会重新构建。这就是为何镜像和虚拟化技术相比如此轻量、小、快速。
###4.4.2 CONTAINERS 容器
容器是镜像的可运行实例。可以通过 Docker API 或 CLI 创建、开始、停止、移动或删除容器。容器可以连接到一个或多个网络，连接到存储，甚至基于当前状态创建一个新镜像。

默认情况下，容器可以与其他容器和宿主机相对很好地隔离。可以控制如何隔离容器之间或容器与宿主机之间的网络、存储或其他底层子系统（underlying subsystems）。

容器通过镜像和创建或启动容器时的配置参数来定义。容器删除后，其所有未存储的状态变更都会消失。
###4.4.3 `docker run` 命令示例
下面的命令运行了一个 `ubuntu` 容器，在容器中运行了 ubuntu 中的 `/bin/bash`，通过本地命令行会话窗口交互式运行（由 `-i` 和 `-t` 参数指定）。
```
$ docker run -i -t ubuntu /bin/bash
```
这个命令运行时，会发生下面的事情（假设使用了默认的 registry 配置）：

1. 如果本地没有 ubuntu 镜像，Docker 会从你配置的 registry 中下载这个镜像，就像手工运行了 `docker pull ubuntu` 命令一样。

2. Docker 创建了一个新容器，就像手工运行了 `docker container create` 命令一样。

3. Docker 为容器分配一块可读写的文件系统作为最后一层 layer，从而允许运行中的容器在其文件系统中创建或修改文件或目录。

4. Docker 创建一个网络接口，由于没有指定任何网络选项，这里会把容器连接到默认网络。这包括为容器分配 IP 地址。默认情况下，容器可以使用宿主机的网络连接到外部网络。

5. Docker 启动容器并且执行 `/bin/bash`。因为容器是交互式运行的并且连接到你的终端窗口（由 `-i` 和 `-t` 参数指定），你可以使用键盘输入命令到容器中运行的 ubuntu，并且在终端看到结果。

6. 输入 `exit` 命令来结束 `/bin/bash` 命令时，容器也会停止，但不会删除。可以再次启动容器或删除容器。
###4.4.4 SERVICES 服务
通过 service 可以在 swarm 集群中一起运行的多个 Docker 守护进程上伸缩容器，swarm 集群可以有多个 manager，多个 worker。swarm 的每一个成员都是一个 Docker 守护进程，守护进程通过 Docker API 通信。service 使得可以定义想要的状态，比如任意时刻可用的服务副本的数量。默认情况下，service 会在所有工作节点之间实现负载均衡（load-balanced）。对于消费者来说，Docker service 表现为一个独立应用程序。Docker 引擎在 Docker 1.12 及更高版本中支持 swarm 模式。
#5. 底层技术
Docker 使用 [Go](https://golang.org/) 语言编写，利用 Linux 内核的几个特性来提供其功能。
##5.1 Namespaces 命名空间
Docker 通过名为 `namespaces` 的技术来提供名为容器的隔离工作空间。运行容器时，Docker 会为这个容器创建一系列的 namespaces。

这些 namespaces 提供了一层隔离。容器的每个方面都运行在独立 namespaces 中，并且其访问权限仅限于该 namespace。

Docker Engine 使用 Linux 上的下列 namespaces：

- The `pid` namespace: Process isolation (PID: Process ID).
- The `net` namespace: Managing network interfaces (NET: Networking).
- The `ipc` namespace: Managing access to IPC resources (IPC: InterProcess Communication).
- The `mnt` namespace: Managing filesystem mount points (MNT: Mount).
- The `uts` namespace: Isolating kernel and version identifiers. (UTS: Unix Timesharing System).
##5.2 Control groups 控制组
Linux 上的 Docker Engine 还依赖另一种名为 `control groups` （cgroups）的技术。cgroup 限制应用程序只能访问指定的资源。Control groups 允许 Docker Engine 在容器之间共享可用的硬件资源，并可以选择强制实施限制和约束。例如，可用对某个容器限制可用的内存资源。
##5.3 Union file systems
Union file systems 或 UnionFS 是通过创建层来操作的文件系统，使得它们非常轻巧和快速。Docker 引擎使用 UnionFS 为容器提供构建块。Docker 引擎可以使用 UnionFS 的多种变体，包括 AUFS、btrfs、vfs 和 DeviceMapper。

>UnionFS 就是把不同物理位置的目录合并，然后挂载 mount 到同一个目录中。
##5.4 Container format
Docker 引擎把 namespaces、control groups 和 UnionFS 打包到一起，称为 `container format`。默认的 `container format` 是 `libcontainer`。Docker 在将来可能会通过集成 BSD Jails 或 Solaris Zones 来支持其他的 `container format`。
#6. 下一步
- [安装 Docker](https://docs.docker.com/install/)
- 通过 [Getting started with Docker tutorial](https://docs.docker.com/get-started/) 获得动手经验