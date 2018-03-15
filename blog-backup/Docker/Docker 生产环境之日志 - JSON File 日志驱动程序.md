[原文地址](https://docs.docker.com/config/containers/logging/json-file/)

默认情况下，Docker 捕获所有容器的标准输出（和标准错误），并将其写入使用 JSON 格式的文件。这个 JSON 格式用每个行的来源（stdout 或 stderr）及其时间戳注释。每个日志文件都包含只有一个容器的信息。
# 1. 使用
要使用 `json-file` 驱动程序作为默认的日志驱动程序，需要设置 `daemor.json` 文件中的 `log-driver` 和 `log-opt` 关键字为合适的值。这个文件通常在 `/etc/docker/`（Linux）或 `C:\ProgramData\docker\config\daemon.json` （Windows）。更多信息参考 [ `daemor.json` ](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)。

下面示例设置日志驱动程序为 `json-file` 并设置了 `max-size` 选项。
```
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m"
  }
}
```
重新启动 Docker 以使更改对新创建的容器生效。现有容器即使重启 Docker 也不会使用新的日志记录配置。

可以在 `docker container create` 或 `docker run` 命令中通过 `--log-driver` 标志为某个特定容器设置日志驱动程序：
```
$ docker run \
      --log-driver json-file --log-opt max-size=10m \
      alpine echo hello world
```
## 1.1 选项
`json-file`日志驱动程序支持下面的日志选项：

选项	|描述|	示例值
-|-|-
`max-size`|	滚动前日志的最大大小。一个正整数加上一个代表测量单位（k，m 或 g）的修饰符。默认为 -1（无限制）。	|`--log-opt max-size=10m`
`max-file`|	可以存在的最大日志文件数量。如果滚动日志会创建多余文件，则会删除最旧的文件。只有在设置了 `max-size` 时才有效。一个正整数。 默认为1。	|`--log-opt max-file=3`
`labels`|	在启动 Docker 守护进程时适用。守护进程接受的日志相关标签的逗号分隔列表。用于高级日志标记选项。	|`--log-opt labels=production_status,geo`
`env`|	在启动 Docker 守护进程时适用。此守护程序接受的与日志相关的环境变量的逗号分隔列表。用于高级日志标记选项。	|`--log-opt env=os,customer`
`env-regex`|	与 `env` 类似且兼容。一个正则表达式来匹配与日志相关的环境变量。用于高级日志标记选项。	|`--log-opt env-regex=^(os|customer)`
## 1.2 示例
这个例子启动的 alpine 容器最多有 3 个日志文件且每个日志文件不大于 10MB。
```
$ docker run -it --log-opt max-size=10m --log-opt max-file=3 alpine ash
```
