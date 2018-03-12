[原文地址](https://docs.docker.com/network/network-tutorial-macvlan/)

本教程基于连接到 `macvlan` 网络的独立容器。在这种网络中，Docker 主机接受来自多个 MAC 地址和对应 IP 地址的请求，并将这些请求路由到合适的容器。
#1. 目标
本教程的目标是设置一个**桥接**的 `macvlan` 网络并将一个容器连接到该网络，然后设置一个 **802.1q trunked** `macvlan` 网络并将一个容器连接到该网络。
#2. 先决条件
- 大多数云提供商阻止 `macvlan` 网络。可能需要物理访问你的网络设备（You may need physical access to your networking equipment）。
- `macvlan` 网络驱动程序只能在 Linux 主机上工作。
- Linux 内核版本至少是 3.9，建议使用 4.0 或以上内核。
- 示例中假设你的网络接口是 `eth0`。请根据需要替换。
#3. 网桥示例
在这个简单的网桥示例中，流量经过 `eth0`，Docker 将流量路由到你的使用 MAC 地址的容器。要将设备连接到你的网络，你的容器需要是物理连接到网络上的（To network devices on your network, your container appears to be physically attached to the network.）。
####1. 创建名为 `my-macvlan-net` 的 macvlan 网络。定义子网、网关和根据实际环境定义的 `parent` 值。
```
$ docker network create -d macvlan \
  --subnet=172.16.86.0/24 \
  --gateway=172.16.86.1 \
  -o parent=eth0 \
  my-macvlan-net
```
可以通过 `docker network ls` 和 `docker network inspect pub_net` 命令来验证网络存在并且是 `macvlan` 网络。
####2. 启动 `alpine` 容器并连接到 `my-macvlan-net` 网络
`-dit` 标志后台启动容器，并且可以后面连接上这个容器。`--rm` 标志表示容器在停止后就被删除。
```
$ docker run --rm -itd \
  --network my-macvlan-net \
  --name my-macvlan-alpine \
  alpine:latest \
  ash
```
####3. 检查 `my-macvlan-alpine` 容器，注意 `Networks` 中的 `MacAddress` 关键字：
```
$ docker container inspect my-macvlan-alpine

...truncated...
"Networks": {
  "my-macvlan-net": {
      "IPAMConfig": null,
      "Links": null,
      "Aliases": [
          "bec64291cd4c"
      ],
      "NetworkID": "5e3ec79625d388dbcc03dcf4a6dc4548644eb99d58864cf8eee2252dcfc0cc9f",
      "EndpointID": "8caf93c862b22f379b60515975acf96f7b54b7cf0ba0fb4a33cf18ae9e5c1d89",
      "Gateway": "172.16.86.1",
      "IPAddress": "172.16.86.2",
      "IPPrefixLen": 24,
      "IPv6Gateway": "",
      "GlobalIPv6Address": "",
      "GlobalIPv6PrefixLen": 0,
      "MacAddress": "02:42:ac:10:56:02",
      "DriverOpts": null
  }
}
...truncated
```
####4. 通过运行两个 `docker exec` 命令来了解容器如何查看自己的网络接口
```
$ docker exec my-macvlan-alpine ip addr show eth0

9: eth0@tunl0: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
link/ether 02:42:ac:10:56:02 brd ff:ff:ff:ff:ff:ff
inet 172.16.86.2/24 brd 172.16.86.255 scope global eth0
   valid_lft forever preferred_lft forever
$ docker exec my-macvlan-alpine ip route

default via 172.16.86.1 dev eth0
172.16.86.0/24 dev eth0 scope link  src 172.16.86.2
```
####5. 停止容器（因为 `--rm` 标志，Docker 会删除容器），删除网络。
```
$ docker container stop my-macvlan-alpine

$ docker network rm my-macvlan-net
```
#4. 802.1q trunked bridge 示例
在 802.1q trunked 网桥示例中，流量流经 eth0 的子接口（称为 eth0.10），Docker 使用其 MAC 地址将流量路由到你的容器。要将网络中的设备联网，你的容器看起来就是物理连接到网络上的。
原文：
In the 802.1q trunked bridge example, your traffic flows through a sub-interface of eth0 (called eth0.10) and Docker routes traffic to your container using its MAC address. To network devices on your network, your container appears to be physically attached to the network.
####1. 创建名为 `my-8021q-macvlan-net` 的 macvlan 网络。定义子网、网关和根据实际环境定义的 `parent` 值。
```
$ docker network create -d macvlan \
  --subnet=172.16.86.0/24 \
  --gateway=172.16.86.1 \
  -o parent=eth0.10 \
  my-8021q-macvlan-net
```
通过 `docker network ls` 和 `docker network inspect pub_net` 命令验证网络是否存在，是不是 macvlan 网络，并且有 parent `0.10`。可以在 Docker 主机上使用 `ip addr show` 命令来验证接口 `eth0.10` 存在并且有独立的 IP 地址。
####2. 启动 `alpine` 容器并连接到 `my-8021q-macvlan-net` 网络
`-dit` 标志后台启动容器，并且可以后面连接上这个容器。`--rm` 标志表示容器在停止后就被删除。
```
$ docker run --rm -itd \
  --network my-8021q-macvlan-net \
  --name my-second-macvlan-alpine \
  alpine:latest \
  ash
```
####3. 检查 `my-second-macvlan-alpine` 容器，注意 `Networks` 中的 `MacAddress` 关键字：
```
$ docker container inspect my-second-macvlan-alpine

...truncated...
"Networks": {
  "my-8021q-macvlan-net": {
      "IPAMConfig": null,
      "Links": null,
      "Aliases": [
          "12f5c3c9ba5c"
      ],
      "NetworkID": "c6203997842e654dd5086abb1133b7e6df627784fec063afcbee5893b2bb64db",
      "EndpointID": "aa08d9aa2353c68e8d2ae0bf0e11ed426ea31ed0dd71c868d22ed0dcf9fc8ae6",
      "Gateway": "172.16.86.1",
      "IPAddress": "172.16.86.2",
      "IPPrefixLen": 24,
      "IPv6Gateway": "",
      "GlobalIPv6Address": "",
      "GlobalIPv6PrefixLen": 0,
      "MacAddress": "02:42:ac:10:56:02",
      "DriverOpts": null
  }
}
...truncated
```
####4. 通过运行两个 `docker exec` 命令来了解容器如何查看自己的网络接口
```
$ docker exec my-second-macvlan-alpine ip addr show eth0

11: eth0@if10: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
link/ether 02:42:ac:10:56:02 brd ff:ff:ff:ff:ff:ff
inet 172.16.86.2/24 brd 172.16.86.255 scope global eth0
   valid_lft forever preferred_lft forever
$ docker exec my-second-macvlan-alpine ip route

default via 172.16.86.1 dev eth0
172.16.86.0/24 dev eth0 scope link  src 172.16.86.2
```
####5. 停止容器（因为 `--rm` 标志，Docker 会删除容器），删除网络。

```
$ docker container stop my-second-macvlan-alpin

$ docker network rm my-8021q-macvlan-net
```