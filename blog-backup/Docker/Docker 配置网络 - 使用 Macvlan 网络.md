[原文地址](https://docs.docker.com/network/macvlan/)

有些应用程序，尤其是遗留的或用于监控网络流量的程序，需要直接连接到物理网络。这种场景下，可以使用 `macvlan` 网络驱动程序将 MAC 地址分配给每个容器的虚拟网络接口，使得看上去是物理网络接口直接连接到物理网络。此时，需要指定 Docker 主机上的物理接口用于 Macvlan，以及 Macvlan 的子网和网关。甚至可以使用不同的物理网络接口来隔离你的 Macvlan 网络。记住以下几点：

- 由于 IP 地址耗尽或“VLAN spread”（这种情况下，您的网络中存在不适当的大量唯一 MAC 地址）非常容易导致网络意外损坏。
- 你的网络设备需要能够处理“混杂模式”（promiscuous mode），其中一个物理接口可以分配多个 MAC 地址。
- 如果你的应用程序可以使用桥接网络 bridge（在单个 Docker 主机上）或 overlay（在多个 Docker 主机之间通信），从长远看这两种解决方案更好。
#1. 创建 macvlan 网络
创建的 macvlan 网络可以是 bridge 模式或 802.1q trunk bridge 模式。

- bridge 模式中，Macvlan 流量通过主机上的物理设备。
- 802.1q trunk bridge 模式中，流量通过 Docker 在运行中创建的802.1q 子接口。这使你可以更细粒度地控制路由和过滤。
##1.1 Bridge 模式
要创建与给定物理网络接口桥接的 Macvlan 网络，请在 `docker network create` 命令中使用 `--driver macvlan`。 您还需要指定 `parent`，这是流量在 Docker 主机上实际通过的接口。
```
$ docker network create -d macvlan \
  --subnet=172.16.86.0/24 \
  --gateway=172.16.86.1  \
  -o parent=eth0 pub_net
```
如果需要排除在 Macvlan 网络中使用的 IP 地址，例如当某个给定的 IP 地址已被占用时，请使用 `--aux-addresses`：
```
$ docker network create -d macvlan  \
  --subnet=192.168.32.0/24  \
  --ip-range=192.168.32.128/25 \
  --gateway=192.168.32.254  \
  -o parent=eth0 macnet32
```
##1.2 802.1q trunk bridge 模式
如果你指定了一个带有点的 `parent` 接口名称，例如 `eth0.50`，Docker 会将其解释为 `eth0` 的子接口并自动创建子接口。
```
$ docker network  create  -d macvlan \
    --subnet=192.168.50.0/24 \
    --gateway=192.168.50.1 \
    -o parent=eth0.50 macvlan50
```
##1.3 使用 ipvlan 代替 macvlan
上面的例子中，你仍然使用 L3 桥。可以使用 `ipvlan` 来取代 L2 桥。指定 `-o ipvlan_mode = l2`。
```
$ docker network create -d ipvlan \
    --subnet=192.168.210.0/24 \
    --subnet=192.168.212.0/24 \
    --gateway=192.168.210.254  \
    --gateway=192.168.212.254  \
     -o ipvlan_mode=l2 ipvlan210
```
#2. 使用 IPv6
如果将 Docker 守护进程配置为允许 IPv6，你可以使用双栈 IPv4/IPv6 Macvlan 网络。
```
$ docker network  create  -d macvlan \
    --subnet=192.168.216.0/24 --subnet=192.168.218.0/24 \
    --gateway=192.168.216.1  --gateway=192.168.218.1 \
    --subnet=2001:db8:abc8::/64 --gateway=2001:db8:abc8::10 \
     -o parent=eth0.218 \
     -o macvlan_mode=bridge macvlan216
```