[原文地址](https://docs.docker.com/compose/networking/)
https://docs.docker.com

>注意：只有在使用 Compose 的版本 2 或更高版本时，本文才适用。网络功能不支持版本1（传统）的 Compose 文件。

默认情况下，Compose 会为你的应用程序设置一个 [网络](https://docs.docker.com/engine/reference/commandline/network_create/)。服务的每个容器都加入默认网络，并且该网络上的其他容器都可以访问它们，并且可以通过与容器名称相同的主机名来发现它们。

>注意：根据“project name 项目名称”为你的应用程序的网络提供了一个名称，该名称基于其所处目录的名称可以使用 [`--project-name` 标志](https://docs.docker.com/compose/reference/envvars/#compose-project-name) 或 [`COMPOSE_PROJECT_NAME` 环境变量](https://docs.docker.com/compose/reference/envvars/#compose-project-name) 覆盖项目名称。

例如，假设应用程序位于名为 `myapp` 的目录中，并且 `docker-compose.yml` 如下所示：
```
version: "3"
services:
  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres
    ports:
      - "8001:5432"
```
运行 `docker-compose up` 命令时，会发生以下事件：

1. 创建名为 `myapp_default` 的网络
2. 创建使用 `web` 的配置的容器，并且加入在 `web` 名下的 `myapp_default` 网络
3. 创建使用 `db` 的配置的容器，并且加入在 `db` 名下的 `myapp_default` 网络

>在 v2.1+ 中，`overlay` 网络一直可用 `attachable`
><br/>
从 Compose 2.1 开始，`overlay` 网络总是被创建为 `attachable`，并且这是不可配置的。这意味着独立容器可以连接到 `overlay` 网络。
><br/>
在 Compose 3.x 中，可以选择将 `attachable` 属性设置为 false。

现在每个容器可以查找主机名 `web` 或 `db`，并获取适当的容器的 `IP` 地址。例如，`web` 的应用程序代码可以连接到 `postgres://db:5432` 这个 URL 并开始使用 Postgres 数据库。

注意 `HOST_PORT` 和 `CONTAINER_PORT` 之间的区别很重要。在上面的例子中，对于 `db`，`HOST_PORT` 是 `8001` 而 `CONTAINER_PORT` 是 `5432`（postgres 默认值）。 联网的服务到服务通信使用 `CONTAINER_PORT`。当定义 `HOST_PORT` 时，该服务也可以在 swarm 外访问。

在 `web` 容器中，你的到 `db` 的连接字符串看起来像 `postgres://db:5432`，并且主机的连接字符串看起来像 `postgres://{DOCKER_IP}:8001`。
# 1. 更新容器
如果你对服务的配置进行更改并运行 `docker-compose up` 进行更新，则旧容器将被删除，并且新的容器将以不同的 IP 地址加入网络，但名称相同。正在运行的容器可以查找该名称并连接到新地址，旧地址停止工作。

如果有任何容器连接到旧容器，这些容器将会关闭。检测这种情况，再次查找名称并重新连接是容器的责任。
# 2. Links
Links 允许你定义额外的别名，通过它可以从另一个服务访问服务。他们不需要启用服务进行通信 - 默认情况下，任何服务都可以通过该服务的名称到达任何其他服务。在以下示例中，可通过主机名 `db` 和 `database` 从 `web` 访问 `db`：
```
version: "3"
services:

  web:
    build: .
    links:
      - "db:database"
  db:
    image: postgres
```
更多资料参考 [links 手册](https://docs.docker.com/compose/compose-file/#links)。
# 3. 多主机网络
>注意：本节中的指示信息涉及 [传统的 Docker Swarm](https://docs.docker.com/compose/swarm/) 操作，并且仅在定位传统 Swarm 集群时起作用。有关将 Compose 项目部署到较新的集成 swarm 模式的说明，请参阅 [Docker Stacks 文档](https://docs.docker.com/compose/bundles/)。

在 [将 Compose 应用程序部署到 Swarm 集群](https://docs.docker.com/compose/swarm/) 时，可以使用内置的 `overlay` 驱动程序启用容器之间的多主机通信，而不会更改你的 Compose 文件或应用程序代码。

请参阅 [多主机网络入门](https://docs.docker.com/engine/userguide/networking/get-started-overlay/) 以了解如何设置 Swarm 集群。集群默认使用 `overlay` 驱动程序，但如果你愿意，可以明确指定它 - 请参阅下文了解如何执行此操作。
# 4. 指定自定义网络
可以在顶层上下文使用 `networks` 关键字指定你自己的网络，而不仅仅使用默认的应用程序网络。这使你可以创建更复杂的拓扑并指定 [自定义网络驱动程序](https://docs.docker.com/engine/extend/plugins_network/) 和选项。还可以使用它将服务连接到不受 Compose 管理的外部创建的网络。

每项服务都可以使用服务级上下文中的 `networks` 关键字来指定要连接的网络，该关键字是顶层上下文中使用的 `networks` 关键字列表中的一个条目。

以下是定义两个自定义网络的示例 Compose 文件。代理服务与数据库服务是隔离的，因为它们不共享共享的网络 - 只有应用程序可以与两者通信。
```
version: "3"
services:

  proxy:
    build: ./proxy
    networks:
      - frontend
  app:
    build: ./app
    networks:
      - frontend
      - backend
  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
    # Use a custom driver
    driver: custom-driver-1
  backend:
    # Use a custom driver which takes special options
    driver: custom-driver-2
    driver_opts:
      foo: "1"
      bar: "2"
```
可以通过为每个连接的网络设置 [ipv4_address and/or ipv6_address](https://docs.docker.com/compose/compose-file/#ipv4-address-ipv6-address) 来为网络配置静态 IP 地址。

有关可用网络配置选项的完整详细信息，请参阅以下参考资料：

- [顶级上下文中的 networks 关键字](https://docs.docker.com/compose/compose-file/#network-configuration-reference)
- [服务上下文中的 networks 关键字](https://docs.docker.com/compose/compose-file/#networks)
# 5. 配置默认网络
除了指定自己的网络外，还可以通过在 `networks ` 下的名为 `default` 的网络下定义条目来更改应用程序范围的默认网络的设置：
```
version: "3"
services:

  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres

networks:
  default:
    # Use a custom driver
    driver: custom-driver-1
```
# 6. 使用已经存在的网络
如果想让容器加入一个已经存在的网络，使用 [`external` 选项](https://docs.docker.com/compose/compose-file/#network-configuration-reference)：
```
networks:
  default:
    external:
      name: my-pre-existing-network
```
与其试图创建名为 `[projectname] _default` 的网络，Compose 查找称为 `my-pre-existing-network` 的网络，并将应用的容器连接到它。
