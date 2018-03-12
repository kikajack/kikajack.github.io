[原文地址](https://docs.docker.com/config/daemon/systemd/)

许多 Linux 发行版通过 `systemd` 来启动 Docker 守护进程。本文包括几个关于如何自定义 Docker 设置的示例。
#1. 启动 Docker 守护进程
##1.1 手动启动
Docker 安装好后，需要启动 Docker 守护进程。大多数 Linux 发行版通过 `systemd` 来启动服务。如果你没有 `systemctl`，使用 `service` 命令。

- systemctl:
```
$ sudo systemctl start docker
```
- service:
```
$ sudo service docker start
```
##1.2 设置为开机启动
参考 [这里](https://docs.docker.com/install/linux/linux-postinstall//#configure-docker-to-start-on-boot)。
#2. 自定义 Docker 守护进程选项
有很多方法可以为你的 Docker 守护进程配置守护进程标志和环境变量。推荐的方法是使用独立于平台的 `daemon.json` 文件，该文件默认位于 Linux 上的 `/etc/docker/` 中。请参阅 [守护进程配置文件](https://docs.docker.com/engine/reference/commandline/dockerd//#daemon-configuration-file)。

可以使用 `daemon.json` 配置几乎所有守护进程配置选项。以下示例配置了两个选项。无法使用 `daemon.json` 机制配置的一件事是 HTTP 代理。
##2.1 运行时目录和存储驱动程序
你可能希望通过将 Docker 镜像、容器和卷所使用的磁盘空间移动到单独的分区来方便控制这个磁盘空间。

要实现这个目的，在 `daemon.json` 文件中设置下面的标志：
```
{
    "data-root": "/mnt/docker-data",
    "storage-driver": "overlay"
}
```
##2.2 HTTP/HTTPS 代理
Docker 守护进程在其启动环境中使用 `HTTP_PROXY`、`HTTPS_PROXY` 和 `NO_PROXY` 环境变量来配置 HTTP 或 HTTPS 代理的行为。不能通过 `daemon.json` 文件来配置这些环境变量。

这个例子会覆盖默认的 `default docker.service` 文件

如果你在 HTTP 或 HTTPS 代理服务器后面（例如在公司设置中），则需要将此配置添加到 Docker systemd 服务文件中。
####1. 为 docker 服务创建一个 systemd 放置目录：
```
$ sudo mkdir -p /etc/systemd/system/docker.service.d
```
####2. 创建添加了 `HTTP_PROXY` 环境变量的名为 `/etc/systemd/system/docker.service.d/http-proxy.conf` 的文件： 
```
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80/"
```
或者，如果你在 HTTPS 代理服务器后面，请创建添加了 `HTTPS_PROXY` 环境变量的名为 `/etc/systemd/system/docker.service.d/https-proxy.conf` 的文件：
```
[Service]
Environment="HTTPS_PROXY=https://proxy.example.com:443/"
```
####3. 如果你有内部 Docker 注册表，需要不用代理的连接，则可以通过 NO_PROXY 环境变量指定它们：
```
[Service]    
Environment="HTTP_PROXY=http://proxy.example.com:80/" "NO_PROXY=localhost,127.0.0.1,docker-registry.somecorporation.com"
```
或者，如果你在 HTTPS 代理服务器后面：
```
[Service]    
Environment="HTTPS_PROXY=https://proxy.example.com:443/" "NO_PROXY=localhost,127.0.0.1,docker-registry.somecorporation.com"
```
####4. 使更改生效：
```
$ sudo systemctl daemon-reload
```
####5. 重启 Docker：
```
$ sudo systemctl restart docker
```
####6. 验证是否加载配置：
```
$ systemctl show --property=Environment docker
Environment=HTTP_PROXY=http://proxy.example.com:80/
```
或者，如果你在 HTTPS 代理服务器后面：
```
$ systemctl show --property=Environment docker
Environment=HTTPS_PROXY=https://proxy.example.com:443/
```
#3. 配置 Docker 守护进程监听连接的位置
参考 [这里](https://docs.docker.com/install/linux/linux-postinstall/#control-where-the-docker-daemon-listens-for-connections)。
#4. 手动创建 systemd 单元文件
在没有包的情况下安装二进制文件时，可能需要将 Docker 与 systemd 集成。为此，请将两个单元文件（服务和套接字）从 [github 仓库](https://github.com/moby/moby/tree/master/contrib/init/systemd) 安装到 `/etc/systemd/system`。