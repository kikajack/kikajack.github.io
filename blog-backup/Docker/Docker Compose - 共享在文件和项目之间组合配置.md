[原文地址](https://docs.docker.com/compose/extends/)
https://docs.docker.com

Compose 支持两种方式共享配置：

1. 使用多个 Compose 文件扩展整个 Compose 文件
2. 使用 `extends` 字段扩展单个服务
# 1. 多个 Compose 文件
使用多个 Compose 文件可以为不同的环境或不同的工作流自定义 Compose 应用程序。
## 1.1 理解多个 Compose 文件
默认情况下，Compose 读取两个文件，即 `docker-compose.yml` 和可选的 `docker-compose.override.yml` 文件。按照惯例，`docker-compose.yml` 包含你的基本配置。顾名思义，override 这个文件可以覆盖已有服务或全新服务的配置。

如果同时在两个文件中定义了服务，Compose 将使用下面第三部分 添加和覆盖配置 中描述的规则合并配置。

要使用多个 override 文件或具有不同名称的 override 文件，可以使用 `-f` 选项指定文件列表。按照它们在命令行上指定的顺序组合文件。有关使用 `-f` 的更多信息，请参阅 [docker-compose 命令参考](https://docs.docker.com/compose/reference/overview/)。

在使用多个配置文件时，必须确保这些文件中的所有路径都与基础 Compose 文件（使用 `-f` 指定的第一个 Compose 文件）相关。这是必需的，因为 override 文件不需要是有效的 Compose 文件。override 文件可以包含小部分配置。跟踪哪个服务片段与哪个路径相关是困难和混乱的，因此为了使路径更容易理解，所有路径必须相对于基本文件进行定义。
## 1.2 用例
在本节中，有两个常见的用于多个 compose 文件的使用案例：更改 Compose 应用程序以适应不同环境，以及针对 Compose 应用程序运行管理任务。
### 不同环境
多文件的常见用例是为各种生产环境（可能是production，staging 或 CI）更改开发环境中的 Compose 应用程序。为了支持这些差异， 可以将你的Compose 配置分成几个不同的文件：

从定义服务规范配置的基础文件开始。
#### docker-compose.yml
```
web:
  image: example/my_web_app:latest
  links:
    - db
    - cache

db:
  image: postgres:latest

cache:
  image: redis:latest
```
在此示例中，开发环境中的配置向主机公开了一些端口，将我们的代码作为卷装入，并构建 Web 镜像。
#### docker-compose.override.yml
```
web:
  build: .
  volumes:
    - '.:/code'
  ports:
    - 8883:80
  environment:
    DEBUG: 'true'

db:
  command: '-d'
  ports:
    - 5432:5432

cache:
  ports:
    - 6379:6379
```
运行 `docker-compose up` 时会自动读取 override 文件。

现在，在生产环境中可以正常使用此 Compose 应用程序。因此，创建另一个 override 文件（可能存储在不同的 git 仓库或由不同的团队管理）。
#### docker-compose.prod.yml
```
web:
  ports:
    - 80:80
  environment:
    PRODUCTION: 'true'

cache:
  environment:
    TTL: '500'
```
可以运行下面的命令来部署这个生产环境中的 Compose 文件：
```
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```
这将使用 `docker-compose.yml` 和 `docker-compose.prod.yml` 中的配置（但不是 `docker-compose.override.yml` 中的 dev 配置）部署所有三个服务。

有关生产中的 Compose 的更多信息，请参阅 [生产](https://docs.docker.com/compose/production/)。
### 管理任务
另一个常见用例是针对 Compose 应用程序中的一个或多个服务运行 adhoc 或管理任务。这个例子演示了运行数据库备份。

从 `docker-compose.yml` 开始。
```
web:
  image: example/my_web_app:latest
  links:
    - db

db:
  image: postgres:latest
```
在 `docker-compose.admin.yml` 中添加新服务来运行数据库导出或备份：
```
dbadmin:
  build: database_admin/
  links:
    - db
```
通过 `docker-compose up -d` 命令启动正常环境，在命令中添加 `docker-compose.admin.yml` 参数可以启动数据库备份。
```
docker-compose -f docker-compose.yml -f docker-compose.admin.yml \
    run dbadmin db-backup
```
# 2. 扩展服务
>注意：`extends` 关键字最高在 v2.1 版本的 Compose 文件格式中支持（请参阅 [v1中的 `extends`](https://docs.docker.com/compose/compose-file/compose-file-v1/#extends) 和 [v2中的 `extends`](https://docs.docker.com/compose/compose-file/compose-file-v2/#extends)），但在 3.x 中不支持。请参阅第3版的添加和删除键摘要以及有关 [如何升级](https://docs.docker.com/compose/compose-file/compose-versioning/#upgrading) 的信息。请参阅 [moby/moby＃31101](https://github.com/moby/moby/issues/31101) 以关注在未来版本中添加对某种形式的扩展的支持的可能性。

Docker Compose 的 `extends` 关键字使得可以在不同文件甚至不同项目之间共享通用配置。 如果有多个服务可以重复使用一组通用配置选项，则扩展服务很有用。使用 `extends` 你可以在一个地方定义一套通用的服务选项并从任何地方引用它。

请记住，`links`，`volumes_from` 和 `depends_on` 永远不会在使用 `extends` 的服务之间共享。这些例外存在以避免隐式依赖性（ implicit dependencies），你应该始终在本地定义 `links` 和 `volumes_from`。这可以确保在读取当前文件时，服务之间的依赖关系清晰可见。本地定义也可以确保对引用文件的更改不会破坏任何内容。
## 2.1 了解扩展配置
在 `docker-compose.yml` 中定义任何服务时，可以声明你正在扩展另一个服务，如下所示：
```
web:
  extends:
    file: common-services.yml
    service: webapp
```
这指示 Compose 重新使用 `common-services.yml` 文件中定义的 webapp 服务的配置。假设 `common-services.yml` 如下所示：
```
webapp:
  build: .
  ports:
    - "8000:8000"
  volumes:
    - "/data"
```
在这种情况下，可以得到与直接在 web 中使用相同的构建，端口和卷配置值编写 `docker-compose.yml` 完全相同的结果。

可以进一步在 `docker-compose.yml` 中本地定义（或重新定义）配置：
```
web:
  extends:
    file: common-services.yml
    service: webapp
  environment:
    - DEBUG=1
  cpu_shares: 5

important_web:
  extends: web
  cpu_shares: 10
```
还可以编写其他服务并将你的 Web 服务链接到它们：
```
web:
  extends:
    file: common-services.yml
    service: webapp
  environment:
    - DEBUG=1
  cpu_shares: 5
  links:
    - db
db:
  image: postgres
```
## 2.2 用例
当多个服务具有通用的配置时，扩展单个服务很有用。下面的例子是一个包含两个服务的 Compose 应用程序：一个 web 应用程序和一个队列。这两个服务使用相同的代码库并共享许多配置选项。

在 `common.yml` 中定义了通用配置：
```
app:
  build: .
  environment:
    CONFIG_FILE_PATH: /code/config
    API_KEY: xxxyyy
  cpu_shares: 5
```
在 `docker-compose.yml` 中定义使用通用配置的具体服务：
```
webapp:
  extends:
    file: common.yml
    service: app
  command: /code/run_web_app
  ports:
    - 8080:8080
  links:
    - queue
    - db

queue_worker:
  extends:
    file: common.yml
    service: app
  command: /code/run_worker
  links:
    - queue
```
# 3. 添加并覆盖配置
Compose 将原始服务的复制配置复制到本地服务。如果在原始服务和本地服务中都定义了配置选项，则本地值将替换或扩展原始值。

对于像 `image`，`command` 或 `mem_limit` 这样的单值选项，新值将替换旧值。
```
# original service
command: python app.py

# local service
command: python otherapp.py

# result
command: python otherapp.py
```
>Compose 第一版文件中的 `build` 和 `image`
><br/>
>对于 `build` 和 `image`，使用 Compose 第一版文件时，使用本地服务中的一个选项会导致 Compose 在原始服务中定义其他选项时放弃其他选项。
><br/>
>例如，如果原始服务定义了 `imagew:webapp` 并且本地服务定义了 `build: .` 那么生成的服务就有一个  `build: .` 并没有 `image` 选项。
><br/>
>这是因为  `build` 和 `image` 不能在版本1文件中一起使用。

对于多值选项 `ports`，`expose`，`external_links`，`dns`，`dns_search` 和 `tmpfs`，Compose 将两组值连接起来：
```
# original service
expose:
  - "3000"

# local service
expose:
  - "4000"
  - "5000"

# result
expose:
  - "3000"
  - "4000"
  - "5000"
```
在环境，标签，卷和设备的情况下，。 对于环境和标签，环境变量或标签名称决定使用哪个值：
对于 `environment`、`labels`、`volumes` 和 `devices`，Compose 优先使用本地定义的值“合并”条目。对于 `environment`、`labels`，环境变量和标签名决定使用哪个值：
```
# original service
environment:
  - FOO=original
  - BAR=original

# local service
environment:
  - BAR=local
  - BAZ=local

# result
environment:
  - FOO=original
  - BAR=local
  - BAZ=local
```
`volumes` 和 `devices`通过容器中的挂载路径合并（merge）：
```
# original service
volumes:
  - ./original:/foo
  - ./original:/bar

# local service
volumes:
  - ./local:/bar
  - ./local:/baz

# result
volumes:
  - ./original:/foo
  - ./local:/bar
  - ./local:/baz
```
