[原文地址](https://docs.docker.com/get-started/part5/)

#1. Prerequisites
- 安装的 Docker 版本在 1.13 以上。
- 获取了第 3 章节讲的 [Docker Compose](https://docs.docker.com/compose/overview/)。
- 获取了第 4 章节讲的 [Docker Machine](https://docs.docker.com/machine/overview/)。
- 读了第 1 章节和第 2 章节，知道如何创建容器。
- 确保已经发布了 friendlyhello 这个镜像并上传到了 registry。
- 确保你的镜像可以部署为容器并运行。运行这个命令，用你的信息替换 username、repo 和 tag：`docker run -p 80:80 username/repo:tag`，然后访问：`http://localhost/`。
- 第 3 章节的 docker-compose.yml 文件。
- 确保第 4 章节设置的虚拟机在运行中。运行 `docker-machine ls` 命令来验证一下。如果停止运行了，通过 `docker-machine start myvm1` 命令启动管理器 manager，然后用 `docker-machine start myvm2` 命令启动 worker。
- 确保第 4 章节创建的 swarm 在运行中。运行 `docker-machine ssh myvm1 "docker node ls"` 来验证一下。如果 swarm 正常工作，则两个节点均报告就绪状态。如果没有，请按照设置 swarm 中的说明重新初始化 swarm 并加入 worker。
#2. 概述
在第 4 章节，你学会了如何设置运行 Docker 的 swarm 集群，并在集群中部署应用，容器在多台机器上运行。
这一章节，你到达了分布式应用的层次结构的最顶层：stack。stack 是一组相互关联的 service，它们可以共享依赖关系，并且可以进行协调和伸缩。 单个 stack 能够定义和协调整个应用程序的功能（尽管非常复杂的应用程序可能需要使用多个 stack）。
好消息是，从第 3 章节创建 Compose 文件并使用 `docker stack deploy` 时，我们就在使用 stack。 但是，生产环境中通常不会在单个主机上运行单个服务堆栈（service stack）。 在这里，你可以把你学到的东西，使多个 service 相互关联，并在多台机器上运行它们。
你做得很好，这就是主场！
#3. 添加新服务并重新部署
很容易添加新服务到 `docker-compose.yml` 文件。首先，添加一个免费的可视化器服务，让我们看看我们的 swarm 是如何调度容器的。
##3.1 编辑 `docker-compose.yml` 文件
将下面的内容填入 `docker-compose.yml` 文件，用你的名字和镜像信息替换 `username/repo:tag`。
```
version: "3"
services:
  web:
    # 用你的名字和镜像信息替换 `username/repo:tag`
    image: username/repo:tag
    deploy:
      replicas: 5
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
    ports:
      - "80:80"
    networks:
      - webnet
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
networks:
  webnet:
```
这里唯一的变化是名为 visualizer 的和 web 平级的一个 service。注意这里的两个新事物：`volumes` 关键字让 visualizer 访问 Docker 的宿主机套接字文件（socket file），`placement` 关键字确保这项服务只能运行在一个 swarm manager 上 - 而不是 worker 上。这是因为这个容器是从 Docker 的 [一个开源项目](https://github.com/ManoMarks/docker-swarm-visualizer) 构建，用来以图表形式展示 swarm 中运行的 Docker 服务。

我们稍后会详细讨论放置 `volumes` 关键字和 `placement` 关键字。
##3.2 确保你的 shell 配置为和 myvm1 交流。
###3.2.1 运行 `docker-machine ls` 列出虚拟机并确保已经连接到了 myvm1（用星号 * 指示）。
###3.2.2 如果有必要，再次运行 `docker-machine env myvm1`，然后运行指定的命令来配置 shell
- Mac 或 Linux 的命令：
```
eval $(docker-machine env myvm1)
```
- Windows 的命令：
```
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression
```
##3.3 再次运行 `docker stack deploy`
在 swarm 管理器上再次运行 `docker stack deploy`，所有需要更新的 service 都会被更新。
```
$ docker stack deploy -c docker-compose.yml getstartedlab
Updating service getstartedlab_web (id: angi1bf5e4to03qu9f93trnxm)
Creating service getstartedlab_visualizer (id: l9mnwkeq2jiononb5ihz9u7a4)
```
##3.4 查看 visualizer
可以看到 Compose 文件中 visualizer 运行在 8080 端口。运行 `docker-machine ls` 命令获取一个节点的 IP 地址。连接到任何一个 IP 和 8080 端口，可以看到 visualizer 正在运行：
![get-started-virtualized](http://img.blog.csdn.net/20180224110519945?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
正如期望的那样，visualizer 的唯一副本在 swarm 管理器上运行，web 的 5 个实例在整个 swarm 集群中分布。可以通过运行 `docker stack ps <stack>` 来确认此可视化：
```
docker stack ps getstartedlab
```
visualizer 是一个独立的服务，可以在 stack 中的包含它的任何应用程序中运行。它不依赖于其他任何东西。现在让我们创建一个具有依赖性的服务：提供访问者计数器的 Redis 服务。
#4. 数据持久化
我们再做一次相同的流程，把存储应用数据的 Redis 数据库添加进来。
##4.1 编辑 docker-compose.yml 文件
保存下面的添加了 Redis 服务的 `docker-compose.yml` 文件。用你的名字和镜像信息替换 `username/repo:tag`。
```
version: "3"
services:
  web:
    # 用你的名字和镜像信息替换 `username/repo:tag`。
    image: username/repo:tag
    deploy:
      replicas: 5
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
    ports:
      - "80:80"
    networks:
      - webnet
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
  redis:
    image: redis
    ports:
      - "6379:6379"
    volumes:
      - "/home/docker/data:/data"
    deploy:
      placement:
        constraints: [node.role == manager]
    command: redis-server --appendonly yes
    networks:
      - webnet
networks:
  webnet:
```
Redis 在 Docker library 中有一个官方镜像，并且已被授予 `redis` 这个简短的镜像名称，所以在这里没有用 `username/repo` 符号。 Redis 的端口 6379 已经由 Redis 预先配置为从容器暴露给主机，并且在我们的 Compose 文件中，我们将它从主机暴露出来，因此可以实际输入任何节点的 IP 将其添加到 Redis Desktop Manager 中并管理此 Redis 实例，如果愿意的话。
注意，要使这个  stack 的部署中实现数据持久化，配置 redis 的时候有两件事要做：

- redis 总是在管理器 manager 上运行，所以总是使用相同的文件系统。
- redis 在主机文件系统中访问任意目录作为容器内的 /data，这是 Redis 存储数据的地方。

这就是在你的**主机物理文件系统中**为 Redis 数据创建可信源（source of truth）。如果没有这个，Redis 会将其数据存储在**容器文件系统中**的 /data 中，如果该容器被重新部署，这些数据将被清除。

这个可信源（source of truth）由两部分组成：

- Redis 服务上的放置约束（placement constraint），确保它始终使用相同的主机。
- 创建的卷（volume），允许容器作为 `/data`（位于 Redis 容器内）访问 `./data`（宿主机上）。在容器创建销毁时，存储在指定主机上的 `./data` 目录中的文件仍然存在，从而保持连续性。

现在可以部署使用了 Redis 的 stack 了。
##4.2 在管理器 manager 上创建  `./data` 目录
```
docker-machine ssh myvm1 "mkdir ./data"
```
##4.3 确保 shell 配置为和 myvm1 交流
###4.3.1 运行 `docker-machine ls` 命令
运行 `docker-machine ls` 命令列出所有虚拟机，并确保连接到了 myvm1（通过星号 * 指示）。
###4.3.2 运行 `docker-machine env myvm1` 命令
如果需要，再次运行 `docker-machine env myvm1` 命令，然后运行下面的命令来配置 shell：

- Mac 或 Linux 的命令：
```
eval $(docker-machine env myvm1)
```
- Windows 的命令：
```
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression
```
##4.4 部署
再次运行 `docker stack deploy` 命令
```
$ docker stack deploy -c docker-compose.yml getstartedlab
```
##4.5 验证服务正常运行
运行 `docker service ls` 命令验证服务正常运行。
```
$ docker service ls
ID                  NAME                       MODE                REPLICAS            IMAGE                             PORTS
x7uij6xb4foj        getstartedlab_redis        replicated          1/1                 redis:latest                      *:6379->6379/tcp
n5rvhm52ykq7        getstartedlab_visualizer   replicated          1/1                 dockersamples/visualizer:stable   *:8080->8080/tcp
mifd433bti1d        getstartedlab_web          replicated          5/5                 orangesnap/getstarted:latest    *:80->80/tcp
```
##4.6 检查网页
检查任何一个节点中的网页，例如 `http://192.168.99.101`，查看通过 Redis 存储数据的访客计数器的结果是否生效：
![app-in-browser-redis](http://img.blog.csdn.net/20180224114349350?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

同时，检查另一个节点上的 8080 端口的 visualizer，注意与 web 和 visualizer 服务一同运行的 redis 服务。

![visualizer-with-redis](http://img.blog.csdn.net/20180224114424768?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#5. 概括
你现在已经了解到 stack 就是相互关联、一致运行的 service，而且从本教程第 3 章节开始，你就一直在使用 stack。 通过插入到 Compose 文件可以将更多 service 添加到 stack。 通过使用放置约束（placement constraint）和卷（volume）的组合，可以创建永久保存数据的位置，以便在容器关闭和重新部署时保存应用程序的数据。