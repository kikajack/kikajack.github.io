[原文地址](https://docs.docker.com/network/bridge/)

就网络而言，桥接网络（bridge network，也叫网桥）是一种链路层设备，用于转发网段之间的流量。 bridge 可以是硬件设备或在主机内核中运行的软件设备。

对 Docker 而言，桥接网络使用允许容器连接到同一个桥接网络来通信的软件网桥，同时提供与未连接到该桥接网络的容器的隔离。Docker bridge 驱动程序自动在主机中安装规则使不同桥接网络上的容器不能直接相互通信。

桥接网络用于在同一个 Docker 守护进程上运行的容器通信。对于不同 Docker 守护进程的容器，可以在操作系统层级管理路由或使用 overlay 网络来实现通信。

启动 Docker 时，会自动创建默认的桥接网络，新启动的容器如果没有特别指定都会连接到这个默认桥接网络。也可以创建用户自定义的桥接网络，且用户自定义的桥接网络比默认的优先级要高。
#1. 用户自定义 bridge 和默认 bridge 的差别
##1.1 用户定义网桥提供更好的隔离和容器化应用之间的互操作性
连接到同一个用户自定义网桥的容器会自动互相暴露所有端口，并且不会暴露到外部。这会让容器化应用之间的通信更方便，而不会意外开放进入外部世界。

假设一个应用包含 web 前端和数据库后端。外部需要访问前端（可能是 80 端口），但是只有前端需要访问数据库后端。使用用户自定义网桥，只需要将前端的端口暴露到外部，数据库应用不需要开启任何端口，因为 web 前端可以通过用户自定义网桥直接访问到。

如果在默认网桥上运行同一个应用堆栈，需要同时打开 web 前端和数据库后端的端口，每次都需要使用 `-p` 或 `--publish` 标志。在意味着 Docker 主机需要通过其他方式来限制对数据库后端端口的访问。
##1.2 用户自定义 bridge 提供容器间自动 DNS 解析（automatic DNS resolution）
默认网桥上的容器只能通过 IP 地址互相访问，除非你使用 `--link` 选项，这被认为是遗留的。在用户自定义网桥中，容器之间可以通过名字会别名互相访问。

这里还是用上面的例子分析，web 前端和数据库后端。如果容器称为 web 和 db，web 容器可以连接到 db 上的 db 容器（the web container can connect to the db container at db），不管这个应用堆栈运行在哪个 Docker 主机上。

如果在默认网桥上运行相同应用堆栈，需要人工创建容器之间的连接（使用遗留的 `--link`）标志。这些连接需要双向创建，所以当需要通信的容器个数大于 2 个时复杂度会呈指数增长。或者，你可以编辑容器内的 `/etc/hosts` 文件，但这会产生难以调试的问题。
##1.3 容器可以在运行中与用户自定义网络连接和断开
在一个容器的生命周期中，可以在容器运行中将容器与用户自定义网络连接和断开。要从默认网桥中移除容器，需要停止容器并且通过不同的网络选项重新创建。
##1.4 每个用户自定义网络创建一个可配置的桥
如果你的容器使用默认网桥，你可以配置它，但是所有容器都使用了相同设置，例如 MTU 和 iptables 规则。此外，对默认网桥的配置发生在 Docker 之外，需要重启 Docker。

用户自定义网桥通过 `docker network create` 来创建和配置。如果应用程序的不同分组有不同的网络需求，可以独立配置每个用户自定义网桥，就像独立创建一样。
##1.5 默认网桥中连接的容器共享环境变量
最初，在两个容器之间共享环境变量的唯一方法是使用 `--link` 标志连接它们。用户自定义网络中无法使用这种类型的变量共享方式。然而，共享环境变量有更好的方式。一些想法：

- 多个容器可以使用 Docker volume 卷挂载用于共享信息的同一个文件或目录。

- 可以通过 `docker-compose` 同时启动多个容器，compose 文件可以定义共享变量。

- 可以使用 swarm 服务代替独立的容器，可以利用 swarm 的共享的 [secrets](https://docs.docker.com/engine/swarm/secrets/) 和 [configs](https://docs.docker.com/engine/swarm/configs/)。

连接到同一个用户自定义网桥的容器可以有效地将所有端口暴露给对方。 要使不同网络上的容器或非 Docker 主机访问到容器的端口，该端口必须使用 `-p` 或 `--publish` 标志来发布。
#2. 管理用户自定义网桥
通过 `docker network create` 命令创建用户自定义网桥：
```
$ docker network create my-net
```
可以指定子网 subnet，IP 地址段，网关和其他选项。查看 [`docker network create` 命令参考手册](https://docs.docker.com/engine/reference/commandline/network_create/#specify-advanced-options) 或通过 `docker network create --help` 命令查看详情。

通过 `docker network rm` 命令删除用户自定义的网桥。如果容器仍然连接到网络，需要先断开连接才能删除这个网桥。
```
$ docker network rm my-net
```
>到达发生了什么？
当你创建或删除用户自定义网桥，或将容器从用户自定义网桥连接或断开，Docker 使用特定于操作系统的工具来管理底层网络架构（例如增删网桥设备或配置 Linux 上的 iptables 规则）。这些是具体的实现细节。让 Docker 替你管理你的用户自定义网桥就好了。
#3. 连接容器到用户自定义网桥
创建新容器时可以指定一个或多个 `--network` 标志。下面的例子将 Nginx 容器连接到 my-net 网络。同时还将容器的 80 端口发布到 Docker 主机的 8080 端口，这样外部的客户端就可以访问这个端口了。任何其他连接到 my-net 的网络的容器都可以访问这个网络中其他容器的所有端口，反之亦然。
```
$ docker create --name my-nginx \
  --network my-net \
  --publish 8080:80 \
  nginx:latest
```
使用 `docker network connect` 命令将运行中的容器连接到已经存在的用户自定义网桥。下面的命令将运行中的 my-nginx 容器连接到已经存在的 my-net 网络：
```
$ docker network connect my-net my-nginx
```
#4. 断开容器到用户自定义网络的连接
使用 `docker network disconnect` 命令断开运行中的容器到一个用户自定义网桥的连接。下面的命令将会断开 my-nginx 容器到 my-net 网络的连接：
```
$ docker network disconnect my-net my-nginx
```
#5. 使用 IPv6
如果需要 Docker 容器支持 IPv6，则需要在创建任何 IPv6 网络或为容器分配 IPv6 地址之前，在 Docker 守护进程中 [开启选项](https://docs.docker.com/config/daemon/ipv6/) 并重新加载配置。

创建网络时指定 `--ipv6` 标志可以开启 IPv6。默认网桥上不能有选择地禁用 IPv6。
#6. 开启容器到外部的访问
默认情况下，从容器发送到默认网桥的流量，并不会被转发到外部。要开启转发，需要改变两个设置。这些不是 Docker 命令，并且它们会影响 Docker 主机的内核。

- 配置 Linux 内核来允许 IP 转发
```
$ sysctl net.ipv4.conf.all.forwarding=1
```
- 改变 iptables 的政策，FORWARD 政策从 DROP 变为 ACCEPT
```
$ sudo iptables -P FORWARD ACCEPT
```
这些设置在重新启动时失效，因此可能需要将它们添加到启动脚本中。
#7. 使用默认网桥
默认桥接网络被视为 Docker 的遗留细节，不建议用于生产用途。配置默认网桥是一个手动操作，并且它有技术上的缺点。
##7.1 将容器连接到默认网桥
如果没有通过 `--network` 标志声明网络，并且指定了网络驱动程序，则默认情况下容器已连接到默认网桥。连接到默认桥接网络的容器可以进行通信，但只能通过 IP 地址进行通信，除非它们使用遗留标志 `--link` 进行链接。
##7.2 配置默认网桥
要配置默认网桥，需要在 `daemon.json` 配置文件中指定选项。下面的例子声明了几个选项。只需要在文件中指定需要自定义的设置。
```
{
  "bip": "192.168.1.5/24",
  "fixed-cidr": "192.168.1.5/25",
  "fixed-cidr-v6": "2001:db8::/64",
  "mtu": 1500,
  "default-gateway": "10.20.1.1",
  "default-gateway-v6": "2001:db8:abcd::89",
  "dns": ["10.20.1.2","10.20.1.3"]
}
```
重启 Docker 使变更生效。
##7.3 通过默认网桥使用 IPv6
如果 Docker 被配置为支持 IPv6（查看 [使用 IPv6](https://docs.docker.com/network/bridge/#use-ipv6)），则默认网桥会被自动配置为支持 IPv6。不像用户自定义网桥，不能在默认网桥中选择性的关闭 IPv6。
#8. 后续步骤
- 通过 [独立的网络教程](https://docs.docker.com/network/network-tutorial-standalone/)