[原文地址](https://docs.docker.com/machine/overview/)

Docker Machine 可以用来：

- 在 Mac 或 Windows 上安装运行 Docker 
- 配置和管理多个远程Docker主机
- 配置 Swarm 集群
#1. Docker Machine 是什么
Docker Machine 是一个允许你在虚拟机上安装 Docker Engine 的工具，并且可以通过 `docker-machine` 命令管理这些主机。可以使用 Machine 在本地 Mac 或 Windows 上、在公司网络上、在数据中心或在 AWS、Azure 等云服务提供商上创建 Docker 主机。

通过 `docker-machine` 命令可以启动、检查、停止和重启托管主机，升级 Docker 客户端和守护程序，并配置 Docker 客户端以与主机通信。

将 Machine  CLI 指向正在运行的托管主机，就可以直接在该主机上运行 `docker` 命令。 例如，运行 `docker-machine env` 默认为指向一个名为 `default` 的主机，按照屏幕上的说明完成 `env` 设置，然后运行 `docker ps`、`docker run hello-world` 等命令。

在 Docker 1.12 之前的版本中，Machine 是在 Mac 或 Windows 上运行 Docker 的唯一方法。从 Docker v1.12 开始，[Docker for Mac](https://docs.docker.com/docker-for-mac/) 和 [Docker for Windows](https://docs.docker.com/docker-for-windows/) 作为可安装的应用程序提供，在台式机和笔记本上这是更好的选择。我们鼓励你尝试这些新应用程序。 Docker for Mac 和 Docker for Windows 的安装程序包括 Docker Machine 和 Docker Compose。

如果你不知道如何开始，参考 [Docker 入门 - 原版](https://docs.docker.com/get-started/) 或 [Docker 入门 - 中文版](http://blog.csdn.net/kikajack/article/details/79350391)，指导你完成Docker简要的端到端教程。
#2. 为何使用 Docker Machine
通过 Docker Machine 可以配置多个运行在不同发行版的 Linux 上的远程 Docker 主机。

此外，通过 Machine 还可以在老旧的 Mac 或 Windows 系统上运行 Docker。

Docker Machine 主要有下面两个用途：

- 需要在老旧的 Mac 或 Windows 桌面系统中运行 Docker
![Docker Machine on Mac and Windows](http://img.blog.csdn.net/20180227093825459?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
如果你使用较旧的 Mac 或 Windows 笔记本电脑或台式机，不符合新版 Docker for Mac 和 Docker for Windows 应用程序要求，则需要使用 Docker Machine 来运行 Docker Engine。使用 [Docker Toolbox](https://docs.docker.com/toolbox/overview/) 安装程序在 Mac 或 Windows 上安装 Docker Machine 可以为本地虚拟机配置 Docker Engine，使你能够连接这个虚拟机并运行 `docker` 命令。

- 需要配置远程系统上的多个 Docker 主机
![Docker Machine for provisioning multiple systems](http://img.blog.csdn.net/20180227093853846?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
Docker Engine 在 Linux 系统上本地运行。如果有一个 Linux 系统作为你的主系统，并且想运行 `docker` 命令，你需要做的就是下载并安装 Docker Engine。然而，如果您想要有效的配置在网络上，云上或本地的多个 Docker 主机，则需要 Docker Machine。

不管你的主系统是 Mac、Windows 还是 Linux，都可以在其上安装 Docker Machine 并使用 `docker-machine` 命令来配置和管理大量的 Docker 主机。Docker Machine 可以自动批量创建主机并安装 Docker Engine，然后配置 `docker` 客户端。每个托管主机（“machine”）都是一个 Docker 主机和已配置客户端的组合。
#3. Docker Engine 和 Docker Machine 的区别
我们谈起 Docker 时通常指的是 **Docker Engine**，由 Docker 守护进程、一个指明了与守护进程交互接口的 REST API 和一个用来与守护进程交互（通过这个 REST API）的命令行接口（CLI）客户端组成的 client-server 应用程序。Docker Engine 从 CLI 接受 Docker 命令，例如 `docker run <image>`，`docker ps` 来列出运行中的容器，`docker  image ls` 来列出镜像等命令。
![Docker Engine](http://img.blog.csdn.net/20180227095733061?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**Docker Machine** 是一个用于配置和管理容器化主机（已经安装了 Docker Engine 的主机）的工具。Docker Machine 一般安装在本地主机上。Docker Machine 有专用的命令行客户端 `docker-machine`，而 Docker Engine 使用的是 `docker`。可以使用 Machine 在一个或多个虚拟系统上安装 Docker Engine。这些虚拟系统可以是本地（用 Machine 在 Mac 或 Windows 上的虚拟机中安装运行 Docker Engine）或远程（用 Machine 配置云服务提供商的容器化主机）的。容器化主机通常被认为是托管的“machines”。
![Docker Machine](http://img.blog.csdn.net/20180227095754409?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#4. 下一步
- [安装 Docker Machine](https://docs.docker.com/machine/install-machine/)
- 在 [本地系统上通过 VirtualBox](https://docs.docker.com/machine/get-started/) 创建和运行 Docker 主机
- 配置 [云服务提供商](https://docs.docker.com/machine/get-started-cloud/) 的多个主机
- [通过 Docker Machine 配置 Docker Swarm 集群](https://docs.docker.com/swarm/provision-with-machine/)（swarm 遗留的）
- [swarm 模式入门](https://docs.docker.com/engine/swarm/swarm-tutorial/)（Docker Engine 1.12 或更高）
- [理解 Machine 概念](https://docs.docker.com/machine/concepts/)
- [Docker Machine 驱动参考](https://docs.docker.com/machine/drivers/)
- [Docker Machine 子命令 参考](https://docs.docker.com/machine/reference/)
- [从 Boot2Docker 迁移到 Docker Machine](https://docs.docker.com/machine/migrate-to-machine/)