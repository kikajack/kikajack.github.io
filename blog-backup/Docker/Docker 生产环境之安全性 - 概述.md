[原文地址](https://docs.docker.com/engine/security/security/)

在审查 Docker 安全性时，需要考虑四个主要方面：

- 内核固有的安全性及其支持的 namespace 及 cgroup
- Docker 守护进程本身的攻击面（attack surface）
- 容器默认的或用户自定义的配置中的漏洞
- 内核的“强化”安全功能以及它们如何与容器交互
# 1. 内核命名空间 namespace
Docker 容器跟 LXC 容器类似，并且它们具有相似的安全特性。通过 `docker run` 命令启动容器时，Docker 会在背后为这个容器创建一组命名空间和控制组。

**命名空间提供了第一个也是最直接的隔离形式**：在容器中运行的进程看不到并且几乎不会影响在另一个容器或主机系统中运行的进程。

**每个容器具有独立的网络堆栈**，这意味着容器不会获得对另一个容器的套接字或接口的特权访问。当然，如果主机系统进行相应设置，容器可以通过各自的网络接口相互交互 - 就像他们可以与外部主机进行交互一样。当你为容器指定公共端口或使用 [link](https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/) 时，容器之间允许 IP 流量。它们可以互相 ping 通，发送/接收 UDP 数据包，并建立 TCP 连接，当然如果需要的话可以限制它们之间的连接。从网络体系结构的角度来看，给定 Docker 主机上的所有容器都位于网桥接口上。这意味着它们就像通过普通以太网交换机连接的物理机器一样，完全类似。

用于内核命名空间和私有网络的代码有多成熟呢？内核命名空间在 [内核的  2.6.15 和 2.6.26 版本之间引入](http://man7.org/linux/man-pages/man7/namespaces.7.html)。这意味着自从 2008 年 7 月（2.6.26 版本发布时间）开始，命名空间代码已经在大量的生产系统上被执行和审查。还有更多：命名空间代码的设计和灵感甚至更老。命名空间实际上是为了重新实现 [OpenVZ](http://en.wikipedia.org/wiki/OpenVZ) 的功能，以便它们可以在主流内核中合并。OpenVZ 最初于2005年发布，因此设计和实现都相当成熟。
# 2. 控制组 control groups
控制组是 Linux 容器的另一个关键组件。它们实现了资源认定和限制（They implement resource accounting and limiting）。控制组提供许多有用指标，但是它们还有助于确保每个容器公平的获得内存，CPU，磁盘 I/O；更重要的是，单个容器是无法耗尽这些资源中的某一种，从而避免了系统性能下降。

因此，尽管它们不能阻止一个容器访问或影响另一个容器的数据和进程，但它们对抵御一些拒绝服务攻击至关重要。它们对于多租户平台尤其重要，例如公共和私有 PaaS，即使在某些应用程序开始出现故障时也能保证一致的正常运行时间（和性能）（to guarantee a consistent uptime (and performance) even when some applications start to misbehave.）。

控制组也有一段时间了：代码是在2006年开始的，最初被合并到内核 2.6.24 中。
# 3. Docker 守护进程攻击面
使用 Docker 运行容器（和应用程序）意味着运行 Docker 守护进程。这个守护进程当前需要 root 权限，因此你应该知道一些重要的细节。

首先，**只允许可信用户控制你的 Docker 守护进程**。这是因为一些强大的 Docker 功能的可能带来严重后果。具体来说，Docker 允许你在 Docker 主机和访客容器之间共享一个目录；它允许你在不限制容器访问权限的情况下这样做。这意味着可以启动一个容器，其中 `/host` 目录是主机上的 `/` 目录；容器可以不受任何限制地改变你的主机文件系统。这与虚拟化系统如何允许文件系统资源共享类似。没有什么能够阻止你与虚拟机共享你的根文件系统（甚至你的根块设备）。

这具有很强的安全意义：例如，如果你通过 Web 服务器指示 Docker 通过 API 调配容器，则应该比平时更加仔细地进行参数检查，以确保恶意用户无法传递伪造的参数，从而导致 Docker 创建任意容器。

考虑到这个问题，REST API 终端（被 Docker CLI 用来跟 Docker 守护进程通信）从 Docker 0.5.2 起发生变化，使用 UNIX 套接字替代绑定到 127.0.0.1 的 TCP 套接字（如果你直接在 VM 之外的本地主机上运行 Docker，后者可能会有跨站请求伪造攻击）。然后你就可以使用传统的 UNIX 权限检查来限制对控制套接字的访问。

只有愿意，还可以通过 HTTP 公开 REST API。但是，如果这样做，请注意上述安全隐患。确保它只能从受信任的网络或 VPN 访问，或者使用 `stunnel` 和客户端 SSL 证书等机制进行保护。还可以使用 [HTTPS 和证书 ](https://docs.docker.com/engine/security/https/) 来保护 API 端点。

守护进程也可能容易受到其他输入的影响，例如通过 `docker load` 从磁盘加载镜像或通过 `docker pull` 从网络加载镜像。从 Docker 1.3.2 开始，镜像现在在 Linux/Unix 平台的 chrooted 子进程中提取，这是实现特权分离的工作的第一步。从 Docker 1.10.0 开始，所有镜像都通过其内容的加密校验和（cryptographic checksum）进行存储和访问，从而限制了攻击者与现有镜像发生冲突的可能性。

最后，如果你在服务器上运行 Docker，建议在服务器上专门运行 Docker，并将所有的其他服务移动到由 Docker 控制的容器内。当然，保留管理工具（至少是 SSH）以及现有的监控/监督流程（如 NRPE 和 collectd）是没问题的。
# 4. Linux 内核的 capabilities
默认情况下，Docker 使用一组受限制的 capabilities 启动容器。那是什么意思？

capabilities 将“root/non-root”二分法转变为一个细粒度的访问控制系统。只需要在低于 1024 的端口上绑定的进程（如 Web 服务器）不需要以 root 身份运行：它们可以被赋予 `net_bind_service` capabilities。对于几乎所有需要 root 权限的特定领域，还有许多其他 capabilities（there are many other capabilities, for almost all the specific areas where root privileges are usually needed.）

这对于容器安全意义重大。让我们看看为什么！

通常服务器会以 root 身份运行几个进程，包括 SSH 守护进程，cron 守护进程，logging 守护进程，内核模块，网络配置工具等等。容器是不同的，因为几乎所有这些任务都是由容器周围的基础设施处理的：

- SSH 访问通常由运行在 Docker 主机上的一个单独的服务管理
- cron，如果需要的话，应该以用户进程运行，专门针对需要其调度服务的应用程序定制，而不是作为平台范围的设施
- 日志管理通常交给 Docker 或第三方服务，例如 Loggly 或 Splunk
- 硬件管理是无关紧要的，这意味着你永远不需要在容器中运行 `udevd` 或等价的守护进程
- 网络管理发生在容器之外，尽可能地强化关注点分离，这意味着容器应该永远不需要执行 `ifconfig`，`route` 或 `ip` 命令（除非容器专门设计为像路由器或防火墙一样工作）。

这意味着大多数情况下，容器并不需要真正的 root 权限。因此，容器可以通过一组受限制的 capabilities 来运行，此时容器中的 root 比真正的 root 权限少得多。例如，下面的几个是可能的情况：

- 禁止所有的挂载 mount 操作
- 禁止访问原始套接字（以防止数据包欺骗）
- 禁止某些文件系统相关操作，例如创建新的设备节点，改变文件所有者或改变属性（包括只读标志）
- 禁止加载模块
- 其他

这意味着，即使入侵者设法在容器内获取了 root 权限，要难以做到严重破坏或获得主机 root 权限。

这不会影响常规的 Web 应用程序，但会大大减少恶意用户的攻击媒介。默认情况下，Docker 会删除除了 [必须的 capabilities](https://github.com/moby/moby/blob/master/oci/defaults.go#L14-L30) 外的所有 capabilities，即白名单而不是黑名单方法。可以在 [Linux 手册页](http://man7.org/linux/man-pages/man7/capabilities.7.html) 中看到完整的可用 capabilities 列表。

运行 Docker 容器的一个主要风险是给容器默认的一组 capabilities 和挂载可能会提供不完全的隔离或独立性，或者与内核漏洞结合使用（when used in combination with kernel vulnerabilities）。

Docker 支持添加和删除 capabilities，允许使用非默认配置文件。删除 capabilities 可能会使 Docker 更安全，增加 capabilities 会降低 Docker 的安全性。对于用户来说，最好的做法是删除所有不需要的 capabilities。
# 5. 其他的内核安全特性
capabilities 只是现代 Linux 内核提供的众多安全功能之一。还可以在 Docker 中使用现有的知名系统，如 TOMOYO，AppArmor，SELinux，GRSEC 等。

虽然现在 Docker 仅支持 capabilities，但不会干扰其他系统。这意味着有很多不同的方法来加固 Docker 主机。这里有一些例子。

- 可以使用 GRSEC 和 PAX 运行内核。这在编译时和运行时都增加了许多安全检查；它也通过地址随机化等技术击败了许多漏洞。它不需要特定于 Docker 的配置，因为这些安全特性适用于系统范围，独立于容器。
- 如果你的发行版带有用于 Docker 容器的安全模型模板，可以直接使用。例如，我们发布了一个可与 AppArmor 配合使用的模板，而 Red Hat 提供了适用于 Docker 的 SELinux 策略。这些模板提供了一个额外的安全网络（尽管它与 capabilities 重叠）。
- 可以使用你喜欢的访问控制机制来定义自己的策略。

就像你可以使用第三方工具来扩充 Docker 容器（包括特殊网络拓扑或共享文件系统）一样，存在用于强化 Docker 容器而无需修改 Docker 本身的工具。

对于 Docker 1.10 的用户，Docker 守护进程直接支持命名空间。这个特性允许容器中的 root 用户映射到容器外部的非 uid-0 用户，这有助于减轻容器突破的风险。该工具可用，但默认情况下不启用。

有关此功能的更多信息，请参阅命令行参考中的 [守护程序命令](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-user-namespace-options)。有关 Docker 中用户命名空间实现的其他信息可以在 [此博客文章](https://integratedcode.us/2015/10/13/user-namespaces-have-arrived-in-docker/) 中找到。
# 6. 结论
Docker 容器默认情况下十分安全，尤其是你在容器中通过非特权用户运行进程时。

可以通过开启 AppArmor、SELinux、GRSEC 或其他强化系统来添加额外的安全层。
