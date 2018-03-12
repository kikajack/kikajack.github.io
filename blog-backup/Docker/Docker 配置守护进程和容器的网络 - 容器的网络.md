[原文地址](https://docs.docker.com/config/containers/container-networking/)

容器使用的网络类型（无论是 bridge、overlay、macvlan 网络还是自定义网络插件）在容器内是透明的。从容器的角度来看，它有一个带 IP 地址，网关，路由表，DNS 服务和其他网络细节的网络接口（假设容器没有使用 none 网络驱动程序）。这个话题是从容器的角度来看网络问题。
#1. 发布的端口
默认情况下，创建容器时，它不会将其任何端口发布到外部世界。要使端口可用于 Docker 之外的服务或未连接到容器网络的 Docker 容器，请使用 `--publish` 或 `-p` 标志。这会创建一个防火墙规则，将容器端口映射到 Docker 主机上的端口。这里有些例子。

|标志值| 描述|
|-|-|
|`-p 8080:80`|  将容器的 80 端口映射到 Docker 主机的 8080 端口（TCP）
|`-p 8080:80/udp`|  将容器的 80 端口映射到 Docker 主机的 8080 端口（UDP）
|`-p 8080:80/tcp -p 8080:80/udp`| 将容器的 80 端口映射到 Docker 主机的 8080 端口（TCP 和 UDP）
#2. IP 地址和主机名
默认情况下，容器会为其连接的每个 Docker 网络分配一个 IP 地址。IP 地址是从分配给网络的池中分配的，因此 Docker 守护程序有效地充当每个容器的 DHCP 服务器。每个网络也有一个默认的子网掩码和网关。

容器启动时，只能使用 `--network` 连接到单个网络。但是，可以使用 `docker network connect` 将正在运行的容器连接到多个网络。使用 `--network` 标志启动容器时，可以使用 `--ip` 或 `--ip6` 标志指定分配给该网络上容器的 IP 地址。

当通过 `docker network connect` 将已经存在的容器连接到其他网络时，可以使用 `--ip` 或 `--ip6` 标志指定附加网络上容器的 IP 地址。

同样，容器的主机名默认是 Docker 中的容器名称。可以使用 `--hostname` 覆盖主机名。使用 `docker network connect` 连接到现有网络时，可以使用 `--alias` 标志为该网络上的容器指定其他网络别名。
#3. DNS 服务
默认情况下，容器从 Docker 守护进程继承 DNS 设置，包括  `/etc/hosts` 和 `/etc/resolv.conf`。可以基于每个容器覆盖这些设置。

|标志|  描述|
|-|-|
|`--dns`|DNS 服务器的 IP 地址。可以使用多个 `--dns` 标志指定多个 DNS 服务器。如果容器无法连接到任一指定的 IP 地址，则会使用谷歌的公共 DNS 服务器 `8.8.8.8`，这样容器可以解析域名。
|`--dns-search`|一个 DNS 搜索域，用于搜索非完全限定的主机名。要指定多个 DNS 搜索前缀，请使用多个 `--dns-search` 标志。
|`--dns-opt`|代表 DNS 选项及其值的键值对。可用选项，请参阅操作系统的 `resolv.conf` 文档。
|`--hostname`|容器为自己使用的主机名。如果未指定，则默认为容器的名称。
#4. 代理服务器
如果容器需要使用代理服务器，参考 [Use a proxy server](https://docs.docker.com/network/proxy/)。