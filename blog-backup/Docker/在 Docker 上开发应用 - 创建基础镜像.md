[原文地址](https://docs.docker.com/develop/develop-images/baseimages/)

大多数 Dockerfile 都是在父镜像（parent image）的基础上开始的。如果你需要完全控制你的镜像，那你需要创建一个基础镜像（base image）。下面是对比：

- [父镜像](https://docs.docker.com/glossary/?term=parent%20image) 你创建的镜像基于的镜像。它引用 Dockerfile 中 FROM 指令的内容。Dockerfile 中的每个后续声明都会修改此父镜像。大多数 Dockerfile 都会从父镜像开始，而不是基本映像。但是，这些术语有时可以互换使用。

- [基础镜像](https://docs.docker.com/glossary/?term=base%20image) 在其 Dockerfile 中没有 `FROM` 行，或 `FROM scratch`。

这个主题展示了创建基础镜像的几种方式。具体的过程将在很大程度上取决于你想打包的 Linux 发行版。在下面有一些例子，鼓励提交 pull 请求以提供新（you are encouraged to submit pull requests to contribute new ones.）。
#1. 使用 tar 创建完整镜像
一般来说，在运行你想要打包为父映像的发行版的工作机器开始，尽管对于像 Debian 的 [Debootstrap](https://wiki.debian.org/Debootstrap) 这样的工具来说，你也可以使用它来构建 Ubuntu 镜像。

创建 Ubuntu 父镜像就跟下面的例子一样简单：
```
$ sudo debootstrap xenial xenial > /dev/null
$ sudo tar -C xenial -c . | docker import - xenial

a29c15f1bf7a

$ docker run xenial cat /etc/lsb-release

DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04 LTS"
```
在 Docker GitHub Repo 中有更多的创建父镜像的示例脚本：

- [BusyBox](https://github.com/moby/moby/blob/master/contrib/mkimage/busybox-static)
- CentOS / Scientific Linux CERN (SLC) on [Debian/Ubuntu](https://github.com/moby/moby/blob/master/contrib/mkimage/rinse) or on [CentOS/RHEL/SLC/etc.](https://github.com/moby/moby/blob/master/contrib/mkimage-yum.sh)
- [Debian / Ubuntu](https://github.com/moby/moby/blob/master/contrib/mkimage/debootstrap)
#2. 使用 scratch 创建简单的父镜像
可以使用 Docker 保留的最小镜像 scratch 作为构建容器的起点。使用 scratch“image”信号用于希望 Dockerfile 中的下一个命令成为镜像中的第一个文件系统层的构建过程。（Using the scratch “image” signals to the build process that you want the next command in the Dockerfile to be the first filesystem layer in your image.）

虽然 scratch 在 Docker hub 仓库中出现，但是无法获取、运行或将镜像标记为 scratch。相反，可以在 Dockerfile 中引用。例如，使用 scratch 创建一个最小容器：
```
FROM scratch
ADD hello /
CMD ["/hello"]
```
假设你通过 https://github.com/docker-library/hello-world/ 的下列指令构建了一个可执行示例“hello”并且通过 `-static` 标志编译完成，那么可以通过使用下面的 `docker build` 命令构建这个 Docker 镜像：
```
docker build --tag hello .
```
不要忘记最后的句点符号 `.`，它用来设置当前目录位构建上下文。

注意：因为 Docker for Mac 和 Docker for Windows 使用了 Linux 虚拟机，需要使用 Linux 二进制（Linux binary）而不是 Mac 或 Windows 二进制。可以使用 Docker 容器构建：
```
$ docker run --rm -it -v $PWD:/build ubuntu:16.04

container# apt-get update && apt-get install build-essential
container# cd /build
container# gcc -o hello -static -nostartfiles hello.c
```
使用 `docker run` 命令运行新镜像：
```
docker run --rm hello
```
这个例子创建了 hello-world 镜像。如果要测试，可以从这个 [镜像仓库](https://github.com/docker-library/hello-world) 中复制。
#3. 更多资源
有很多可用资源可以帮助你编写 Dockerfile。

- [Dockerfile 中所有指令的完整文档](https://docs.docker.com/engine/reference/builder/)
- 帮助你写出清晰可读易维护的 Dockerfile 的 [Dockerfile 最佳实践](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) 或 [Dockerfile 最佳实践 - 中文版](http://blog.csdn.net/kikajack/article/details/79366043)
- 如果你想创建一个正式仓库，参考 Docker 的 [Official Repositories](https://docs.docker.com/docker-hub/official_repos/)