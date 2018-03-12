[原文地址](https://docs.docker.com/get-started/part4/)

#1. 先决条件 Prerequisites
- 安装的 Docker 版本在 1.13 以上。
- 获取了上一章节讲的 [Docker Compose](https://docs.docker.com/compose/overview/)。
- 获取 [Docker Machine](https://docs.docker.com/machine/overview/)。在 Docker for Mac 和 Docker for Windows 中是默认安装的，但在 Linux 系统中需要 安装。Windows 10 之前的系统没有 Hyper-V，需要使用 Docker Tollbox。
- 读了第一部分和第二部分，知道如何创建容器。
- 确保已经发布了 friendlyhello 这个镜像并上传到了 registry。
- 确保你的镜像可以部署为容器并运行。运行这个命令，用你的信息替换 username、repo 和 tag：`docker run -p 80:80 username/repo:tag`，然后访问：`http://localhost/`。
- 上一章节的 `docker-compose.yml` 文件。
#2. 概述
在第 3 部分，我们启动了第 2 部分创建的应用，并通过放入 service 定义了在生产中这个应用该如何运行，在进程中扩大 5 倍。
这一部分，在集群 cluster 中部署这个应用，在多个机器上运行。通过将多台机器连接到称为 swarm 的“Dockerized”集群，使多容器，多机器应用成为可能。
#3. 理解 Swarm 集群
一个 swarm 就是一组运行 Docker 并加入一个集群的机器。在这之后，仍然可以使用之前使用的 Docker 命令，但是现在这些命令通过 swarm 管理器在集群上执行。swarm 中的机器可以是物理机或虚拟机。加入 swarm 后，机器称为节点 node。
Swarm 管理器可以使用多种策略来运行容器，例如“emptiest node” - 它可以使用容器填充使用率最低的机器。 或者“global”，它确保每台机器只获取指定容器的一个实例。 在 Compose 文件中指示 swarm 管理器使用这些策略。
Swarm 管理器是一个 swarm 中唯一可以运行用户命令或者授权其他机器作为 worker 加入 swarm 集群的机器。worker 只是在那里提供能力，并没有权力告诉任何其他机器可以做什么和不可以做什么。
到目前为止，你已经在本地机器上以“single-host”模式使用 Docker。 但是 Docker 也可以切换到“swarm”模式，这就是开启集群的方式。 启用 swarm 模式使当前的机器成为集群管理器。 从此，Docker 将在你管理的集群上运行命令，而不仅仅是在当前机器上。
#4. 设置 swarm
swarm 由多个物理机或虚拟机节点组成。基本概念足够简单：运行 `docker swarm init` 命令开启 swarm 模式，并且让当前机器成为 swarm 管理器，然后在其他机器上运行 `docker swarm join` 命令使它们加入 swarm 成为 worker。
Choose a tab below to see how this plays out in various contexts. We use VMs to quickly create a two-machine cluster and turn it into a swarm.
##4.1 创建集群
###4.1.1 本地机器上的虚拟机
####1. MAC, LINUX, WINDOWS 7 AND 8
需要可以创建虚拟机的 hypervisor，可以安装 [Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads)。**CentOS 环境下的详细安装步骤**可以 [参考这里](https://wiki.centos.org/zh/HowTos/Virtualization/VirtualBox)。
>附注：这里有一个版本的坑，VirtualBox 对 Linux 内核的版本有要求，需要根据报错来升级 Linux 内核。

使用 `docker-machine` 命令创建 2 台虚拟机，使用 VirtualBox driver：
```
docker-machine create --driver virtualbox myvm1
docker-machine create --driver virtualbox myvm2
```
####2. WINDOWS 10
Windows 10 有 Hyper-V，不需要安装 VirtualBox。如果使用了 Docker Toolbox 则默认安装好了 VirtualBox。

首先，快速为虚拟机创建虚拟交换机（virtual switch）以便共享，这样它们可以互相连接：

- 启动 `Hyper-V` 管理器
- 点击右键菜单中的 `Virtual Switch Manager`（虚拟交换机管理器）
- 点击类型为 `External`（外部） 的 `Create Virtual Switch`（新建虚拟交换机）
- 命名为 myswitch，选中复选框以共享主机的活动网络适配器。

现在，使用 Docker 提供的节点管理工具 `docker-machine` 创建 2 台虚拟机：
```
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm1
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm2
```
###4.1.2 列出虚拟机并获取 IP 地址
现在创建了两个名为 myvm1 和 myvm2 的虚拟机。

使用这个命令列出虚拟机并获取 IP 地址：
```
docker-machine ls
```
示例：
```
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
myvm1   -        virtualbox   Running   tcp://192.168.99.100:2376           v17.06.2-ce   
myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.06.2-ce   
```
###4.1.3 初始化 swarm 并添加节点
第一个机器会成为管理器，可以执行管理命令并授权其他 worker 加入 swarm。第二个机器会成为 worker。
可以使用 `docker-machine ssh` 命令向虚拟机发送命令。通过 `docker swarm init` 命令可以指示 myvm1 成为 swarm 管理器，并看到如下输出。
```
$ docker-machine ssh myvm1 "docker swarm init --advertise-addr <myvm1 ip>"
Swarm initialized: current node <node ID> is now a manager.

To add a worker to this swarm, run the following command:

  docker swarm join \
  --token <token> \
  <myvm ip>:<port>

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
**关于端口 2377 （swarm 管理器端口）和 2376（Docker 守护进程的端口）**，总是在端口 2377 上运行 `docker swarm init` 命令和`docker swarm join` 命令，或者不指定端口直接使用这个默认值。
`docker swarm init` 命令返回的机器 IP 地址包含端口 2376，这是 Docker 守护进程的端口，不要使用这个端口，否则 [会报错](https://forums.docker.com/t/docker-swarm-join-with-virtualbox-connection-error-13-bad-certificate/31392/2)。


**`--native-ssh` 标志**：[使用自己的 SSH](https://docs.docker.com/machine/reference/ssh/#different-types-of-ssh)
Docker 机器可以在发送命令到 swarm 管理器遇到麻烦时，通过选项指定使用自己系统上的 SSH。在调用 ssh 命令时添加 `--native-ssh` 标志即可：
```
docker-machine --native-ssh ssh myvm1 ...
```
可以看到上面的 `docker swarm init` 命令的响应包含了预配置的在要加入的节点上执行的 `docker swarm join` 命令。通过 `docker-machine ssh` 命令在 myvm2 机器上执行这个命令，将 myvm2 加入 swarm：
```
$ docker-machine ssh myvm2 "docker swarm join \
--token <token> \
<ip>:2377"

This node joined a swarm as a worker.
```
祝贺，现在配置好了 swarm。

在 swarm 管理器上运行 `docker node ls` 来查看这个 swarm 中的节点：
```
$ docker-machine ssh myvm1 "docker node ls"
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
brtu9urxwfd5j0zrmkubhpkbd     myvm2               Ready               Active
rihwohkh3ph38fhillhhb84sk *   myvm1               Ready               Active              Leader
```
###4.1.4 退出 swarm
在每个节点上运行 `docker swarm leave`。
#5. 在 swarm 集群上部署应用
困难的部分结束了。现在你可以重复第 3 部分的操作在 swarm 上部署应用。记住，只有 swarm 管理器可以执行 Docker 命令，worker 只提供能力。
##5.1 为 swarm 管理器配置 docker-machine shell
到目前为止，可以通过 `docker-machine ssh` 包装 Docker 命令与虚拟机交流。 另一种选择是运行 `docker-machine env <machine>` 来获取并运行一个命令，该命令将当前 shell 配置为与 VM 上的 Docker 守护程序进行通信。 此方法对下一步的使用更友好，因为它允许使用本地 docker-compose.yml 文件“远程”部署应用程序，而无需将其复制到任何位置。

执行命令 `docker-machine env myvm1`来配置 shell 以与 swarm 管理器 myvm1 进行通信。
不同的操作系统，配置 shell 的命令不同，示例如下。
###5.1.1 Mac 或 Linux 上的 shell 环境
运行 `docker-machine env myvm1` 来获取命令配置 shell 和 myvm1 交流：
```
$ docker-machine env myvm1
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/sam/.docker/machine/machines/myvm1"
export DOCKER_MACHINE_NAME="myvm1"
# Run this command to configure your shell:
# eval $(docker-machine env myvm1)
```
运行给定的命令来配置 shell 和 myvm1 交流：
```
eval $(docker-machine env myvm1)
```
运行 `docker-machine ls` 来验证现在 myvm1 是活动机器，通过星号 * 指示。
```
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
myvm1   *        virtualbox   Running   tcp://192.168.99.100:2376           v17.06.2-ce   
myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.06.2-ce   
```
###5.1.2 Windows 上的 shell 环境
运行 `docker-machine env myvm1` 来获取命令配置 shell 和 myvm1 交流：
```
PS C:\Users\sam\sandbox\get-started> docker-machine env myvm1
$Env:DOCKER_TLS_VERIFY = "1"
$Env:DOCKER_HOST = "tcp://192.168.203.207:2376"
$Env:DOCKER_CERT_PATH = "C:\Users\sam\.docker\machine\machines\myvm1"
$Env:DOCKER_MACHINE_NAME = "myvm1"
$Env:COMPOSE_CONVERT_WINDOWS_PATHS = "true"
# Run this command to configure your shell:
# & "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression
```
运行给定的命令来配置 shell 和 myvm1 交流：
```
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression
```
运行 `docker-machine ls` 来验证现在 myvm1 是活动机器，通过星号 * 指示。
```
PS C:PATH> docker-machine ls
NAME    ACTIVE   DRIVER   STATE     URL                          SWARM   DOCKER        ERRORS
myvm1   *        hyperv   Running   tcp://192.168.203.207:2376           v17.06.2-ce
myvm2   -        hyperv   Running   tcp://192.168.200.181:2376           v17.06.2-ce
```
##5.2 部署应用到 swarm 管理器上
现在 myvm1 是 swarm 管理器，可以在 myvm1 上执行 `docker stack deploy` 命令部署应用，以及本地副本 `docker-compose.yml`。这个命令需要几秒钟才能完成，并且部署需要一段时间才可用。通过 `docker service ps <service_name>` 命令来验证所有 service 部署完毕。
你通过 docker-machine shell 配置连接到 myvm1，并且您仍然可以访问本地主机上的文件。 确保你和之前在同一个目录下，这个目录中包含你在第 3 部分中创建的 `docker-compose.yml` 文件。

跟以前一样，运行下面的命令在 myvm1 上部署应用：
```
docker stack deploy -c docker-compose.yml getstartedlab
```
这样，应用就部署到了 swarm 集群上！

注意：如果你的镜像保存在私有 registry 中而不是 Docker Hub 中，需要使用 `docker login <your-registry>` 命令登录，然后在上面的命令中添加 `--with-registry-auth` 标志。例如：
```
docker login registry.example.com

docker stack deploy --with-registry-auth -c docker-compose.yml getstartedlab
```
这会使用加密的 WAL 日志将登录 token 从本地客户端传递到要部署服务的 swarm 节点。 有了这些信息，这些节点就能够登录到 registry 并获取镜像。

现在可以使用在第 3 部分使用的 Docker 命令了。只有这次注意到服务（和相关容器）已经在 myvm1 和 myvm2 上分配了。
```
$ docker stack ps getstartedlab

ID            NAME                  IMAGE                   NODE   DESIRED STATE
jq2g3qp8nzwx  getstartedlab_web.1   john/get-started:part2  myvm1  Running
88wgshobzoxl  getstartedlab_web.2   john/get-started:part2  myvm2  Running
vbb1qbkb0o2z  getstartedlab_web.3   john/get-started:part2  myvm2  Running
ghii74p9budx  getstartedlab_web.4   john/get-started:part2  myvm1  Running
0prmarhavs87  getstartedlab_web.5   john/get-started:part2  myvm2  Running
```
使用 `docker-machine env` 和 `docker-machine ssh` 命令连接虚拟机：

- 要将 shell 设置为与 myvm2 等其他机器通信，只需在相同或不同的 shell 中重新运行 `docker-machine env`，然后运行给定的命令以指向 myvm2。 这总是特定于当前的shell。 如果您切换为未配置的 shell 或打开一个新的 shell，则需要重新运行这些命令。 使用 `docker-machine ls` 列出机器，查看状态，获取 IP 地址，并找出您连接的是哪一个（如果有的话）。 要了解更多信息，请参阅 [Docker Machine 入门主题](https://docs.docker.com/machine/get-started/#create-a-machine)。
- 此外，可以用这种形式包含 shell 命令：`docker-machine ssh <machine> "<command>"`，这样可以直接登录到虚拟机，但不能立即访问本地主机上的文件。
- 在 Mac 和 Linux 上，可以使用 `docker-machine scp <file> <machine>:~` 来在机器之间复制文件，但是 Windows 用户必须借助类似 Git Bash 这样的终端才行。

这个教程演示了 `docker-machine ssh` 和 `docker-machine env`，因为这些都可以通过 `docker-machine` CLI在所有平台上使用。
##5.3 访问集群
可以通过 myvm1 或 myvm2 的 IP 地址来访问我们的应用。
你创建的网络在这些机器之间共享并且负载均衡。运行 `docker-machine ls` 命令来获取虚拟机的 IP 地址，然后在浏览器上访问并且刷新（或使用 CURL）。

![app-in-browser-swarm](http://img.blog.csdn.net/20180224100314863?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

有 5 个可能的 ID 随机循环，证明负载均衡生效了。两个 IP 地址都工作的原因是 swarm 中的节点参与入口路由网格（ingress routing mesh）。这可以确保部署在 swarm 中某个端口的服务始终保留该端口，而不管实际运行容器的节点是什么。以下是三节点 swarm 集群中端口 8080 上发布的名为 my-web 的服务的路由网格示意图：
![ingress-routing-mesh](http://img.blog.csdn.net/20180224100407222?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

连接遇到问题了吗？
注意，要使用 swarm 中的入口网络，必须要在开启 swarm 模式之前，开启 swarm 节点之间的下列端口：

- 端口 7946 TCP/UDP 用于容器网络探测。
- 端口 4789 UDP 用于容器入口网络。
#6. 迭代和伸缩应用程序
从这里开始，你可以尝试从第 2 部分和第 3 部分学到的所有东西。
通过变更 `docker-compose.yml` 文件来伸缩应用程序。
通过编辑代码来改变应用程序的表现，然后重构代码，将新镜像推送到 registry。

无论哪种情况，只需再次运行 `docker stack deploy` 来部署这些更改。
可以通过 `docker swarm join` 命令把任意的物理机或虚拟机加入这个 swarm 集群，从而将功能添加到了集群。然后运行 `docker stack deploy` 命令，这样应用程序就可以使用新的资源了。
#7. 清除和重启
##7.1 Stacks 和 swarms
可以通过 `docker stack rm` 命令关闭一个 stack。例如：
```
docker stack rm getstartedlab
```
保留还是移除 swarm？
如果要移除 swarm，可以在 worker 上执行 `docker-machine ssh myvm2 "docker swarm leave"` 命令，在 manager 上执行 `docker-machine ssh myvm1 "docker swarm leave --force"` 命令。下面的例子还要用到这个 swarm，所有暂时不要移除。
##7.2 重置 docker-machine shell 变量
通过下面的命令在当前 shell 中重置 docker-machine 环境变量：
```
eval $(docker-machine env -u)
```
这会断开 shell 和 `docker-machine` 创建的虚拟机的连接，使得可以在当前 shell 中继续使用本地的 Docker 命令进行其他工作。更多资料，参考 [Machine topic on unsetting environment variables](https://docs.docker.com/machine/get-started/#unset-environment-variables-in-the-current-shell)。
##7.3 重启 Docker machines
如果关闭了本地机器，Docker machines 也会停止运行。可以通过 `docker-machine ls` 命令查看机器状态：
```
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL   SWARM   DOCKER    ERRORS
myvm1   -        virtualbox   Stopped                 Unknown
myvm2   -        virtualbox   Stopped                 Unknown
```
要重启停止状态的机器，运行：
```
docker-machine start <machine-name>
```
例如：
```
$ docker-machine start myvm1
Starting "myvm1"...
(myvm1) Check network to re-create if needed...
(myvm1) Waiting for an IP...
Machine "myvm1" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.

$ docker-machine start myvm2
Starting "myvm2"...
(myvm2) Check network to re-create if needed...
(myvm2) Waiting for an IP...
Machine "myvm2" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.
```
#8. 概括和备忘录
这一部分，学习了 swarm 是啥，swarm 中的节点如何成为 manager 或 worker，创建 swarm，在 swarm 上部署应用。可以看到从第 3 部分学习的 Docker 核心命令并没有变化，只是现在是运行在 swarm 集群上了。你还看到了 Docker 网络的力量，即使容器运行在不同的机器上，也可以跨容器实现负载平衡。 最后，学习了如何在集群上迭代和伸缩应用程序。

以下是一些与 swarm 和虚拟机进行交互的命令：
```bash
docker-machine create --driver virtualbox myvm1 # 创建虚拟机(Mac, Win7, Linux)
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm1 # Win10
docker-machine env myvm1                # 查看节点基本信息
docker-machine ssh myvm1 "docker node ls"  # 列出 swarm 中的节点
docker-machine ssh myvm1 "docker node inspect <node ID>"        # 检查节点
docker-machine ssh myvm1 "docker swarm join-token -q worker"   # 查看加入 swarm 的 token
docker-machine ssh myvm1   # 打开虚拟机的 SSH 会话，"exit" 退出
docker node ls             # 在 swarm manager 登录后，列出 swarm 中的节点
docker-machine ssh myvm2 "docker swarm leave"  # 使 worker 退出 swarm
docker-machine ssh myvm1 "docker swarm leave -f" # 使 master 退出 swarm，关闭 swarm
docker-machine ls # 列出虚拟机，星号 表明当前会话在与哪台虚拟机交流
docker-machine start myvm1    # 重启停止运行的虚拟机
docker-machine env myvm1      # 查看 myvm1 的环境变量和命令
eval $(docker-machine env myvm1)    # Mac 命令，连接 shell 到 myvm1
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression   # Windows 命令，连接 shell 到 myvm1
docker stack deploy -c <file> <app>  # 部署应用；必须在连接到 manager 的 shell 中执行该命令，使用本地 Compose 文件
docker-machine scp docker-compose.yml myvm1:~ # 复制文件到节点的 home 目录（只在使用 ssh 连接到 manager 并部署应用时需要）
docker-machine ssh myvm1 "docker stack deploy -c <file> <app>"   # 通过 ssh 部署应用（确保此时 Compose 文件已经复制到了 myvm1）
eval $(docker-machine env -u)     # 断开 shell 和虚拟机的连接，使用本地 Docker 命令
docker-machine stop $(docker-machine ls -q)    # 停止所有虚拟机
docker-machine rm $(docker-machine ls -q) # 删除所有虚拟机和磁盘上的镜像
```