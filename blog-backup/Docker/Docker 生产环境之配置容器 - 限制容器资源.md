[原文地址](https://docs.docker.com/config/containers/resource_constraints/)

默认情况下，容器没有资源限制，可以使用主机内核调度程序允许的给定资源。Docker 提供了一些方法来控制容器可以使用多少内存、CPU 或块 IO，并设置 `docker run` 命令的运行时配置标志。本节详细介绍了何时应该设置限制以及设置它们的可能影响。

许多这些功能需要您的内核支持 Linux 功能。通过 `docker info` 命令检查支持是否可用。如果在内核中禁用了某个功能，则可能会在输出结尾看到类似以下内容的警告：
```
WARNING: No swap limit support
```
请查阅操作系统文档以启用它们。更多知识 [参考这里](https://docs.docker.com/install/linux/linux-postinstall/#your-kernel-does-not-support-cgroup-swap-limit-capabilities)。
# 1. 内存
## 1.1 了解耗尽内存的风险
不让正在运行的容器消耗太多的主机内存是很重要的。在 Linux 主机上，如果内核检测到没有足够的内存来执行重要的系统功能，它会抛出一个 `OOME` 或 `Out Of Memory Exception`，并开始查杀进程以释放内存。任何进程都可能被杀掉，包括 Docker 和其他重要应用程序。如果终止了错误进程，有可能导致系统宕机。

Docker 尝试通过调整 Docker 守护进程的 OOM 优先级来降低这些风险，从而使其和系统上的其他进程相比更不容易被杀掉。容器的 OOM 优先级不调整。这使得单个容器被杀死的可能性要比 Docker 守护进程或其他系统进程被终止的可能性要大。不应该通过手动将守护程序或容器上的 `--oom-score-adj` 设置为极端负数，或通过在容器上设置 `--oom-disable-kill` 来尝试规避这些安全措施。

更多关于 Linux 内核的 OOM 管理资料，参考 [Out of Memory Management](https://www.kernel.org/doc/gorman/html/understand/understand016.html)。

可以通过以下方式降低 OOME 导致系统不稳定的风险：

- 在投入生产之前，执行测试以了解应用程序的内存需求。
- 确保应用程序仅在具有足够资源的主机上运行。
- 限制容器可以使用的内存，如下所述。
- 在 Docker 主机上配置 swap 时需要注意。swap 比内存更慢，性能更低，但可以提供缓冲区以防系统内存耗尽。
- 考虑将容器转换为 [服务](https://docs.docker.com/engine/swarm/services/)，并使用服务级别约束和节点标签来确保应用程序仅在具有足够内存的主机上运行。
## 1.2 限制容器对内存的访问
Docker 可以对内存实施两种限制：硬限制，允许容器使用不超过给定数量的用户或系统内存；软限制，允许容器使用尽可能多的内存，除非满足某些条件，例如内核检测到内存不足或主机上的争用。其中一些选项在单独使用或同时设置多个选项时会有不同的效果。

这些选项大多数都是一个正整数，后跟一个后缀 `b`，`k`，`m`，`g`，以表示字节，千字节，兆字节或千兆字节。

选项|	描述
:---:|-
`-m` 或 `--memory=`|	容器可用的最大内存。如果设置了这个值，最小可用内存是 4MB。
`--memory-swap`*|	允许容器放入磁盘 swap 中的内存数量。详情看下一小节。
`--memory-swappiness`|	默认情况下，主机内核可以交换容器使用的匿名页面的百分比。可以设置为介于0和100之间的值，以调整此百分比。参考下面小节。
`--memory-reservation`|	软限制。指定一个小于 `--memory` 的软限制，当 Docker 检测到主机上的争用或内存不足时，会采用这个限制来替换 `--memory`。如果使用这个限制，则必须将其设置为低于 `--memory`，以使其优先。不能保证容器不会超出限制。
`--kernel-memory`|	容器可以使用的最大内核内存量。允许的最小值是 `4m`。由于内核内存不能被换出，因此内核内存不足的容器可能会阻塞主机资源，这会对主机和其他容器产生副作用。
`--oom-kill-disable`|	默认情况下，如果发生内存不足（OOM）错误，内核会杀死容器中的进程。使用 `--oom-kill-disable` 选项可以更改此行为。注意只能在同时设置了`-m/--memory` 选项的容器上使用此选项，因为如果未设置 `-m` 标志，可能会耗尽主机的内存，导致内核需要终止主机系统的进程以释放内存。
更多关于 cgroup 和内存的资料参考 [Memory Resource Controller](https://www.kernel.org/doc/Documentation/cgroup-v1/memory.txt)。
## 1.3 `--memory-swap` 详情
`--memory-swap` 是一个修饰符标志，只有在 `--memory` 也被设置时才有意义。使用 swap 使得容器可以在耗尽所有可用 RAM 时，将多余的内存需求写入磁盘。对于经常将内存交换到磁盘的应用程序会有性能损失。

其设置可能会产生复杂的效果：

- 如果 `--memory-swap` 设置为正整数，那么 `--memory` 和 `--memory-swap` 都需要设置。`--memory-swap` 表示所有可用的内存和 swap 之和，并且 `--memory` 控制非 swap 内存数量。因此，如果 `--memory="300m"` 和 `--memory-swap="1g"`，则容器可以使用 300MB 内存和 700MB swap。
- 如果 `--memory-swap` 设置为 0，则会忽略这个设置。
- 如果 `--memory-swap` 设置的值与 `--memory`相同，并且 `--memory` 设置为正整数，则**容器无法访问 swap**。
- 如果 `--memory-swap` 未设置，并且 `--memory` 设置了，如果主机容器配置了交换内存，则容器会使用 `--memory` 设置值的两倍作为 swap 的大小。例如，如果 `--memory="300m"`，`--memory-swap`没有设置，则容器可以使用 300MB 内存和 600MB swap。
- 如果 `--memory-swap` 显式设置为 -1，允许容器使用无限制的 swap，直到达到主机系统可用值。

#### 禁止容器使用 SWAP
如果 `--memory-swap` 设置的值与 `--memory`相同，则**容器无法访问 swap**。这是因为 `--memory-swap` 设置的值是可用的内存与 swap 之和，而 `--memory` 是可用的物理内存量。
## 1.4 `--memory-swappiness` 详情
- 值为 0 时，关闭匿名页的 swap。
- 值为 100 时，所有匿名页都可以 swap。
- 默认情况下，如果没有设置 `--memory-swappiness`，会从主机继承这个值。
## 1.5 `--kernel-memory` 详情
内核内存限制以分配给指定容器的全部内存来表示。考虑以下情况：

- **无限内存，无限内核内存**：这是默认行为。
- **无限内存，有限内核内存**：当所有 cgroup 所需的内存大于主机上实际存在的内存时，这是合适的。可以将内核内存配置为永远不会覆盖主机上可用的内容，而需要更多内存的容器需要等待。
- **有限内存，无限内核内存**：整个内存是有限的，但内核内存不是。
- **有限内存，有限内核内存**：限制用户和内核内存可用于调试与内存相关的问题。如果某个容器对任意一种内存的使用数量超量，则会导致内存不足但不会影响其他容器或主机。在此设置下，如果内核内存限制低于用户内存限制，则内核内存用尽会导致容器遇到 OOM 错误。如果内核内存限制高于用户内存限制，则内核限制不会导致容器体验 OOM。

当打开任何内核内存限制时，主机会在每个进程的基础上跟踪“high water mark”（高位标记）统计信息，以便跟踪哪些进程（在这种情况下是容器）正在使用多余的内存。可以通过在主机上查看 `/proc/<PID>/status` 来查看每个进程。
# 2. CPU
默认情况下，每个容器对主机 CPU 的周期访问是无限的。可以设置各种约束来限制给定容器访问主机的 CPU 周期。大多数用户使用和配置默认的 CFS 调度器。在 Docker 1.13 及更高版本中，还可以配置实时调度器。
## 2.1 配置默认的 CFS 调度器
CFS 是用于普通 Linux 进程的 Linux 内核 CPU 调度程序。几个运行时标志允许配置容器的 CPU 资源访问量。使用这些设置时，Docker 会修改主机上容器的 cgroup 设置。

选项|	描述
-|-
`--cpus=<value>`|	指定容器可以使用多少可用 CPU 资源。例如，如果主机有两个 CPU，并且设置了 `--cpus="1.5"`，则该容器最多可以使用一个半 CPU。这相当于设置 `--cpu-period="100000"` 和 `--cpu-quota="150000"`。**Docker 1.13 和更高版本中可用。**
`--cpu-period=<value>`|	指定 CPU CFS 调度程序周期，该周期与 `--cpu-quota` 一起使用。默认为 100 微秒。大多数用户不会更改此默认值。如果使用 Docker 1.13 或更高版本，请改用 `--cpus`。
`--cpu-quota=<value>`|	在容器上添加一个 CPU CFS 配额。每个 `--cpu-period` 是允许容器访问 CPU 的微秒数。换句话说，`cpu-quota` / `cpu-period`（The number of microseconds per --cpu-period that the container is guaranteed CPU access. In other words, cpu-quota / cpu-period）。如果使用 Docker 1.13 或更高版本，请改用 `--cpus`。
`--cpuset-cpus`|	限制容器可以使用的特定 CPU 或内核。如果有多个 CPU，则容器可以使用的逗号分隔列表或连字符分隔的 CPU 范围。第一个 CPU 编号为 0。有效值可能为 `0-3`（使用第一，第二，第三和第四个CPU）或 `1,3`（使用第二个和第四个CPU）。
`--cpu-shares`|	软限制，仅在 CPU 周期受到限制时才会执行。将此标志设置为大于或小于默认值1024的值，以增加或减少容器的权重，并使其能够使用主机 CPU 周期的更多或更少的比例。当大量 CPU 周期可用时，所有容器都使用尽可能多的 CPU。`--cpu-shares` 不会阻止容器在 swarm 模式下进行调度。它优先考虑容器 CPU 资源的可用 CPU 周期（It prioritizes container CPU resources for the available CPU cycles）。它不保证或保留任何特定的 CPU 访问权限。

如果你有 1 个 CPU，下面的每个命令会保证容器每秒分配到至多 50％ 的 CPU 资源。

Docker 1.13 及更高版本：
```
docker run -it --cpus=".5" ubuntu /bin/bash
```
Docker 1.12 及更低版本：
```
$ docker run -it --cpu-period=100000 --cpu-quota=50000 ubuntu /bin/bash
```
## 2.2 配置实时调度器
在 Docker 1.13 及更高版本中，对于能使用 CFS 调度器的任务，可以通过实时调度器配置容器。需要在配置 Docker 守护进程和具体容器前，确保主机内核配置正确。

>警告：CPU 调度和优先级是先进的内核级特性。大多数用户不需要更改默认值。错误地设置这些值会导致主机系统变得不稳定或不可用。
#### 配置主机内核
通过运行 `zcat /proc/config.gz | grep CONFIG_RT_GROUP_SCHED` 命令或检查文件  `/sys/fs/cgroup/cpu.rt_runtime_us` 是否存在来验证 Linux 内核开启了 `CONFIG_RT_GROUP_SCHED`。有关配置内核实时调度程序的指导，请参阅操作系统的文档。

#### 配置 Docker 守护进程
要使用实时调度器运行容器，通过 ` --cpu-rt-runtime` 选项（每个运行周期的实时任务保留的最大微秒数）运行 Docker 守护进程。例如，在默认周期为 1000000 微秒（1秒）的情况下，设置 `--cpu-rt-runtime= 950000` 可确保使用实时调度程序的容器可以每 1000000 微秒周期运行 950000 微秒，并保留至少 50000 微秒用于非实时任务。要在使用 `systemd` 的系统上使此配置永久生效，请参阅 [使用 systemd 控制和配置 Docker](https://docs.docker.com/config/daemon/systemd/)。
#### 配置单个容器
通过 `docker run` 命令启动容器时，可以传递多个标志来控制容器的 CPU 优先级。有关合适值的信息，请查阅操作系统的文档或 `ulimit` 命令。

选项|	描述
-|-
`--cap-add=sys_nice`|	授予容器 `CAP_SYS_NICE` 功能，允许容器提升进程的 `nice` 值，设置实时调度策略，设置 CPU 关联和其他操作。
`--cpu-rt-runtime=<value>`|	容器在 Docker 守护进程的实时调度程序周期中可以以实时优先级运行的最大微秒数。还需要 `--cap-add=sys_nice` 标志。
`--ulimit rtprio=<value>`|	允许容器使用的最大实时优先级。还需要 `--cap-add=sys_nice` 标志。

下面示例在 `` 容器中设置了这三个标志：
```
$ docker run --it --cpu-rt-runtime=950000 \
                  --ulimit rtprio=99 \
                  --cap-add=sys_nice \
                  debian:jessie
```
如果 kernel 或 Docker 守护进程配置有误，这里会报错。
