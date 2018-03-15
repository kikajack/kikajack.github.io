[原文地址](https://docs.docker.com/config/containers/logging/configure/)

Docker 提供多种日志驱动程序帮助你从运行中的容器和服务获取信息。

每个 Docker 守护进程都有一个默认的日志驱动程序，如果你没有将其配置为其他日志驱动程序，则每一个容器都会使用这个默认设置。

除了使用 Docker 自带的日志驱动程序外，你还可以实现和使用日志驱动程序插件（需要 Docker 17.05 或更高版本）。
# 1. 配置默认的日志驱动程序
要配置 Docker 守护进程默认使用指定的日志驱动程序，将 `daemon.json` 文件（Linux 中一般位于 `/etc/docker/`，Windows 中一般位于 `C:\ProgramData\docker\config\`）中的 `log-driver` 值设为日志驱动程序的名字即可。默认的日志驱动程序 `json-file`。下面例子将其设置为 `syslog`：
```
{
  "log-driver": "syslog"
}
```
如果日志驱动程序有可配置的选项，可以在 `daemon.json` 文件的关键字 `log-opts` 中以 JSON 格式设置。下面示例为 `json-file` 日志驱动程序设置了两个可配置选项：
```
{
  "log-driver": "json-file",
  "log-opts": {
    "labels": "production_status",
    "env": "os,customer"
  }
}
```
如果你没有指定日志驱动程序，默认就是 `json-file`。因此，`docker inspect <CONTAINER>` 之类的命令的默认输出就是 JSON。

要找出当前 Docker 守护进程的默认日志驱动程序，运行 `docker info` 命令并在输出中找 `Logging Driver`。下面命令可以在 Linux、macOS 或 Windows 上使用：
```
$ docker info | grep 'Logging Driver'

Logging Driver: json-file
```
# 2. 配置一个容器的日志驱动程序
在启动容器时，可以通过 `--log-driver` 标志将其配置为使用与 Docker 守护进程不同的日志驱动程序。如果日志驱动程序有可配置的选项，可以通过一个多多个 `--log-opt <NAME>=<VALUE>` 来设置。即使容器使用的是默认的日志驱动程序，也可以使用不同的配置选项。

下面的例子启动了一个使用 `none` 日志驱动程序的 Alpine 容器。
```
$ docker run -it --log-driver none alpine ash
```
要找出一个运行中的容器当前使用的日志驱动程序，如果守护进程使用的是 `josn-file` 日志驱动程序，运行下面的 `docker inspect` 命令，将 `<CONTAINER>` 替换为容器名或 ID：
```
$ docker inspect -f '{{.HostConfig.LogConfig.Type}}' <CONTAINER>

json-file
```
# 3. 配置从容器到日志驱动程序的日志消息的传送模式
Docker 提供两种模式来将日志消息从容器发送到日志驱动程序：

- （默认）直接阻塞式的从容器发送到驱动程序
- 非阻塞发送，将日志消息存储在中间每个容器的环形缓冲区中供驱动程序使用（non-blocking delivery that stores log messages in an intermediate per-container ring buffer for consumption by driver）

非阻塞消息传送模式可防止应用程序因记录的压力（logging back pressure）而被阻塞。当 STDERR 或 STDOUT 流阻塞时，应用程序可能会以意想不到的方式失败。

>警告：当缓冲区已满且新消息排入队列时，内存中最早的消息将被丢弃。丢弃消息通常首选阻止应用程序的日志写入过程。（Dropping messages is often preferred to blocking the log-writing process of an application.）

`mode` 这个日志选项用于控制使用阻塞还是非阻塞哪个消息发送方式。

`max-buffer-size` 这个日志选项用于控制非阻塞方式下用作中间消息存储的环形缓冲区大小，默认是 1MB。

下面示例启动了日志输出为非阻塞模式且有 4MB 缓存的 Alpine 容器。
```
$ docker run -it --log-opt mode=non-blocking --log-opt max-buffer-size=4m alpine ping 127.0.0.1
```
## 3.1 为日志驱动程序使用环境变量或标签
部分日志驱动程序会将容器的 `--env|-e` 或 `--label` 标志值添加到容器的日志中。这个例子启动了一个使用 Docker 守护进程默认日志驱动程序（假设是 `json-file`）的容器，但是设置了环境变量 `os=ubuntu`。
```
$ docker run -dit --label production_status=testing -e os=ubuntu alpine sh
```
如果日志驱动程序支持，这会添加额外的字段到日志输出中。下面是 `json-file` 日志驱动程序的输出：
```
"attrs":{"production_status":"testing","os":"ubuntu"}
```
# 4. 支持的日志驱动程序
支持以下日志驱动程序。参考每个驱动程序的文档来了解相关配置选项。如果你使用了 [日志驱动程序插件](https://docs.docker.com/engine/admin/logging/plugins/)，会有更多的选项。

驱动程序|	描述
-|-
none|	容器没有日志可用，`docker logs` 什么都不返回
json-file|	日志格式化为 JSON。这是 Docker 默认的日志驱动程序。
syslog|	将日志消息写入 `syslog` 工具。`syslog` 守护程序必须在主机上运行。
journald|	将日志消息写入 `journald`。`journald` 守护程序必须在主机上运行。
gelf|	将日志消息写入 Graylog Extended Log Format (GELF) 终端，例如 Graylog 或 Logstash。
fluentd|	将日志消息写入 `fluentd`（forward input）。`fluentd` 守护程序必须在主机上运行。
awslogs|	将日志消息写入 Amazon CloudWatch Logs。
splunk	|Writes log messages to splunk using the HTTP Event Collector.
etwlogs|	将日志消息写为 Windows 的 Event Tracing 事件。仅在Windows平台上可用。
gcplogs|	将日志消息写入 Google Cloud Platform (GCP) Logging。
logentries|	将日志消息写入 Rapid7 Logentries。
# 5. 日志驱动程序的限制
在使用 `json-file` 和 `journald` 之外的日志驱动程序时 `docker logs` 命令不可用。
