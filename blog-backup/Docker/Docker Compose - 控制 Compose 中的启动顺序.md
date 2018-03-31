[原文地址](https://docs.docker.com/compose/startup-order/)

可以通过 [depends_on](https://docs.docker.com/compose/compose-file/#depends-on) 选项控制服务的启动顺序。Compose 总是按照依赖顺序启动容器，依赖由 `depends_on`、`links`、`volumes_from` 和 `network_mode: "service:..."` 决定。

但是，Compose 不会等到容器“准备就绪”（无论对于特定应用程序而言是什么意思） - 直到它运行（Compose does not wait until a container is “ready” (whatever that means for your particular application) - only until it’s running）。这有一个很好的理由。

等待数据库准备就绪的问题实际上只是分布式系统的一个更大问题的一个子集。在生产中，数据库可能随时无法使用。应用程序需要适应这些类型的故障。

要处理这个问题，请将应用程序设计为**在发生故障后重新建立与数据库的连接**。如果应用程序重试连接，它最终可以连接到数据库。

最好的解决方案是在你的应用程序代码中执行这个检查，无论是在启动时，还是因任何原因丢失连接。但是，如果你不需要此级别的恢复能力，则可以使用包装脚本解决该问题：
#### 1. 使用 [wait-for-it](https://github.com/vishnubob/wait-for-it)，[dockerize](https://github.com/jwilder/dockerize) 或兼容脚本的 [wait-for](https://github.com/Eficode/wait-for)。这些小包装脚本可以包含在应用程序的镜像中，以轮询给定的主机和端口，直到它接受 TCP 连接。

例如，使用 wait-for-it.sh 或 wait-for 来封装你的服务中的命令：
```
version: "2"
services:
  web:
    build: .
    ports:
      - "80:8000"
    depends_on:
      - "db"
    command: ["./wait-for-it.sh", "db:5432", "--", "python", "app.py"]
  db:
    image: postgres
```
>提示：这个解决方案有一些限制。例如，它不会验证特定服务何时准备就绪。如果向命令添加更多参数，请使用带循环的 `bash shift` 命令，如下例所示。
#### 2. 或者，编写自己的包装脚本以执行更多特定于应用程序的运行状况检查。例如，你可能要等到 Postgres 完全准备好接受命令：
```
#!/bin/bash
# wait-for-postgres.sh

set -e

host="$1"
shift
cmd="$@"

until psql -h "$host" -U "postgres" -c '\q'; do
  >&2 echo "Postgres is unavailable - sleeping"
  sleep 1
done

>&2 echo "Postgres is up - executing command"
exec $cmd
```
可以像前面的示例一样将其用作包装脚本，方法是设置：
```
command: ["./wait-for-postgres.sh", "db", "python", "app.py"]
```
