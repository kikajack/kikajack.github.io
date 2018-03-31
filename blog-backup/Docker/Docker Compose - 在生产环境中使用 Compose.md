[原文地址](https://docs.docker.com/compose/production/)

在开发环境中使用 Compose 定义应用程序时，可以使用此定义在不同环境中运行应用程序，如 CI，staging 和生产环境。

部署应用程序的最简单方法是在单个服务器上运行它，这与运行开发环境的方式类似。如果要扩展应用程序，可以在 Swarm 群集上运行 Compose 应用程序。
# 1. 为生产环境修改 Compose 文件
可能需要对应用配置进行更改，以便为生产做好准备。这些更改可能包括：

- 删除应用程序代码的任何卷绑定，以便代码保留在容器内部，并且不能从外部更改
- 绑定到主机上的不同端口
- 设置各种环境变量，例如，减少日志的详细程度或启用电子邮件发送
- 指定类似 `restart: always` 的重启策略以避免停机
- 增加额外的服务，如日志聚合器

出于这个原因，考虑定义一个额外的 Compose文 件，比如 `production.yml`，它指定了适合生产的配置。此配置文件只需包含要从原始 Compose 文件中进行的更改。额外的 Compose 文件可以在原始的 `docker-compose.yml` 上以创建新的配置。

一旦获得了第二个配置文件，请使用 `-f` 选项告诉 Compose：
```
docker-compose -f docker-compose.yml -f production.yml up -d
```
更多示例参考 [这里](https://blog.csdn.net/kikajack/article/details/79764263)。
## 1.1 部署更改
修改应用程序代码后，记得重新构建镜像并重新创建应用程序的容器。要重新部署名为 `web` 的服务，使用：
```
$ docker-compose build web
$ docker-compose up --no-deps -d web
```
第一句命令重新构建 `web` 镜像，然后停止、销毁并重新创建 `web` 服务。`--no-deps` 标志防止 Compose 同时创建任何 `web` 所依赖的服务。
## 1.2 在单独的服务器上运行 Compose
通过适当设置 `DOCKER_HOST`，`DOCKER_TLS_VERIFY` 和 `DOCKER_CERT_PATH` 环境变量，可以使用 Compose 将应用程序部署到远程 Docker 主机。对于这样的任务，Docker Machine 使得管理本地和远程 Docker 主机变得非常简单，即使不远程部署也是推荐的。

一旦设置了环境变量，所有普通的 `docker-compose` 命令都不需要进一步的配置。
## 1.3 在 swarm 集群上运行 Compose
Docker Swarm 是 Docker 原生的集群系统，它将相同的 API 公开为单个 Docker 主机（exposes the same API as a single Docker host），这意味着可以将 Compose 用于 Swarm 实例并在多个主机上运行应用程序。

在 [集成指南](https://docs.docker.com/compose/swarm/) 中阅读有关 Compose/Swarm 集成的更多信息。
