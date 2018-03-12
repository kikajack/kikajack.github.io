官网：https://www.docker.com/
国内网站：https://www.daocloud.io/
参考：
[一篇不一样的docker原理解析 提高篇](https://zhuanlan.zhihu.com/p/22403015)
[Docker源码分析](http://www.infoq.com/cn/profile/%E5%AD%99%E5%AE%8F%E4%BA%AE#)
[深入浅出Docker](http://www.infoq.com/cn/profile/%E8%82%96%E5%BE%B7%E6%97%B6)
[阮一峰博客相关文章](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)

环境配置：对于普通用户，安装的每一个软件都必须保证两件事：操作系统的设置正确，各种依赖的库和组件安装正确。对于软件开发人员，系统发布时需要保证生产环境与测试环境、本地开发环境的一致，才可以确保正常上线。

终极解决方案：**软件跟环境一同发布、安装。**
这样，就再也不会出现：“我的电脑上面没有问题啊”。

#一. Docker 简介
[参考这里](http://dockone.io/article/101)
说到容器，一定会想到 Docker。
容器和集装箱一样，是一个标准化的基础设施。
Docker 守护进程可以直接与宿主机的操作系统进行通信，为各个Docker容器分配资源；它还可以将容器与主操作系统隔离，并将各个容器互相隔离。
##1. Docker 是什么
- Docker 是一个开源的容器引擎。
- Docker 提供了轻量级的虚拟化，秒级启动，额外开销非常低。
- Docker 容器互相隔离，同一主机上运行多个容器时，不会互相影响。即一个容器中运行的应用程序，是访问不到其他容器的资源的（进程、网络、文件、用户等），除非配置为共享的资源。
- Docker 容器是可移植的，可以在几乎任何地方以相同的方式运行。这就可以确保应用在开发环境、测试环境、生产环境等都有完全一样的运行环境。
- Docker 容器是自包含的，包含一个软件组件及其所有的依赖——二进制文件、库、配置文件、脚本等，打包了应用程序及其所有依赖，可以直接运行。
- Docker 扩展了LXC，使用高层的API，提供轻量虚拟化解决方案来实现进程间隔离。
##2. Docker 和虚拟机的对比
Docker 和虚拟机都可以用于**软件跟环境一同发布、安装。**

虚拟机是一种**模拟系统**，在软件层面上通过模拟硬件的输入和输出，让虚拟机的操作系统得以运行在没有物理硬件的环境中（宿主机的操作系统上）。这个模拟硬件输入输出，让虚拟机上的操作系统可以启动起来的程序，叫做 **Hypervisor**。

1. 虚拟机需要维护一个个独立的操作系统，低效，隔离性好，Docker 只需要一个守护进程加多个容器进程，小巧、高效，但隔离性比虚拟机差。
2. 虚拟机的 Hypervisor 是**硬件层虚拟化**，每个虚拟机有独立的硬件（虚拟的）；容器技术是**操作系统层虚拟化**，每个容器有独立的操作系统（虚拟的），但共享宿主机的硬件。
3. Docker 架构：**硬件->主操作系统Linux->Docker 守护进程->Docker 容器**
虚拟机架构：**硬件->虚拟机管理系统HyperVisor->客户端操作系统->应用**（支持MacOS的HyperKit，支持Windows的Hyper-V、Xen以及支持 Linux 的KVM（Kernel-based Virtual Machine，基于内核的虚拟机）），<br>或**硬件->主操作系统->虚拟机管理系统HyperVisor->客户端操作系统->应用**（VirtualBox和VMWare workstation）。
4. 虚拟机能做到实时的热迁移、热克隆、挂起等高级操作，Docker 做不到。
5. 每个容器镜像的大小通常几十MB，每个虚拟机镜像则是完整的操作系统，高达几个GB。
6. 每个虚拟机有独立的 kernel，启动流程跟普通电脑一样：开机自检->启动 kernel->启动用户进程；每个容器共享宿主机的 kernel， 容器的 kernel version 由宿主机决定。
![这里写图片描述](http://img.blog.csdn.net/20171208141348720?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#二. Docker可以做什么
对开发来说，最重要的一点就是**统一环境配置**。Docker 提供了一个从开发到上线均一致的环境。
Docker 的用途：
- 统一开发环境：通过 Vagrant 封装一个 Linux 的开发环境。[步骤在此。](http://tao.logdown.com/posts/184078-dev-with-vagrant-and-docker)
- 统一环境配置：只要 Docker hub 上有镜像，pull 下来直接使用即可。
- 隔离应用：采用微服务时，可以每个容器只运行一个服务。
- 快速部署：启动时间秒级。
- PaaS，实现方便的服务创建、删除，做到可伸缩。可以快速构建一个 PaaS 容器，对不同的环境使用不同的 Docker 容器即可。
参考：[八个Docker的真实应用场景](http://dockone.io/article/126)

名称解释：

- PaaS（Platform as a Service，平台即服务，把服务器平台作为一种服务）。
- SaaS（Software as a Service，软件即服务，把应用作为一种服务）。
- DaaS（Data as a Service，数据即服务）。

#三. 安装
官方地址：太卡，不用
国内地址：https://get.daocloud.io/
##1. Linux
1. [官方教程在此](https://docs.docker.com/engine/installation/linux/docker-ce/centos/)
2. 通过脚本安装：执行下面命令即可，适用于 Ubuntu，Debian，Centos 等大部分 Linux。
`curl -sSL https://get.daocloud.io/docker | sh`
3. 对于CentOS7（必须是64位系统），可以用`yum`命令安装`yum install docker`。
##2. Windows
###1) win10 pro 及以上版本
 直接安装 Docker for Windows 即可。
###2) win10 pro 以下版本（Windows 7/8.1）
安装 Docker Toolbox。安装的时候，最好勾选所有的安装选项。
实际上是在本机安装了 VirtualBox 虚拟机，VirtualBox 虚拟机内安装有 Docker主机，在 Docker 主机中可以创建容器。
通过 `Docker Quickstart Terminal` 快捷方式可以启动Docker。初次启动时间较长（需要墙外的资源），等鲸鱼图标出现后，可以执行`docker version` 命令查看版本信息，执行`docker run hello-world` 命令确认安装是否成功。
##3. 安装好后，切换镜像源
默认的源太卡了，需要切换国内的官方加速器。
Windows 版本在设置->Daemon->Registry mirrors 中，复制这个地址即可：`https://registry.docker-cn.com`。

Linux 下，修改 Docker 配置文件 `/etc/default/docker` 如下：
```
DOCKER_OPTS="--registry-mirror=https://registry.docker-cn.com"
```
对于 1.12 或更高版本，创建或修改 `/etc/docker/daemon.json` 文件，内容如下
```
{
    "registry-mirrors": [
        "加速地址"
    ],
    "insecure-registries": []
}
```
##4. 每次下载镜像时，都可以单独指定仓库地址
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
#四. 基本使用
###1. 步骤
Docker Daemon 主进程运行常驻内存，处理服务请求。

1. 安装镜像
可以通过命令 `docker build Dockerfile文件` （基于 Dockerfile 文件安装）或 `docker 镜像名`（直接安装）或 `docker pull 镜像名`（只是下载到本地），从镜像仓库安装镜像到本地环境。
2. 运行容器
`docker run 镜像名`
###2. 示例
1. 安装 busybox 镜像到 Docker
`docker pull busybox`
BusyBox是一个最小的Linux系统。
2. 运行
`docker run busybox /bin/echo Hello Docker`
`docker run busybox` 表示启动镜像 busybox。
`/bin/echo` 表示运行镜像内的命令。
`Hello Docker` 表示传给运行命令的参数。