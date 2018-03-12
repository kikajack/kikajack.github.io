[原文地址](https://docs.docker.com/get-started/part3/)

#1. 先决条件 Prerequisites
- 安装的 Docker 版本在 1.13 以上。
- 获取了 [Docker Compose](https://docs.docker.com/compose/overview/)。在 Docker for Mac 和 Docker for Windows 中是默认安装的，但在 Linux 系统中需要 [安装](https://github.com/docker/compose/releases)。Windows 10 之前的系统没有 Hyper-V，需要使用 [Docker Tollbox](https://docs.docker.com/toolbox/overview/)。
- 读了第一部分和第二部分，直到如何创建容器。
- 确保你已经发布了 friendlyhello 这个镜像并上传到了 registry。
- 确保你的镜像可以部署为容器并运行。运行这个命令，用你的信息替换 username、repo 和 tag：`docker run -p 80:80 username/repo:tag`，然后访问：`http://localhost/`。
#2. 概述
在这一章，我们扩展应用并开启负载均衡。要实现这些，必须去分布式应用层级架构的上一层：service。

- Stack
- Services
- Container
#3. 关于 service
在分布式应用程序中，应用程序的不同部分被称为“服务”。例如，如果有一个视频共享网站，它可能包括一个用于将应用程序数据存储在数据库中的服务，一个在用户上传东西后在后台进行视频转码的服务，一个用于前端页面的服务等等。
服务实际上只是“生产中的容器”。每个服务只运行一个映像，但它编码了镜像的运行方式 - 应该使用哪个端口，容器应运行多少个副本以满足性能要求等等。 伸缩服务可以更改运行该软件的容器实例的数量，从而为进程中的服务分配更多计算资源。
定义、运行和伸缩 Docker 平台的服务很简单，只需要写一个 `docker-compose.yml` 文件。
#4. 第一个 `docker-compose.yml` 文件
`docker-compose.yml` 文件是 YAML 格式的，定义了生产中的 Docker 容器的表现。
##4.1 docker-compose.yml
`docker-compose.yml` 文件放在哪了都行。确保上一章节中创建的镜像已经上传到 registry 中了，并且用这个镜像信息替换下面文件中的 `username/repo:tag`。
```
version: "3"
services:
  web:
    # 用你的镜像信息替换 `username/repo:tag`
    image: username/repo:tag
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"
    networks:
      - webnet
networks:
  webnet:
```
这个 `docker-compose.yml` 文件会让 Docker 做下面的事情：

- 从 registry 下载镜像。

- 在一个名为 web 的 service 中运行 5 个实例，限制每个实例最多使用 10% 的 CPU（在所有核心上），最多使用 50 MB 内存。

- 如果容器挂掉，立刻重启。

- 将容器的 80 端口映射到宿主机的 80 端口。

- 指示 web 服务的容器通过名为 webnet 的负载均衡网络共享 80 端口。在内部，容器本身通过临时端口发布到 web 服务的 80 端口。

- 用默认设置（load-balanced overlay network）来定义 webnet 网络。
#5. 运行这个 load-balanced 应用
在使用 `docker stack deploy` 命令之前首先运行：
```
docker swarm init
```
注意：在下一章节就会知道这句话的意思。如果没有运行 `docker swarm init`，会报错 `this node is not a swarm manager`。

现在可以运行了。需要为应用程序指定名字，这里是 getstartedlab：
```
docker stack deploy -c docker-compose.yml getstartedlab
```
我们唯一的 service stack 正在运行 5 个我们之前部署的镜像的实例。下面来检查一下。
获取我们应用中的 service 的 ID：
```
docker service ls
```
查找输出中以你的应用程序名称作为前缀的 web 服务。 如果其命名与此示例中的相同，则服务名称为 getstartedlab_web。 此外还列出了 service ID 以及副本数量，镜像名称和暴露的端口号。
服务中运行的每一个容器叫做一个任务 task。每个任务都有独立的、数值增加的 ID，和 `docker-compose.yml` 文件中定义的副本个数相关。列出服务中的 task：
```
docker service ps getstartedlab_web
```
如果您只列出了系统中的所有容器，任务也显示出来，尽管这不是按服务过滤的：
```
docker container ls -q
```

可以在一行运行多次 `curl -4 http://localhost`，或在浏览器中打开这个 URL 并刷新几次。
![这里写图片描述](http://img.blog.csdn.net/20180223202559157?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
不管怎样，容器的 ID 都会变化，证明负载均衡工作了；对于每个请求，这 5 个 task 中的一个会被选中（通过 round-robin 方式）用来响应。容器的 ID 和之前命令的输出一致（`docker container ls -q`）。

Windows 10 的 PowerShell 应该具备 curl 功能，如果没有的话，你可以使用 Linux 终端模拟器（比如 Git BASH），或下载 wget。

如果响应很慢？
根据你的环境的网络配置，容器响应一个 HTTP 请求可能会花费 30 秒。这并不代表 Docker 或 swarm 的性能，而是我们稍后在本教程中讨论的未满足的 Redis 依赖项。 因为同样的原因，访客计数器也不能工作，我们还没有添加服务来保存数据。
#6. 伸缩应用
可以通过改变 `docker-compose.yml` 文件的 `replicas` 的值来扩展应用。保存变更后需要重新运行 `docker stack deploy` 命令：
```
docker stack deploy -c docker-compose.yml getstartedlab
```
Docker 执行一个就地更新，不需要先停下 stack 或杀死任何容器。

现在，再次运行 `docker container ls -q` 来查看重新配置了的部署的实例。如果增大了 replicas，会更多运行的 task，因此启动更多的容器。
##6.1 取下应用程序和 swarm
- 通过 `docker stack rm` 命令关闭应用：
```
docker stack rm getstartedlab
```
- 关闭 swarm：
```
docker swarm leave --force
```
通过 Docker 很容易启动和伸缩应用。你已经朝着学习如何在生产中运行容器迈出了一大步。 接下来，将学习如何将这个应用程序作为 Docker 机器集群上的真正集群运行（how to run this app as a bonafide swarm on a cluster of Docker machines）。

注意：像这样的 Compose 文件用来定义使用 Docker 的应用，可以被上传到使用 [Docker Cloud](https://docs.docker.com/docker-cloud/) 的云服务商，或使用 Docker 企业版的硬件或云服务商。
#7. 概括和备忘录
虽然通过 `docker run` 启动容器很简单，但是生产中容器的使用方式是当做服务 service 来运行。service 在 Compose 文件中编码了容器的表现，这个文件可以用来伸缩、限制和部署应用。对 service 的变更可以通过再次运行启动服务的命令 `docker stack deploy` 就地生效。

这一章的命令：
```
docker stack ls                                 # 列出 stacks 或应用
docker stack deploy -c <composefile> <appname>  # 运行指定的 Compose 文件
docker service ls            # 列出运行的、关联到应用的 service
docker service ps <service>  # 列出关联到应用的任务 task
docker inspect <task or container>  # 检查 task 或 container
docker container ls -q          # 列出容器的 ID
docker stack rm <appname>       # 关闭应用
docker swarm leave --force      # 从管理端关闭 swarm 的一个节点
```