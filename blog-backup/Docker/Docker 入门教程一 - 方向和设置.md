[原文地址](https://docs.docker.com/get-started/)

Docker 入门教程总共 6 部分：

- 设置 Docker 环境（当前部分）
- 构建镜像 image 并运行为容器 container
- 缩放应用程序以运行多个容器
- 在整个集群中分发应用程序
- 通过添加后端数据库来 Stack 服务
- 部署应用程序到生产环境

#1. Docker 概念
Docker 是开发人员和系统管理员使用容器开发、部署和运行应用程序的平台。使用 Linux 容器来部署应用程序称为容器化（containerization）。容器并不是新概念，但通过容器轻松部署应用程序则是最近才实现的。
容器化正在变得越来越流行，因为容器有以下特点：

- Flexible 灵活性：即使是最复杂的应用也可以放入容器。
- Lightweight 轻量性：容器利用并共享主机内核。
- Interchangeable 通用性：可以即时部署更新和升级。
- Portable 便携性：可以构建本地应用，部署到云端，并在任何地方运行。
- Scalable 可扩展：可以增加和自动分发容器副本。
- Stackable 可堆叠：可以即时纵向堆叠服务。
##1.1 镜像和容器
镜像：Image，容器：containers。
镜像在运行时得到容器。镜像是一个可执行的并且包含运行应用程序时所需的一切资源（代码，运行时，库，环境变量，配置文件）的包。
容器是镜像的运行时实例，镜像运行时进入内存（即有状态或用户进程的镜像）。可以使用命令 `docker ps` 查看正在运行的容器的列表，就像在 Linux 中一样。
##1.2 容器和虚拟机
容器在 Linux 上本地运行，并与其他容器共享宿主机的内核（kernel）。容器运行一个独立的进程，不比其他可执行文件占用更多的内存，使其轻量化。
相比之下，每个虚拟机（VM，virtual machine）运行一个完整的“guest”操作系统，通过虚拟机管理程序 hypervisor 虚拟访问主机资源。 一般来说，虚拟机提供了包含大量资源的环境，而大多数应用程序并不需要这么多。
![容器和虚拟机对比](http://img.blog.csdn.net/20180223171730150?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
#2. 准备 Docker 环境
在 [支持的平台](https://docs.docker.com/engine/installation/#supported-platforms) 上安装 Docker 社区版（CE，Community Edition）或企业版（EE，Enterprise Edition）的 [维护中版本](https://docs.docker.com/engine/installation/#updates-and-patches)。

要完整的集成 Kubernetes：
- [Mac](https://docs.docker.com/docker-for-mac/kubernetes/) 版本需要 17.12.0-ce Edge 或更高。
- [Windows](https://docs.docker.com/docker-for-windows/kubernetes/) 版本需要 18.02.0-ce Edge 或更高。
##2.1 测试 Docker 版本
确保安装了合适版本的 Docker：
```
$ docker --version
Docker version 17.12.0-ce, build c97c6d6
```
运行 `docker version` (去掉了 `--`) 或 `docker info` 可以查看 Docker 的更详细信息：
```
$ docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 17.12.0-ce
Storage Driver: overlay2
...
```
注意：为了防止权限错误（并省略 `sudo` 的使用），将用户添加到 docker 用户组中。[参考这里](https://docs.docker.com/engine/installation/linux/linux-postinstall/)。
##2.2 测试 Docker 的安装功能
通过一个简单的镜像 `hello-world` 来测试 Docker 的安装功能是否正常：
```
$ docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
ca4f61b1923c: Pull complete
Digest: sha256:ca0eeb6fb05351dfc8759c20733c91def84cb8007aa89a5bf606bc8b315b9fc7
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```
列出所有下载到本机机器上的镜像：
```
$ docker image ls
```
列出容器 `hello-world`（由对应镜像产生），它在显示信息后立刻退出了。如果容器仍在运行中，就不需要 `--all` 参数了：
```
$ docker container ls --all
CONTAINER ID     IMAGE           COMMAND      CREATED            STATUS
54f4984ed6a8     hello-world     "/hello"     20 seconds ago     Exited (0) 19 seconds ago
```
#3. 概括和备忘单
```
## 列出 Docker 的 CLI 命令
docker
docker container --help

## 显示 Docker 版本和信息
docker --version
docker version
docker info

## 执行 Docker 镜像
docker run hello-world

## 列出 Docker 镜像
docker image ls

## 列出 Docker 容器（运行中，所有的，quit 模式下所有的）
docker container ls
docker container ls -all
docker container ls -a -q
```
#4. 第一部分的结论
容器化使得持续集成和持续部署（CI/CD）得以无缝实现。例如：

- 应用程序不再对系统有依赖。
- 更新可以推送到分布式应用程序的任何部分。
- 资源密度可以被优化。

使用 Docker，扩展应用程序就是启动新的可执行文件，而不是运行繁重的虚拟机。