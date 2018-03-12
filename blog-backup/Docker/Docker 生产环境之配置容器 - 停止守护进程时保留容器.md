[原文地址](https://docs.docker.com/config/containers/live-restore/)

默认情况下，当 Docker 守护进程终止时，会关闭正在运行的容器。从 Docker Engine 1.12 开始，可以配置守护进程，使容器在守护进程不可用时保持运行。这个功能被称为 live restore。live restore 选项有助于减少由于守护进程崩溃，计划停机或升级导致的容器停机时间。

>注意：Windows 容器不支持 live restore，但对于 Docker for Windows 上运行的 Linux 容器也是适用的。
# 1. 开启 live restore
有两种方式可以开启 live restore 设置，使容器在守护进程不可用时保持活动状态。**下面两种方式二选一**：

- 在守护进程配置文件中设置。在 Linux 上默认是 `/etc/docker/daemon.json`， 在 Docker for Mac 或 Docker for Windows选择任务栏的 Docker 图标并点击 Preferences -> Daemon -> Advanced。
  - 使用下面的 JSON 来开启 live-restore：
	```
	{
	   "live-restore": true
	}
	```
  - 重启 Docker 守护进程。在 Linux 上 可以通过重新加载 Docker 守护进程来避免重启（同时避免容器停止）。如果你使用 `systemd`，使用命令 `systemctl reload docker`。否则，向 `dockerd` 进程发送 `SIGHUP` 信号。
- 也可以在手动启动 `dockerd` 进程时指定 `--live-restore` 参数。不建议使用此方法，因为它不会在启动 Docker 进程时设置 `systemd` 或其他进程管理器使用的环境。这可能会导致意外的行为。
# 2. 升级守护进程时的 live restore
live restore 功能支持将容器恢复到守护进程以便从一个次要版本升级到下一个次要版本，例如从 Docker 1.12.1 升级到 1.12.2 时。

如果在升级过程中跳过版本，守护进程可能无法恢复其与容器的连接。如果守护程序无法恢复连接，则无法管理正在运行的容器，必须手动停止它们。
# 3. 重启时的 live restore
只有守护进程选项（例如网桥 IP 地址和图形驱动程序）未发生更改时 live restore 选项才可以用于还原容器。如果任何这些守护进程级配置选项已更改，则 live restore 可能不起作用，可能需要手动停止容器。
# 4. live restore 对运行容器的影响
如果守护进程停止很长时间，正在运行的容器可能会填满守护程序通常读取的 FIFO 日志。日志填满后会阻止容器记录更多数据。默认缓冲区大小为 64K。如果缓冲区填满，必须重新启动 Docker 守护程序来刷新它们。

在 Linux 上，可以通过更改 `/proc/sys/fs/pipe-max-size` 来修改内核的缓冲区大小。不能修改 Docker for Mac 或 Docker for Windows 的缓冲区大小。
# 5. Live restore 和 swarm 模式
live restore 选项仅适用于独立容器，不适用于 swarm 服务。swarm 服务由 swarm manager 管理。如果 swarm manager 不可用，swarm 服务将继续在 worker 节点上运行但无法管理，只有有足够多的可用 swarm manager 时才能维护（If swarm managers are not available, swarm services continue to run on worker nodes but cannot be managed until enough swarm managers are available to maintain a quorum.）。
