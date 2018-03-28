[原文地址](https://docs.docker.com/compose/overview/)

>最新版的 Compose 文件的手册 [参考这里](https://docs.docker.com/compose/compose-file/)。

Compose 是定义和运行多容器 Docker 应用程序的工具。使用 Compose 时可以通过 YAML 文件来配置应用程序的服务。然后，使用单个命令就可以创建并启动配置中的所有服务。要详细了解 Compose 的所有功能，请参阅下面第一小节  - 特性。

Compose 适用于所有环境：production，staging，development，testing 以及 CI 工作流程。可以参阅下面第二小节  - 常见案例。

使用 Compose 基本上是一个三步过程：

1. 使用 Dockerfile 定义应用程序的环境，以便随时随地进行复制。
2. 在 `docker-compose.yml` 中定义组成应用程序的服务，以便它们可以在隔离的环境中一起运行。
3. 运行 `docker-compose up`，Compose 启动并运行整个应用程序。

`docker-compose.yml` 样式如下：
```
version: '3'
services:
  web:
    build: .
    ports:
    - "5000:5000"
    volumes:
    - .:/code
    - logvolume01:/var/log
    links:
    - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```
更多信息，参考 Compose 文件的手册 [参考这里](https://docs.docker.com/compose/compose-file/)。

Compose 包含管理应用程序完整生命周期的命令：

- 启动、停止、重新构建服务
- 查看运行中服务的状态
- 流式传输运行服务的日志输出
- 在服务上运行一次性命令
# 1. Compose 文档
# 2. 特性
## 2.1 单个主机上的多个隔离环境
Compose 使用项目名称来隔离彼此的环境。可以在多个不同的环境中使用此项目名称：

- 在开发主机上，创建单个环境的多个副本，例如，为项目的每个功能分支运行稳定副本时
- 在 CI 服务器上，为防止构建互相干扰，可以将项目名称设置为唯一的构建编号
- 在共享主机或开发主机上，防止可能使用相同服务名称的不同项目相互干扰

默认项目名称是项目目录的基本名称。可以使用 `-p` [命令行选项](https://docs.docker.com/compose/reference/overview/) 或 `COMPOSE_PROJECT_NAME` [环境变量](https://docs.docker.com/compose/reference/envvars/#compose-project-name) 设置自定义项目名称。
## 2.2 创建容器时保留卷数据
Compose 会保留服务使用的所有卷。当 `docker-compose up` 运行时，如果它找到之前运行的任何容器，则它将卷从旧容器复制到新容器。此过程可确保你在卷中创建的任何数据都不会丢失。

如果在 Windows 机器上使用 `docker-compose`，请参阅 [环境变量](https://docs.docker.com/compose/reference/envvars/#compose-project-name) 并根据你的特定需求调整必要的环境变量。
## 2.3 只重新创建已更改的容器
Compose 缓存用于创建容器的配置。当你重新启动未更改的服务时，Compose 将再次使用已有的容器。重复利用容器意味着可以快速更改环境。
## 2.4 变量及在环境之间移动组合（moving a composition between environments）
Compose 支持 Compose 文件中的变量。可以使用这些变量为不同的环境或不同的用户自定义组合。更多详情，请参阅 [变量替换](https://docs.docker.com/compose/compose-file/#variable-substitution)。

可以使用 `extends` 字段或通过创建多个 Compose 文件来扩展 Compose 文件。请参阅 [extends](https://docs.docker.com/compose/extends/) 了解更多细节。
# 3. 常见用例
Compose 可以用不同的方式使用。
## 3.1 开发环境
在开发软件时，在孤立环境中运行应用程序并与其交互的能力至关重要。Compose 命令行工具可用于创建环境并与之交互。

[Compose 文件](https://docs.docker.com/compose/compose-file/) 提供了一种方式来记录和配置所有应用程序的服务依赖关系（数据库，队列，缓存，Web 服务 API 等）。通过 Compose 命令行工具，可以使用单个命令（`docker-compose up`）为每个依赖项创建和启动一个或多个容器。

总之，这些功能为开发人员开始项目提供了一种便捷方式。Compose 可以将很多页的“开发人员入门指南”减少到一个机器可读的 Compose 文件和一些命令。
## 3.2 自动测试环境
任何持续部署或持续集成过程的一个重要部分是自动化测试套件。自动化的端到端测试需要一个运行测试的环境。Compose 提供了一种创建和销毁测试套件的独立测试环境的便捷方式。通过在 Compose 文件中定义的完整环境，可以通过几条命令创建和销毁这些环境：
```
$ docker-compose up -d
$ ./run_tests
$ docker-compose down
```
## 3.3 单个主机部署
Compose 在传统上一直专注于开发和测试工作流程，但在每个版本中都会增加更多面向生产的功能。可以使用 Compose 部署到远程 Docker 引擎。Docker 引擎可以是 [Docker Machine](https://docs.docker.com/machine/overview/) 或整个 [Docker Swarm](https://docs.docker.com/engine/swarm/) 集群的单个实例。

有关使用面向生产功能的详细信息，请参阅 [生产环境中的 Compose](https://docs.docker.com/compose/production/)。
