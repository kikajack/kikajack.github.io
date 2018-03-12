[原文地址](https://docs.docker.com/config/containers/start-containers-automatically/)

Docker 提供了重启策略，以控制容器在退出时是否自动启动，或在 Docker 重新启动时自动启动。重启策略可确保链接的容器以正确的顺序启动。Docker 建议使用重启策略，并避免使用流程管理器启动容器。

重启策略跟 `dockerd` 命令的 `--live-restore` 标志不同。使用 `--live-restore` 标志使得在 Docker 升级过程中容器可以保持运行，虽然网络和用户输入都中断了。
#1. 使用重启策略
要为容器配置重启策略，使用 `docker run` 命令的时候添加 `--restart` 标志。`--restart` 标志的值可以是下面几个：
|标志|  描述|
|-|
|`no`|  不自动重启容器（默认值）
|`on-failure`|  如果容器由于错误而退出，则将其重新启动，非零退出代码表示错误
|`unless-stopped`|  重新启动容器，除非明确停止容器或者 Docker 被停止或重新启动
|`always`|  只要容器停止了，就重新启动

下面例子的 Redis 容器会一直重启，除非明确停止这个容器或 Docker 重启了。
```
$ docker run -dit --restart unless-stopped redis
```
##1.1 重启策略详情
使用重启策略时，记住以下几点：

- 重启策略只在容器启动成功后才生效。这种情况下，成功启动的意思是容器运行 10 秒以上，并且 Docker 已经开始监控它。这可以防止根本不启动的容器进入重启循环。
- 如果你手动停止一个容器，它的重启策略会被忽略，直到 Docker 守护进程重启或容器手动重启。这是防止重启循环的另一个尝试。
- 重启策略只作用于容器。swarm 服务的重启策略配置方式不同。查看 [与服务重启相关的标志](https://docs.docker.com/engine/reference/commandline/service_create/)。
#2. 使用进程管理器
如果重启策略无法满足你的需求，例如依赖 Docker 容器的 Docker 外部进程，可以使用进程管理器，例如 [upstart](http://upstart.ubuntu.com/)、[systemd](http://freedesktop.org/wiki/Software/systemd/) 或 [supervisor](http://supervisord.org/)。

>警告：不要尝试将 Docker 重启策略与主机级进程管理器结合使用，因为这会产生冲突。

要使用进程管理器，请将其配置为使用通常用于手动启动容器的 `docker start` 或 `docker service` 命令启动容器或服务。有关更多详细信息，请参阅特定流程管理器的文档。
##2.1 在容器内使用进程管理器
进程管理器也可以在容器内运行，以检查进程是否正在运行，如果没运行，则启动/重新启动进程。

>警告：Docker 对这些无感知，只是在容器内监控操作系统进程。

Docker 并不推荐这种方法，因为它依赖于平台，甚至在给定的 Linux 发行版的不同版本中有所不同。