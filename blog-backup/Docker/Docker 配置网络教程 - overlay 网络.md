[原文地址](https://docs.docker.com/network/network-tutorial-overlay/)

这部分教程是关于 swarm 服务相关的网络。分为四部分，可以在 Linux、Windows 或 Mac 上运行，但对于后面两部分，需要另一个运行在其他地方的 Docker 主机。

- 如何使用在初始化或加入 swarm 时 Docker 自动为你设置的**默认 overlay 网络**。这个网络并不是生产系统的最佳选择。
- 如何创建并使用**自定义 overlay 网络**来连接服务。生产环境中的服务建议用这种网络。
- 如何使用 overlay 网络**在不同 Docker 守护进程的独立容器之间通信**。
- 如何使用 overlay 网络**在一个独立容器和 swarm 服务之间通信**。需要 Docker 最低版本为 17.06。
#1. 先决条件
需要最少有一个单节点的 swarm，这意味着要在主机上启动 Docker 并运行 `docker swarm init`。当然也可以用多节点的 swarm。

最后一个例子要 Docker 最低版本为 17.06。
#2. 使用默认的 overlay 网络
这个例子中，启动了 `alpine` 服务并从单个服务容器的角度检查网络的特征。

当前教程不会深入操作系统层面讲述 overlay 网络的实现原理，而是从服务的角度聚焦 overlay 的功能。
##2.1 先决条件
需要三个能互相通信的物理机或虚拟 Docker 主机，都运行 17.03 或更高版本的 Docker，在没有涉及防火墙的同一网络上运行。

这三个主机分别称为 `manager`、`worker-1` 和 `worker-2`。`manager` 主机同时作为 manager 和 worker，既可以运行服务任务又可以管理 swarm。而 `worker-1` 和 `worker-2` 则只能作为 worker。

如果没有三台主机，那么一个简单的解决方案就是在云提供商（例如 Amazon EC2）上设置三台 Ubuntu 主机，所有这些主机都位于同一网络上，并允许该网络上所有主机的所有通信（使用诸如 EC2 安全组），然后按照 Ubuntu 上 Docker CE 的安装说明进行操作。
##2.2 演练
###2.2.1 创建 swarm
创建三个加入 swarm 的 Docker 主机并使用名为 `ingress` 的 overlay 网络互相连接。
####1. 在 `master` 上初始化 swarm
如果主机只有一个网络接口，可以使用 `--advertise-addr` 选项。
```
$ docker swarm init --advertise-addr=<IP-ADDRESS-OF-MANAGER>
```
记下打印的文本，因为这包含用于将 `worker-1` 和 `worker-2` 加入 swarm 的令牌。将令牌存储在密码管理器中是一个好主意。
####2. 在 `worker-1` 中加入 swarm
如果主机只有一个网络接口，可以使用 `--advertise-addr` 选项。
```
$ docker swarm --join --token <TOKEN> \
  --advertise-addr <IP-ADDRESS-OF-WORKER-1> \
  <IP-ADDRESS-OF-MANAGER>:2377
```
####3. 在 `worker-2` 中加入 swarm
如果主机只有一个网络接口，可以使用 `--advertise-addr` 选项。
```
$ docker swarm --join --token <TOKEN> \
  --advertise-addr <IP-ADDRESS-OF-WORKER-2> \
  <IP-ADDRESS-OF-MANAGER>:2377
```
####4. 在 `master` 上列出所有节点
```
$ docker node ls

ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
d68ace5iraw6whp7llvgjpu48 *   ip-172-31-34-146    Ready               Active              Leader
nvp5rwavvb8lhdggo8fcf7plg     ip-172-31-35-151    Ready               Active              
ouvx2l7qfcxisoyms8mtkgahw     ip-172-31-36-89     Ready               Active
```
可以使用 `--filter` 标志来按照规则过滤：
```
$ docker node ls --filter role=manager

ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
d68ace5iraw6whp7llvgjpu48 *   ip-172-31-34-146    Ready               Active              Leader

$ docker node ls --filter role=worker

ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
nvp5rwavvb8lhdggo8fcf7plg     ip-172-31-35-151    Ready               Active              
ouvx2l7qfcxisoyms8mtkgahw     ip-172-31-36-89     Ready               Active  
```
####5. 列出 `manager`、`worker-1` 和 `worker-2` 上的 Docker 网络
注意每个节点的网络都包含名为 `ingress` 的 overlay 网络和名为 `docker_gwbridge` 的 bridge 网络。下面只列出了 `manager` 的列表：
```
$ docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
495c570066be        bridge              bridge              local
961c6cae9945        docker_gwbridge     bridge              local
ff35ceda3643        host                host                local
trtnl4tqnc3n        ingress             overlay             swarm
c8357deec9cb        none                null                local
```
`docker_gwbridge` 网络将 `ingress` 网络连接到 Docker 主机的网络接口，这样流量就可以在 swarm 的 manager 和 worker 直接流动。如果你创建的 swarm 服务没有指定网络，它们会连接到 `ingress` 网络。建议为每个应用或协同工作的一组应用分配独立的 overlay 网络。下个步骤中，你会创建两个 overlay 网络并将一个服务连接到这两个网络。
###2.2.2 创建服务
####1. 在 `manager` 上创建名为 `nginx-net` 的 overlay 网络
```
$ docker network create -d overlay nginx-net
```
不需要在其他节点上创建 overlay 网络，因为这会在这些节点开始运行需要 overlay 网络的服务时自动创建。
####2. 在 `manager` 上创建连接到 `nginx-net` 的 5 个副本的 Nginx 服务
将服务通过 80 端口暴露到外部。所有的服务任务容器都可以互相通信，不需要开启任何端口。

注意：只能通过 manager 创建服务。
```
$ docker service create \
  --name my-nginx \
  --publish target=80,published=80 \
  --replicas=5 \
  --network nginx-net \
  nginx
```
当未指定 `--publish` 标志的模式时使用默认的 `ingress` 发布模式，这意味着如果通过浏览器访问 `manager`、`worker-1` 或 `worker-2` 上的 80 端口，则将连接到 5 个服务任务之一的 80 端口，即使在你浏览的节点上当前没有任何任务正在运行。如果要使用 host 模式发布端口，则可以将 `mode=host` 添加到 `--publish` 输出。但是，在这种情况下，还应该使用 `--mode global` 而不是 `--replicas=5`，因为只有一个服务任务可以绑定给定节点上的给定端口。
####3. 运行 `docker service ls` 来监控服务启动的进度，这可能需要几秒钟的时间。
####4. 检查 `manager`、`worker-1` 或 `worker-2` 上的 `nginx-net` 网络
记住，不需要在 `worker-1` 或 `worker-2` 上手动创建这个网络，因为 Docker 会自动创建。输出将很长，但请注意 Containers 和 Peers 部分。容器列出了从该主机连接到 overlay 网络的所有服务任务（或独立容器）。
####5. 在 `manager` 中通过 `docker service inspect my-nginx` 检查服务，注意端口信息和服务使用的终端信息。
####6. 创建新网络 `nginx-net-2`，然后更新服务使用这个网络代替 `nginx-net`：
```
$ docker network create -d overlay nginx-net-2
```
```
$ docker service update \
  --network-add nginx-net-2 \
  --network-rm nginx-net \
  my-nginx
```
####7. 运行 `docker service ls` 来验证服务更新完毕且任务已经重新部署
运行 `docker network inspect nginx-net` 来验证没有容器连接到这个网络。对 `nginx-net-2` 运行同样命令验证所有服务任务的容器已经连接上。

注意：即使 overlay 网络在需要的时候自动在 swarm 的 worker 节点上创建，并不会自动删除。
####8. 清除服务和网络。在 manager 上运行下面的命令
manager 会指示 worker 自动删除网络。
```
$ docker service rm my-nginx
$ docker network rm nginx-net nginx-net-2
```
#3. 使用用户自定义的 overlay 网络
##3.1 先决条件
假设 swarm 已经安装好，你已经登入 manager。
##3.2 演练
####1. 创建用户自定义的 overlay 网络
```
$ docker network create -d overlay my-overlay
```
####2. 使用 overlay 网络启动服务并将 80 端口发布到 Docker 主机的 8080 端口
```
$ docker service create \
  --name my-nginx \
  --network my-overlay \
  --replicas 1 \
  --publish published=8080,target=80 \
  nginx:latest
```
####3. 运行 `docker network inspect my-overlay` 验证 `my-nginx` 服务任务已经连接上
####4. 删除服务和网络
```
$ docker service rm my-nginx

$ docker network rm my-overlay
```
#4. 为独立容器使用 overlay 网络
这个例子做下面的事情：

- 初始化 `host1` 上的 swarm
- 将 `host2` 加入 swarm
- 创建一个可以加入的 overlay 网络
- 创建一个有 3 个副本的 `alpine` 服务，连接到 overlay 网络
- 在 `host2` 上创建独立的 `alpine` 容器，也连接到 overlay 网络
- 证明独立容器可以与服务任务进行通信，反之亦然
##4.1 先决条件
对于这个测试，你需要两个不同的并可以互相通信的 Docker 主机。版本需要在 17.06 以上。两个 Docker 主机的下列端口必须开启：

- TCP 端口 2377
- TCP 和 UDP 端口 7946
- UDP 端口 4789

一个简单的方法是设置两个虚拟机（本地或像 AWS 这样的云提供商），每个虚拟机都安装并运行 Docker。如果你使用的是 AWS 或类似的云计算平台，最简单的配置是使用安全组（security group），该安全组可以打开两台主机之间的所有传入端口以及客户端 IP 地址的 SSH 端口。

这个例子中的主机名为 `host1` 和 `host2`。
##4.2 演练
###4.2.1 设置 swarm
####1. 在 `host1` 上运行 `docker swarm init`，为与其他主机通信的接口指定 IP 地址。
```
(host1) $ docker swarm init --advertise-addr 192.0.2.1

Swarm initialized: current node (l9ozqg3m6gysdnemmhoychk9p) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-3mtj3k6tkuts4cpecpgjdvgj1u5jre5zwgiapox0tcjs1trqim-bfwb0ve6kf42go1rznrn0lycx \
    192.0.2.1:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
swarm 初始化完成，`host1` 同时是 manager 和 worker。
####2. 复制 `docker swarm join` 命令。打开新的终端，连接到 `host2`，并执行刚才复制的命令。添加 `--advertise-addr` 标志为与其他主机通信的接口指定 IP 地址。最后一个参数是 `host1` 的 IP 地址。
```
(host2) $ docker swarm join \
          --token SWMTKN-1-3mtj3k6tkuts4cpecpgjdvgj1u5jre5zwgiapox0tcjs1trqim-bfwb0ve6kf42go1rznrn0lycx \
          --advertise-addr 192.0.2.2:2377 \
          192.0.2.1:2377
```
如果命令执行成功会显示下面的内容：
```
This node joined a swarm as a worker.
```
否则 `docker swarm join` 命令会尝试。此时在 `node2` 上运行 `run docker swarm leave --force` 验证你的网络和防火墙设置并再次尝试。
###4.2.2 在 `host1` 上创建名为 `test-net` 的可加入的 overlay 网络
```
$ docker network create --driver=overlay --attachable test-net
```
不需要在 `host2` 上手动创建，因为系统会在需要时自动创建。
###4.2.3 在 `host1` 上启动连接到 `test-net` 的容器：
```
(host1) $ docker run -dit \
          --name alpine1 \
          --network test-net \
          alpine
```
###4.2.4 在 `host2` 上启动连接到 `test-net` 的容器：
```
(host2) $ docker run -dit \
          --name alpine2 \
          --network test-net \
          alpine
```
`-dit` 标志意味着容器后台运行，交互式（可以向其输入信息），使用 TTY（可以看到输入和输出信息）。

注意：没有办法阻止你在多个主机上使用同一个容器，但是如果你这么做会使得自动服务探测失效，并且你需要通过 IP 地址来访问容器。

验证 `host2` 上创建了 `test-net`：
```
(host2) $ docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
6e327b25443d        bridge              bridge              local
10eda0b42471        docker_gwbridge     bridge              local
1b16b7e2a72c        host                host                local
lgsov6d3c6hh        ingress             overlay             swarm
6af747d9ae1e        none                null                local
uw9etrdymism        test-net            overlay             swarm
```
###4.2.5 记住你在 `host1` 创建了 `alpine1`,在 `host2` 创建了 `alpine2`。现在在 `host1` 上连接到 `alpine2`：
```
(host1) $ docker container attach alpine2

#
```
自动服务探测会在 overlay 网络上的两个容器之间工作！！

在 attached 会话中，尝试从 `alpine2` pinging `alpine1`：
```
# ping -c 2 alpine1

PING alpine1 (10.0.0.2): 56 data bytes
64 bytes from 10.0.0.2: seq=0 ttl=64 time=0.523 ms
64 bytes from 10.0.0.2: seq=1 ttl=64 time=0.547 ms

--- alpine1 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.523/0.535/0.547 ms
```
这证明两个容器可以使用已经连接到 `host1` 和 `host2` 的 overlay 网络互相通信。

使用 CTRL + P CTRL + Q 顺序来退出 `alpine2`。
###4.2.6 停止容器并在每个主机上删除 `test-net`
因为 Docker 守护进程独立运行，这些容器是独立容器，你需要在每个主机上运行这些命令：
```
(host1) $ docker container stop alpine1
        $ docker container rm alpine1
        $ docker network rm test-net
(host2) $ docker container stop alpine2
        $ docker container rm alpine2
        $ docker network rm test-net
```
#5. 容器和 swarm 服务之间通信
##5.1 先决条件
Docker 版本在 17.06 或更高。
##5.2 演练
这个例子中，需要在同一个 Docker 主机上启动两个 `alpine` 容器，做一些测试来理解它们是如何互相通信的。
####1. 打开终端窗口
先列出网络。如果你从未在这个 Docker 守护进程上添加网络或初始化 swarm 的话应该看到下面的内容。可能会看到不同的网络，但至少应该看到这些：
```
$ docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
17e324f45964        bridge              bridge              local
6ed54d316334        host                host                local
7092879f2cc8        none                null                local
```
默认的 bridge 网络与 host 和 none 一块列出来了。后两者不是完全成熟的网络，而是用于启动直接连接到 Docker 守护进程主机的网络堆栈的容器，或者启动没有网络设备的容器。本教程将两个容器连接到 bridge 网络。
####2. 开启两个 `alpine` 容器运行 `ash`（这是 Alpine 的默认脚本工具而不是 `bash`）
` -dit` 标志意味着容器在后台运行，交互式（有能力接受输入）并有一个 TTY 窗口（可以看到输入和输出）。因为是后台启动，所以不会直接连接到容器。相反，容器在启动后会把 ID 打印出来。因为没有指定任何 `--network` 标志，容器会连接到默认的 bridge 网络。
```
$ docker run -dit --name alpine1 alpine ash

$ docker run -dit --name alpine2 alpine ash
```
确认所有容器都成功启动：
```
$ docker container ls

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
602dbf1edc81        alpine              "ash"               4 seconds ago       Up 3 seconds                            alpine2
da33b7aa74b0        alpine              "ash"               17 seconds ago      Up 16 seconds                           alpine1
```
####3. 检查 bridge 网络，确保容器已经连接
```
$ docker network inspect bridge

[
    {
        "Name": "bridge",
        "Id": "17e324f459648a9baaea32b248d3884da102dde19396c25b30ec800068ce6b10",
        "Created": "2017-06-22T20:27:43.826654485Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "602dbf1edc81813304b6cf0a647e65333dc6fe6ee6ed572dc0f686a3307c6a2c": {
                "Name": "alpine2",
                "EndpointID": "03b6aafb7ca4d7e531e292901b43719c0e34cc7eef565b38a6bf84acf50f38cd",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "da33b7aa74b0bf3bda3ebd502d404320ca112a268aafe05b4851d1e3312ed168": {
                "Name": "alpine1",
                "EndpointID": "46c044a645d6afc42ddd7857d19e9dcfb89ad790afb5c239a35ac0af5e8a5bc5",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```
在顶部附近，列出了有关 bridge 网络的信息，其中包括 Docker 主机和 bridge 网络（172.17.0.1）之间网关的 IP 地址。在容器密钥下，列出了每个连接的容器以及有关其 IP 地址（ `alpine1` 的 `172.17.0.2` 和 `alpine2` 的 `172.17.0.3`）的信息。
####4. 容器在后台运行，使用 `docker attach` 命令连接到 `alpine1`
```
$ docker attach alpine1

/ #
```
提示符合变为 `#` 提示你现在变成容器中的 root 用户。使用 `ip addr show` 命令在容器内部显示 `alpine1` 的网络接口：
```
# ip addr show

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
27: eth0@if28: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe11:2/64 scope link
       valid_lft forever preferred_lft forever
```
第一个接口是回环设备，暂时忽略掉。注意第二个接口的 IP 地址是 `172.17.0.2`，跟上一步显示的 `alpine1` 的地址一样。
####5. 在 `alpine1` 中，通过 pinging `baidu.com` 确保已经联网
`-c` 标志限制只发送两次 ping 尝试。
```
# ping -c 2 google.com

PING google.com (172.217.3.174): 56 data bytes
64 bytes from 172.217.3.174: seq=0 ttl=41 time=9.841 ms
64 bytes from 172.217.3.174: seq=1 ttl=41 time=9.897 ms

--- google.com ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 9.841/9.869/9.897 ms
```
####6. 现在尝试 ping 第二个容器
首先通过 IP 地址 `172.17.0.3` 来尝试：
```
# ping -c 2 172.17.0.3

PING 172.17.0.3 (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.086 ms
64 bytes from 172.17.0.3: seq=1 ttl=64 time=0.094 ms

--- 172.17.0.3 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.086/0.090/0.094 ms
```
成功了。然后通过容器名来 ping `alpine2` 容器，这会失败：
```
# ping -c 2 alpine2

ping: bad address 'alpine2'
```
####7. 使 `alpine2` 转到后台运行而不停止（CTRL + p CTRL + q ，按住 CTRL 的同时依次按下 p 和 q）。可以分别使用其他容器再实验一番。
####8. 停止并删除所有容器
```
$ docker container stop alpine1 alpine2
$ docker container rm alpine1 alpine2
```
记住，默认的 bridge 网络不建议在生产环境中使用。下一个教程会讲解用户自定义的 bridge 网络。