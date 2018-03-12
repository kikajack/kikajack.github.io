[原文地址](https://docs.docker.com/network/overlay/)

overlay 网络驱动程序在多个 Docker 守护进程主机之间创建一个分布式网络。这个网络在允许容器连接并进行安全通信的主机专用网络之上（overlay 覆盖在上面）。Docker 透明地处理每个 Docker 守护进程与目标容器之间的数据包的路由。

当初始化 swarm 集群或将一个 Docker 主机加入已经存在的 swarm 集群时，Docker 主机上会创建两个新网络：
- 一个称为 `ingress` 的 overlay 网络，用来处理与 swarm 服务相关的控制和数据流。当创建的 swarm 服务没有连接到用户自定义的 overlay 网络时，这个服务会默认连接到 ingress 网络。
- 一个称为 `docker_gwbridge` 的bridge 网络，用来将单个的 Docker 守护进程连接到 swarm 中的其他守护进程。

可以使用 `docker network create` 命令创建用户定义的 overlay 网络，就像可以创建用户定义的 bridge 网络一样。服务或容器一次可以连接到多个网络。服务或容器只能通过它们各自连接的网络进行通信。

尽管可以将 swarm 服务和独立容器连接到同一个 overlay 网络，但默认行为和配置规则不同。因此，本主题分为几个部分：适用于所有 overlay 网络的操作、适用于 swarm 服务网络的操作和适用于独立容器的 overlay 网络的操作。
#1. 适用于所有 overlay 网络的操作
##1.1 创建 overlay 网络
先决条件：

- Docker 守护进程的防火墙规则使用 overlay 网络
需要打开下面的端口才能访问 overlay 网络上的每个主机：
 - TCP 端口 2377：用于集群管理通信
 - TCP 和 UDP 端口 7946：用于节点之间通信
 - UDP 端口 4789：overlay 网络流量

- 在可以创建 overlay 网络之前，需要通过 `docker swarm init` 命令将你的 Docker 守护进程初始化为 swarm manager 或使用 `docker swarm join` 命令将其加入一个已经存在的 swarm。即使你不打算使用 swarm 服务，你也得这么干。这两种方式都会创建默认提供给 swarm 服务使用的 ingress overlay 网络。之后，可以创建其他用户自定义的 overlay 网络。

要通过创建 overlay 网络来使用 swarm 服务，可以用这样的命令：
```
$ docker network create -d overlay my-overlay
```
要创建可以用于 swarm 服务和独立容器跟其他 Docker 守护进程中运行的独立容器通信的 overlay 网络，添加 `--attachable` 标志：
```
$ docker network create -d overlay --attachable my-attachable-overlay
```
可以指定 IP 地址范围、子网、网关和其他选项。详情可以查看 `docker network create --help` 命令。
##1.2 加密 overlay 网络上的流量
所有的 swarm 服务管理流量默认使用 AES 算法 GCM 模式加密。swarm 中的 manager 节点每 12 小时旋转一次用于加密八卦数据（gossip data）的密钥。

如果要加密应用数据，需要在创建 overlay 网络时添加 `--opt encrypted`。这会在 vxlan 层级开启 IPSEC 加密。这种加密技术会带来不可忽视的性能损失，因此应该在生产中使用该选项之前对其进行测试。

添加 overlay 加密后，Docker 在所有的节点间创建 IPSEC 通道，这些节点上为连接到 overlay 网络的服务调度任务。这些通道也使用 AES 算法 GCM 模式加密，manager 节点每 12 小时自动旋转一次密钥。
>不要将 Windows 节点连接到加密过的 overlay 网络.
Windows 不支持 Overlay 网络的加密。如果一个 Windows 节点试图连接到加密过的 overlay 网络，不会报错但是节点无法通信。

**SWARM 模式 OVERLAY 网络和独立容器**
可以同时使用 `--opt encrypted`、 `--attachable` 选项来使用 overlay 网络，并将非托管容器连接到这个网络：（You can use the overlay network feature with both --opt encrypted --attachable and attach unmanaged containers to that network）
```
$ docker network create --opt encrypted --driver overlay --attachable my-attachable-multi-host-network
```
##1.3 配置默认的 ingress 网络
###1.3.1 概述
大部分用户不需要配置 ingress 网络，但是 Docker 17.05 及更高版本支持配置功能。如果自动选择的子网和网络上已经存在的子网发生冲突，或需要配置其他的底层网络选项例如 MTU 时，这个配置功能就很有用了。

配置 ingress 网络涉及到删除和重新创建这个网络。这通常在你在 swarm 中创建任何服务之前完成。如果已经存在暴露端口的服务，需要删除那些服务后才可以移除 ingress 网络。

在没有 ingress 网络工作期间，已经存在的那些不需要暴露端口的服务会继续工作但不会被负载均衡。这会影响到那些暴露端口的服务，比如暴露 80 端口的 WordPress 服务。
###1.3.2 步骤
#### 1. 停止服务
使用 `docker network inspect ingress` 检查 `ingress`，并删除容器连接到的 这个 `ingress` 的所有服务。这些是暴露端口的服务，例如暴露端口 80 的 WordPress 服务。如果这些服务未停止，则下一步将失败。

####2. 删除已经存在的 ingress 网络：
```
$ docker network rm ingress

WARNING! Before removing the routing-mesh network, make sure all the nodes
in your swarm run the same docker engine version. Otherwise, removal may not
be effective and functionality of newly created ingress networks will be
impaired.
Are you sure you want to continue? [y/N]

```
####3. 创建新的 overlay 网络
使用 `--ingress` 标志创建新的 overlay 网络，可以使用其他你想设置的选项。下面例子设置 MTU 为 1200，子网为 `10.11.0.0/16`， 网关为 `10.11.0.2`。
```
$ docker network create \
  --driver overlay \
  --ingress \
  --subnet=10.11.0.0/16 \
  --gateway=10.11.0.2 \
  --opt com.docker.network.mtu=1200 \
  my-ingress
```
注意：可以给你创建的 ingress 网络命名，不一定是 ingress，但是只能有一个名字。
####4. 重启步骤一中停止的服务
##1.4 配置 docker_gwbridge 接口
###1.4.1 概述
`docker_gwbridge` 是用于连接 overlay 网络（包括 ingress 网络）和独立 Docker 守护进程的物理网络的虚拟桥梁。Docker 会在初始化 swarm 或将一个 Docker 主机加入 swarm 时自动创建他，但它并不是 Docker 设备。它存在于 Docker 主机的内核中。如果需要自定义其设置，则必须在将 Docker 主机加入 swarm 之前或临时从 swarm 中暂时删除之后才能执行此操作。
###1.4.2 步骤
####1. 停止 Docker
####2. 删除已经存在的 `docker_gwbridge` 接口
```
$ sudo ip link set docker_gwbridge down

$ sudo ip link del name docker_gwbridge
```
####3. 开启 Docker，不要加入或初始化 swarm

####4. 创建 `docker_gwbridge`
通过 `docker network create` 命令使用自定义配置手动创建或手动重新创建 `docker_gwbridge`。下面的例子使用子网 `10.11.0.0/16`。完整的配置选项可以参考 [Bridge 驱动程序选项](https://docs.docker.com/engine/reference/commandline/network_create/#bridge-driver-options)。
```
$ docker network create \
--subnet 10.11.0.0/16 \
--opt com.docker.network.bridge.name=docker_gwbridge \
--opt com.docker.network.bridge.enable_icc=false \
--opt com.docker.network.bridge.enable_ip_masquerade=true \
docker_gwbridge
```
####5. 加入或初始化 swarm
由于该桥已经存在，因此Docker不会使用自动设置来创建它。
#2. 适用于 swarm 服务网络的操作
##2.1 开放  overlay 网络上的接口
连接到同一 overlay 网络的 swarm 服务可以有效地将所有端口暴露给对方。要使端口可以在服务之外访问，必须在 `docker service create` 或 `docker service update` 命令中使用 `-p` 或 `--publish` 标志发布该端口。支持传统的以冒号分隔的语法和较新的逗号分隔值语法。较长的语法是首选，因为它可以自我注释。
|Flag value|  描述|
|-|-|
|-p 8080:80 或 <br/> -p published=8080,target=80|  将服务的 80 端口映射到路由网格的 8080（都是 TCP）
|-p 8080:80/udp 或 <br/> -p published=8080,target=80,protocol=udp  |将服务的 80 端口映射到路由网格的 8080（都是 UDP）
|-p 8080:80/tcp -p 8080:80/udp 或 <br/> -p published=8080,target=80,protocol=tcp -p published=8080,target=80,protocol=udp| 将服务的 80 端口映射到路由网格的 8080（TCP 和 UDP 都分别映射）
##2.2 绕过 swarm 服务的路由网格
默认情况下，发布端口的 swarm 服务使用路由网格（routing mesh）来完成。当连接到任何 swarm 节点上的已发布端口（无论是否运行给定服务）时，都会透明地将连接重定向到正在运行该服务的 worker 节点。实际上，Docker 充当了 swarm 服务的负载平衡器。基于路由网格的服务以虚拟 IP（VIP）模式运行。即使在每个节点上运行的服务（通过 `--global` 标志）也使用路由网格。使用路由网格时，不能预知哪个 Docker 节点会服务客户端请求。

要绕过路由网格，可以使用 DNS Round Robin（DNSRR）模式启动服务，方法是将 `--endpoint-mode` 标志设置为 `dnsrr`。必须在服务前运行自己的负载均衡器。对运行在 Docker 主机上的服务名称的 DNS 查询会返回运行该服务的节点的 IP 地址列表。配置负载均衡器使用此列表并平衡各节点间的流量。
##2.3 分离控制流量和数据流量
默认情况下，尽管 swarm 控制流量是加密的，但 swarm 的控制流量和应用程序之间的控制流量运行在同一网络上。可以将 Docker 配置为使用单独的网络接口来处理两种不同类型的流量。初始化或加入 swarm 时，分别指定 `--advertise-addr` 和 `--datapath-addr`。必须为加入 swarm 的每个节点执行此操作。
#3. 适用于独立容器的 overlay 网络的操作
##3.1 将独立容器附加到 overlay 网络
不使用 `--attachable` 标志时创建的是 ingress 网络，这意味着只有 swarm 服务可以使用它，而独立容器则不可以。可以将独立容器连接到用户自定义的使用 `--attachable` 标志创建的 overlay 网络。这使得在不同 Docker 守护进程上运行的独立容器可以通信，而无需在各个 Docker 守护进程主机上设置路由。
##3.2 发布端口
|Flag value|  描述|
|-|-|
|-p 8080:80|  将容器的 80 端口映射到 overlay 网络的 8080 端口（TCP）
|-p 8080:80/udp|  将容器的 80 端口映射到 overlay 网络的 8080 端口（UDP）
|-p 8080:80/tcp -p 8080:80/udp| 将容器的 80 端口映射到 overlay 网络的 8080 端口（TCP 和 UDP）
#4. 下一步
- 学习 [overlay 网络教程](https://docs.docker.com/network/network-tutorial-overlay/)
- [从容器的角度了解网络](https://docs.docker.com/config/containers/container-networking/)