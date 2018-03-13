[原文地址](https://docs.docker.com/config/containers/runmetrics/)

# 1. Docker stats
可以通过 `docker stats` 命令实时流式传输容器的运行时指标。该命令支持CPU，内存使用情况，内存限制和网络 IO 指标。

下面是 `docker stats` 命令的输出示例：
```
$ docker stats redis1 redis2

CONTAINER           CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O
redis1              0.07%               796 KB / 64 MB        1.21%               788 B / 648 B       3.568 MB / 512 KB
redis2              0.07%               2.746 MB / 64 MB      4.29%               1.266 KB / 648 B    12.4 MB / 0 B
```
[docker stats](https://docs.docker.com/engine/reference/commandline/stats/)  手册上有关于 `docker stats` 命令的更多信息。
# 2. 控制组（Control groups）
Linux 容器依赖于 [控制组](https://www.kernel.org/doc/Documentation/cgroup-v1/cgroups.txt)，这些组不仅跟踪进程组，还暴露有关 CPU，内存和块 I/O 使用情况的指标。也可以访问这些指标并获取网络使用指标。这与“pure” LXC 容器以及 Docker 容器有关。

控制组通过 pseudo 文件系统暴露。在最近的发行版中，应该在 `/sys/fs/cgroup` 下找到这个文件系统。在该目录下，会看到多个子目录，称为 devices，freezer，blkio 等；每个子目录实际上对应于不同的 cgroup 层次结构。

旧系统中，控制组可能会挂载到 `/cgroup`，没有明确的层次结构。在这种情况下，不是看到子目录，而是看到该目录中的一堆文件，并且可能有一些与现有容器相对应的目录。

可以通过下面的命令找出你的控制组挂载的位置：
```
$ grep cgroup /proc/mounts
```
## 2.1 枚举 cgroups
可以查看 `/proc/cgroups` 以查看系统已知的不同控制组子系统，它们所属的层次结构以及它们包含的组数。

可以查看 `/proc/<pid>/cgroup` 以查看某个进程属于哪个控制组。控制组显示为相对于层次结构挂载位置根目录的路径。`/` 表示该进程尚未分配给组，而 `/lxc/pumpkin` 则表示进程是名为 pumpkin 的容器的成员。
## 2.2 找出指定容器的 cgroup
对每个容器而言，每个层次结构中创建一个cgroup。采用在较旧版本的 LXC userland 工具的旧系统上，cgroup 的名称是容器的名称。使用更新版本的 LXC 工具，cgroup 是 `lxc/<container_name>`。

对于使用 cgroup 的 Docker 容器，容器名是容器的完整 ID 或长 ID。如果通过 `docker ps` 命令看到的容器为 `ae836c95b4c3`，长 ID 可能是类似 `ae836c95b4c3c9e9179e0e91015512da89fdec91612f63cebae57df9a5444c79` 这样的。可以通过 `docker inspect` 或 `docker ps --no-trunc` 命令查看。

可以通过 `/sys/fs/cgroup/memory/docker/<longid>/` 把所有东西放在一起同时看 Docker 容器的内存指标。
## 2.3 来自 cgroups 的指标：内存，CPU，块 I/O
对于每个子系统（内存、CPU 和块 I/O），存在一个或多个 pseudo 文件并包含统计信息。
#### 内存指标：MEMORY.STAT
内存指标可在“memory” cgroup 中找到。内存控制组添加了一点开销，因为它对主机上的内存使用情况进行了非常细致的计算。因此，许多发行版默认选择不启用它。通常，要启用它，只需添加一些内核命令行参数：`cgroup_enable=memory swapaccount=1`。

指标在 pseudo 文件 `memory.stat` 中。下面是文件内容的大概示例：
```
cache 11492564992
rss 1930993664
mapped_file 306728960
pgpgin 406632648
pgpgout 403355412
swap 0
pgfault 728281223
pgmajfault 1724
inactive_anon 46608384
active_anon 1884520448
inactive_file 7003344896
active_file 4489052160
unevictable 32768
hierarchical_memory_limit 9223372036854775807
hierarchical_memsw_limit 9223372036854775807
total_cache 11492564992
total_rss 1930993664
total_mapped_file 306728960
total_pgpgin 406632648
total_pgpgout 403355412
total_swap 0
total_pgfault 728281223
total_pgmajfault 1724
total_inactive_anon 46608384
total_active_anon 1884520448
total_inactive_file 7003344896
total_active_file 4489052160
total_unevictable 32768
```
前一半（没有 `total_` 前缀）包含 cgroup 中除 sub-cgroups （子 cgroup）之外的进程的指标。后一半（有 `total_` 前缀）也包含 sub-cgroups。

一些指标是“gauges”（量表），或者可以增加或减少的值。例如，swap 是 cgroup 成员使用的交换空间量。其他一些是“counters”（计数器），或值只能增加，因为它们代表特定事件的发生。例如，pgfault 表示创建 cgroup 以来的页面错误数。

指标|	描述
-|-
cache|	此控制组的进程使用的内存，可以与块设备上的块精确关联。在使用“conventional”（常规）I/O（打开，读取，写入系统调用）以及映射文件（使用 mmap）读写磁盘上的文件时，此数量会增加。它也解释了 tmpfs 挂载使用的内存，但原因尚不清楚（It also accounts for the memory used by tmpfs mounts, though the reasons are unclear）。
rss|	不和磁盘上的任何东西关联的内存：堆栈，匿名内存映射
mapped_file|	控制组中的进程映射的内存量。它不会告诉你有多少内存被使用; 它会告诉你它是如何使用的。
pgfault, pgmajfault|	cgroup 的进程分别触发““page fault”（页面错误）和“major fault”（严重错误）的次数。当进程访问不存在或受保护的部分虚拟内存空间时，会发生页面错误。如果进程有 BUG 并尝试访问无效地址（它会发送一个 `SIGSEGV` 信号，通常使用 `Segmentation fault` 消息将其杀死），则前者可能发生。当进程从已被换出的内存区读取或者对应于映射文件时，后者可能发生：在这种情况下，内核从磁盘加载页面，并让 CPU 完成内存访问。当进程写入写时复制内存区域时，也可能发生这种情况：同样，内核会抢占进程，复制内存页面，并在进程自己的页面副本上恢复写入操作。内核实际需要从磁盘读取数据时发生“Major”故障。当它只是复制现有页面或分配空白页面时，它是一个常规（或“次要”）错误。
swap|	当前 cgroup 中的进程使用的 swap 的大小。
active_anon, inactive_anon|	内核已识别的分别处于活动状态和非活动状态的匿名内存大小。“匿名”内存是未链接到磁盘页面的内存。换句话说，这就是上述 rss 计数器的等价物。实际上，rss 计数器的定义是 **active_anon** + **inactive_anon** - **tmpfs**（其中，tmpfs 是由此控制组装载的 tmpfs 文件系统使用的内存量）。“active”和“inactive”之间有什么区别？页面最初是“active”; 内核定期扫描内存，并将某些页面标记为“inactive”。每当再次访问时，他们立即被重新标记为“active”。当内核几乎内存不足，就到了需要将部分内存换出磁盘回收内存的时刻，内核会交换“inactive”页面。
active_file, inactive_file|	高速缓冲存储器，具有与上述匿名存储器相似的 active 和 inactive 状态。确切的公式是 **cache** = **active_file** + **inactive_file** + **tmpfs**。内核用于在 active 和 inactive 集之间移动内存页的规则与用于匿名内存的规则不同，但一般原则相同。当内核需要回收内存时，从该池中回收干净（=未修改）页面会更便宜，因为它可以立即回收（而匿名页面和脏/修改页面需要先写入磁盘）（When the kernel needs to reclaim memory, it is cheaper to reclaim a clean (=non modified) page from this pool, since it can be reclaimed immediately (while anonymous pages and dirty/modified pages need to be written to disk first)）。
unevictable|	无法回收的内存；一般来说，会统计被 mlock “锁定”的内存。通常被加密框架用来确保加密密钥和其他敏感材料永远不会换出到磁盘。
memory_limit, memsw_limit|	这些并不是真正的指标，但是提醒了应用于此 cgroup 的限制。第一个表示该控制组的进程可以使用的最大物理内存量；第二个表示 RAM + swap 的最大数量。

记录页面缓存中的内存非常复杂。如果不同控制组中的两个进程读取同一文件（最终依靠磁盘上的相同块），则相关内存将在控制组之间分配。这很好，但这也意味着当一个 cgroup 被终止时，可能会增加另一个 cgroup 的内存使用率，因为它们不再为这些内存页面分摊成本。
原文：
Accounting for memory in the page cache is very complex. If two processes in different control groups both read the same file (ultimately relying on the same blocks on disk), the corresponding memory charge is split between the control groups. It’s nice, but it also means that when a cgroup is terminated, it could increase the memory usage of another cgroup, because they are not splitting the cost anymore for those memory pages.
## 2.4 CPU 指标：cpuacct.stat
现在我们已经介绍了内存指标，其他一切都比较简单。CPU 指标位于 `cpuacct` 控制器中。

对于每个容器，一个 pseudo 文件 `cpuacct.stat` 包含容器进程累积的 CPU 使用量，并分解为用户和系统时间。区别是：

- `user` 时间是进程直接控制 CPU，执行进程代码的时间。
- `system` 时间是内核代表进程执行系统调用的时间。

这些时间以 1/100 秒的刻度表示，也称为“user jiffies”。每秒有 `USER_HZ` 个 “jiffies”，而在x86系统中，`USER_HZ` 为100。从历史上看，这恰好映射到调度器“ticks”每秒运作的数量，但更高频率调度和 tickless 内核已经使得“ticks”次数不相干。
#### 块 I/O 指标
块 I/O 在 `blkio` 控制器中计算。不同的指标分散在不同的文件中。虽然可以在内核文档的 [blkio-controller](https://www.kernel.org/doc/Documentation/cgroup-v1/blkio-controller.txt) 文件中找到详细的详细信息，但下面是一些最相关的列表：

指标|	描述
-|-
blkio.sectors|	包含由 cgroup 的进程成员逐个设备读取和写入的 512 字节的扇区的个数。读取和写入被合并在一个计数器中。
blkio.io_service_bytes|	指示 cgroup 读写的字节数。它在每个设备上有 4 个计数器，因为每个设备有同步和异步 I/O，读和写操作总共四种情况。
blkio.io_serviced|	I/O 操作执行的此时，和操作的大小无关。在每个设备上也有 4 个计数器。
blkio.io_queued|	指示当前 cgroup 队列的 I/O 操作数量。换句话说，如果 cgroup 没有 I/O 操作，则这个值是 0。但是如果没有 I/O 队列，则并不能表示 cgroup 就是空闲状态（I/O 方面）。它可以在静态设备上进行纯粹的同步读取，因此可以立即处理它们，而无需排队（It could be doing purely synchronous reads on an otherwise quiescent device, which can therefore handle them immediately, without queuing）。此外，尽管找出哪个 cgroup 正在对 I/O 子系统施加压力是有帮助的，但请记住这是相对的。即使进程组没有执行更多的 I/O，其队列大小也会因为其他设备的负载增加而增加（Even if a process group does not perform more I/O, its queue size can increase just because the device load increases because of other devices）。
## 2.5 网络指标
网络指标没有通过控制组直接暴露。对此的解释是：网络接口在网络命名空间上下文中存在。内核可能会计算出一组进程收发的包和字节数，但这些指标并没啥用。你需要每个接口的指标（因为在本地 `lo` 接口上发生的流量并不算真正的数量）。但是同一个 cgroup 中的进程可能属于多个网络命名空间，那些指标很难解释：多个网络名称空间表示多个 `lo` 接口，可能包含多个 `eth0` 接口等；所以这就是为什么没有简单的方法来收集控制组的网络指标。

相反，我们可以从其他来源收集网络指标：
#### IPTABLES
IPtables（或者说，iptables 只是一个接口的 netfilter 框架，the netfilter framework for which iptables is just an interface）可以做一些严肃的统计。

例如，可以设置规则来计算 web 服务器的出站 HTTP 流量：
```
$ iptables -I OUTPUT -p tcp --sport 80
```
没有 `-j` 或 `-g` 标志，所以规则只是对匹配的数据包进行计数并到后面的规则。

之后，可以检查计数器的值：
```
$ iptables -nxvL OUTPUT
```
从技术上讲，`-n` 不是必需的，但它阻止 iptables 进行 DNS 反向查找，在这种情况下这可能没有用处。

计数器包括数据包和字节。如果你想为这样的容器流量设置指标，可以执行一个 `for` 循环，在 `FORWARD` 链中为每个容器 IP 地址添加两个 `iptables` 规则（每个方向一个）。这只会测量通过 NAT 层的流量；还需要添加通过用户级代理的流量。

然后，需要定期检查这些计数器。如果你碰巧使用 `collectd`，那么有一个 [很好的插件](https://collectd.org/wiki/index.php/Table_of_Plugins) 来自动化 iptables 计数器集合。
#### 接口层计数器
由于每个容器都有一个虚拟以太网接口，因此可能需要直接检查该接口的 TX 和 RX 计数器。每个容器都与主机中的虚拟以太网接口关联，其名称类似 `vethKk8Zqi`。找出哪个接口对应于哪个容器是非常困难的。

但是现在，最佳做法是在容器内检查指标。要实现这个目的，可以使用 **ip-netns magic** 在容器的网络命名空间内的主机环境中运行一个可执行文件（you can run an executable from the host environment within the network namespace of a container using ip-netns magic）。

`ip-netns exec` 命令允许你在任何对当前进程可见的网络命名空间中执行任何程序（在主机系统上的）。这意味着你的主机可以进入你的容器的网络命名空间，但是容器无法进入主机或其他容器的网络命名空间。但是容器可以和子容器交互。

命令的完整格式是：
```
$ ip netns exec <nsname> <command...>
```
例如：
```
$ ip netns exec mycontainer netstat -i
```
`ip netns` 通过使用命名空间的 pseudo 伪文件找到“mycontainer”容器。每个进程属于一个网络命名空间，一个 PID 命名空间，一个 `mnt` 命名空间等，并且这些命名空间在 `/proc/<pid>/ns/` 下实现。例如，PID 42 的网络命名空间由伪文件 `/proc/42/ns/net` 实现。

当你运行 `ip netns exec mycontainer ...` 时，它会希望 `/var/run/netns/mycontainer`  成为那些伪文件之一（接受符号链接）。

换句话说，要在容器的网络命名空间中执行命令，我们需要：

- 找出我们想要调查的容器内的任何进程的 PID
- 创建 `/var/run/netns/<somename>` 到 `/proc/<thepid>/ns/net` 的符号链接
- 执行 `ip netns exec <somename> ....`

查看上面部分的枚举 cgroup 以了解如何查找要测量其网络使用情况的容器内进程所属的 cgroup。从那里，你可以检查伪文件命名的任务，其中包含 cgroup 中的所有PID（因此，在容器中）。选择任何一个 PID。
原文：
Review Enumerate Cgroups for how to find the cgroup of an in-container process whose network usage you want to measure. From there, you can examine the pseudo-file named tasks, which contains all the PIDs in the cgroup (and thus, in the container). Pick any one of the PIDs.

把所有东西放在一起，如果一个容器的“短ID”保存在环境变量 `$CID` 中，那么你可以这样做：
```
$ TASKS=/sys/fs/cgroup/devices/docker/$CID*/tasks
$ PID=$(head -n 1 $TASKS)
$ mkdir -p /var/run/netns
$ ln -sf /proc/$PID/ns/net /var/run/netns/$CID
$ ip netns exec $CID netstat -i
```
# 3. 高性能指标收集的建议
每次想要更新指标时运行一个新进程都相当昂贵。如果想要以高分辨率和/或通过大量容器收集度量标准（将单个主机上的容器想成1000个），则不需要每次都分叉一个新进程。

以下是如何从单个进程收集指标。需要使用 C 语言编写指标收集器（或任何允许执行低级别系统调用的语言）。需要使用一个特殊的系统调用 `setns()`，它允许当前进程进入任意的命名空间。然而，它需要一个打开的到命名空间伪文件（记住：这是 `/proc/<pid>/ns/net` 中的伪文件）的文件描述符。

然而，有一个问题：你不能一直保持这个文件描述符打开。如果这样，当控制组的最后一个进程退出时，名称空间不会被销毁，并且其网络资源（如容器的虚拟接口）永远占用（或直到你关闭该文件描述符）。

正确的做法是跟踪每个容器的第一个 PID，并且每次都重新打开命名空间伪文件。
# 4. 在容器退出时收集指标
有时候，你不关心实时指标收集，但是当一个容器退出时，你想知道它使用了多少CPU，内存等。

Docker 使得这很困难，因为它依赖于 `lxc-start`，它在它自己之后仔细清理（carefully cleans up after itself）。 定期收集指标通常更容易，这就是 `collectd` LXC插件的工作方式。

但是，如果你仍想在容器停止时收集统计信息，请执行以下操作：

对于每个容器，启动一个收集进程，并通过将其 PID 写入 cgroup 的任务文件，将其移至要监控的控制组。收集进程应定期重新读取任务文件以检查它是否是控制组的最后一个进程。（如果还想按前一节中的说明收集网络统计信息，则还应该将进程移至适当的网络命名空间。）

当容器退出时，`lxc-start` 尝试删除控制组。它失败了，因为控制组仍在使用中；但没关系。现在你的进程应该检测到它是该组中剩下的唯一一个。现在是收集你需要的所有指标的适当时机！

最后，你的进程应该移回到根控制组，并删除容器控制组。要删除控制组，只需 `rmdir` 其目录。由于 `rmdir` 仍然包含文件，因此它是违反直觉的；但请记住这是一个伪文件系统，所以通常的规则不适用。清理完成后，收集过程可以安全地退出。
