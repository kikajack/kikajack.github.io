[原文地址](https://docs.docker.com/config/daemon/)

在安装并成功启动 Docker 后，`dockerd` 守护进程就会使用默认配置运行。本主题讨论如何自定义配置，手动启动守护进程，以及在发生问题时的问题定位和调试守护进程。
#1. 用操作系统工具开启守护进程
启动 Docker 的命令根据操作系统的不同而不同。在 [安装 Docker](https://docs.docker.com/install/) 部分查找对应资料。要将 Docker 配置为开机自动启动，参考 [Configure Docker to start on boot](https://docs.docker.com/install/linux/linux-postinstall/#configure-docker-to-start-on-boot)。
#2. 手动启动守护进程
通常用操作系统的工具启动 Docker。对于调试目的，可以通过 `dockerd` 命令手动启动 Docker。根据操作系统的不同可能需要 `sudo`。当你通过这种方式启动 Docker 时，它会在前台运行并将日志直接发送到终端窗口。
```
$ dockerd

INFO[0000] +job init_networkdriver()
INFO[0000] +job serveapi(unix:///var/run/docker.sock)
INFO[0000] Listening for HTTP on unix (/var/run/docker.sock)
...
...
```
在终端窗口中通过 Ctrl+C 停止手动启动的 Docker。
#3. 配置 Docker 守护进程
守护进程包含许多配置选项，可以在手动启动 Docker 时将其作为标志传递，或者在 `daemon.json` 配置文件中进行设置。推荐使用第二种方法，因为重新启动 Docker 时，这些配置更改仍然存在。

完整的配置选项列表，参考 [dockerd](https://docs.docker.com/engine/reference/commandline/dockerd/)。

下面是使用一些配置选项手动启动 Docker 守护程序的示例：
```
$ dockerd -D --tls=true --tlscert=/var/docker/server.pem --tlskey=/var/docker/serverkey.pem -H tcp://192.168.59.3:2376
```
这个命令开启了调试（`-D`），TLS（`-tls`），指定了服务器证书和密钥（`--tlscert` 和 `--tlskey`），并指定了守护进程监听连接的网络接口（`-H`）。

更好的方法是将这些选项放入 `daemon.json` 文件并重新启动 Docker。此方法适用于每个 Docker 平台。以下 `daemon.json` 示例将所有相同的选项设置为上述命令：
```
{
  "debug": true,
  "tls": true,
  "tlscert": "/var/docker/server.pem",
  "tlskey": "/var/docker/serverkey.pem",
  "hosts": ["tcp://192.168.59.3:2376"]
}
```
Docker 文档中会讨论许多特定的配置选项。可以参考的地方有：

- [Automatically start containers](https://docs.docker.com/engine/admin/host_integration/)
- [Limit a container’s resources](https://docs.docker.com/engine/admin/resource_constraints/)
- [Configure storage drivers](https://docs.docker.com/engine/userguide/storagedriver/)
- [Container security](https://docs.docker.com/engine/security/)
##3.1 解决 `daemon.json` 和启动脚本之间的冲突
如果同时使用 `daemon.json` 文件，并手工传入选项到 `dockerd` 命令或使用启动脚本，这些选项会冲突，Docker 会启动失败并报错：
```
unable to configure the Docker daemon with file /etc/docker/daemon.json:
the following directives are specified both as a flag and in the configuration
file: hosts: (from flag: [unix:///var/run/docker.sock], from file: [tcp://127.0.0.1:2376])
```
如果你看到类似这样的错误，并且你在启动守护进程的时候使用了选项，那么可能需要调整你的选项或 `daemon.json` 文件来解决冲突。

如果你使用操作系统的 init 脚本启动 Docker，则可能需要以特定于操作系统的方式覆盖这些脚本中的默认值。
###在 `DAEMON.JSON` 中通过 SYSTEMD 使用宿主机的密钥
配置冲突难以解决的一个值得注意的例子是，你希望从默认值指定不同的守护程序地址。默认情况下，Docker 在套接字上侦听。在使用` systemd` 的 Debian 和 Ubuntu 系统上，这意味着在启动 `dockerd` 时始终使用主机标志 `-H`。如果在 `daemon.json` 中指定了 `hosts` 条目，这会导致配置冲突，并且 Docker 无法启动。

要解决这个问题，创建新文件 `/etc/systemd/system/docker.service.d/docker.conf` 并填入下面的内容，并在启动守护进程时不使用默认的 `-H` 参数。
```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
```
在其他时候，可能需要使用 Docker 配置 systemd，例如 [配置 HTTP 或 HTTPS 代理](https://docs.docker.com/engine/admin/systemd/#httphttps-proxy)。

>注意：如果重写此选项，然后在 `daemon.json` 中未指定 `hosts` 条目或手动启动 Docker 时指定 `-H` 标志，则 Docker 无法启动。

在尝试启动 Docker 之前运行 `sudo systemctl daemon-reload`。如果 Docker 成功启动，它会开始监听`daemon.json` 中的 `hosts` 条目指定的 IP 地址，而不再是套接字。

>重要提示：Docker for Windows 或 Docker for Mac 不支持在 `daemon.json` 中进行设置。
#4. 排除守护进程故障
可以在守护进程上启用调试，以了解守护进程的运行时活动并帮助进行故障排除。如果守护进程完全没有响应，也可以通过将 `SIGUSR` 信号发送到 Docker 守护进程来强制将所有线程的 [完整堆栈跟踪](https://docs.docker.com/config/daemon/#force-a-full-stack-trace-to-be-logged) 添加到守护进程日志中。
##4.1 内存不足异常（OOME）
如果你的容器尝试使用比系统可用内存更多的内存，则可能会遇到内存不足异常（OOME），并且容器或 Docker 守护程序可能会被内核 OOM killer 所杀。要防止发生这种情况，请确保你的应用程序在具有足够内存的主机上运行，并且请参阅了解 [耗尽内存的风险](https://docs.docker.com/engine/admin/resource_constraints/#understand-the-risks-of-running-out-of-memory)。
##4.2 读取日志
守护进程日志可以帮助你诊断问题。根据操作系统配置和使用的日志记录子系统，日志可以保存在几个位置之一中：

|操作系统|  位置|
|-|-|
|RHEL, Oracle Linux|  `/var/log/messages`
|Debian |`/var/log/daemon.log`
|Ubuntu 16.04+, CentOS| 使用命令 `journalctl -u docker.service`
|Ubuntu 14.10-  |`/var/log/upstart/docker.log`
|macOS (Docker 18.01+)  |`~/Library/Containers/com.docker.docker/Data/vms/0/console-ring`
|macOS (Docker <18.01)  |`~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/console-ring`
|Windows  |`AppData\Local`
##4.3 开启调试
有两种方式开启调试。推荐的方法是在 `daemon.json` 文件中将 `debug` 关键字设置为 true。此方法适用于每个 Docker 平台。
####1. 编辑 `daemon.json` 文件，通常位于 `/etc/docker/`
如果文件不存在，需要创建这个文件。在 macOS 或 Windows 上，不需要直接编辑这个文件，相反，去 **Preferences / Daemon / Advanced**。
####2. 在文件中增加内容
如果是新创建的文件或文件为空，直接放入下面的内容：
```
{
  "debug": true
}
```
如果文件已经包含了 JSON 数据，只需要添加 ` "debug": true`，并注意用逗号分隔。还需要验证 `log-level` 关键字是否设置，它只能设置为 `info` 或 `debug`（其所有可选值是 `debug`、`info`、`warn`、`error`、`fatal`）。 
####3. 发送 HUP 信号到守护进程，使其重新加载配置
Linux 主机上的命令如下：
```
$ sudo kill -SIGHUP $(pidof dockerd)
```
Windows 主机上重新启动 Docker。

也可以停止 Docker 守护进程并使用调试标志 `-D` 手动重新启动它，而不是遵循此过程。但是，这可能会导致 Docker 在与主机启动脚本创建的环境不同的环境中重新启动，这可能会使调试更加困难。
##4.4 强制将堆栈跟踪记入日志
如果守护进程没有响应，可以通过向守护进程发送一个 `SIGUSR1` 信号来强制将堆栈跟踪记入日志。

- Linux：
```
$ sudo kill -SIGUSR1 $(pidof dockerd)
```
- Windows Server：
下载 [docker-signal](https://github.com/jhowardmsft/docker-signal)。
通过 `--pid=<PID of daemon>` 标志运行可执行文件。

这会强制记录堆栈跟踪，但不会停止守护进程。守护进程日志显示堆栈跟踪或包含堆栈跟踪的文件的路径（如果它已记录到文件中）。

守护进程在处理完 `SIGUSR1` 信号并将堆栈跟踪转储到日志后继续运行。堆栈跟踪可用于确定守护进程内所有 goroutine 和线程的状态。
##4.5 查看堆栈跟踪
可以通过下面的方式查看 Docker 守护进程的日志：

- 在 Linux 系统上运行 `journalctl -u docker.service` 来使用 `systemctl`
- 在老旧的 Linux 系统上 `/var/log/messages`, `/var/log/daemon.log`, or `/var/log/docker.log`
- 在 Windows Server 版 Docker EE 上运行 `Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time | Export-CSV ~/last5minutes.CSV`

>注意：无法在 Docker for Mac 或 Docker for Windows 上手动生成堆栈跟踪。但是，如果遇到问题，可以单击 Docker 任务栏图标并选择诊断和反馈来向 Docker 发送信息。

查 看Docker 日志中的消息，如下所示：
```
...goroutine stacks written to /var/run/docker/goroutine-stacks-2017-06-02T193336z.log
...daemon datastructure dump written to /var/run/docker/daemon-data-2017-06-02T193336z.log
```
Docker 保存这些堆栈跟踪（stack trace）和转储（dump）的位置取决于操作系统和配置。有时可以直接从堆栈跟踪和转储中获取有用的诊断信息。否则，可以将此信息提供给 Docker 以帮助诊断问题。
#5. 检查 Docker 是否在运行
和操作系统无关的检查 Docker 是否运行的方式是直接问 Docker，使用 `docker info` 命令。

可以使用操作系统的工具，例如 `sudo systemctl is-active docker` 或 `sudo status docker` 或 `sudo service docker status`。

最后，可以使用 `ps` 或 `top` 之类的命令在进程列表中检查 `dockerd` 进程。