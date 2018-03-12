[原文地址](https://docs.docker.com/get-started/part6/)

#1. 先决条件 Prerequisites
- 安装的 Docker 版本在 1.13 以上。
- 获取了第 3 章节讲的 [Docker Compose](https://docs.docker.com/compose/overview/)。
- 获取了第 4 章节讲的 [Docker Machine](https://docs.docker.com/machine/overview/)。
- 读了第 1 章节和第 2 章节，知道如何创建容器。
- 确保已经发布了 friendlyhello 这个镜像并上传到了 registry。
- 确保你的镜像可以部署为容器并运行。运行这个命令，用你的信息替换 username、repo 和 tag：`docker run -p 80:80 username/repo:tag`，然后访问：`http://localhost/`。
- 第 5 章节的 docker-compose.yml 文件。
#2. 概述
你一直在为这个系列教程编辑同一个 Compose 文件。好消息是这个 Compose 文件在生产中的效果与在您的计算机上的效果相同。 在这里，我们通过一些选项来运行 Dockerized 应用程序。
#3. Choose an option
如果你的生产环境使用的是 `Docker Community Edition`，可以使用 `Docker Cloud` 来辅助管理特定服务提供商上的应用，比如 Amazon Web Services、DigitalOcean 和 Microsoft Azure。

设置和部署：

- 将 Docker Cloud 和你的首选服务提供商连接，为 Docker Cloud 授权，以便自动配置和将虚拟机容器化（Dockerize）。
- 使用 Docker Cloud 创建计算资源并创建 swarm。
- 部署应用程序。

注意：这里并没有列出 Docker Cloud 的文档，务必在完成每个步骤后回到这个页面。
##3.1 连接 Docker Cloud
可以在标准模式或 swarm 模式下运行 Docker Cloud。

如果在标准模式下运行 Docker Cloud，按照下面的指示将你的服务提供商连接到 Docker Cloud。

- [Amazon Web Services setup guide](https://docs.docker.com/docker-cloud/cloud-swarm/link-aws-swarm/)
- [DigitalOcean setup guide](https://docs.docker.com/docker-cloud/infrastructure/link-do/)
- [Microsoft Azure setup guide](https://docs.docker.com/docker-cloud/infrastructure/link-azure/)
- [Packet setup guide](https://docs.docker.com/docker-cloud/infrastructure/link-packet/)
- [SoftLayer setup guide](https://docs.docker.com/docker-cloud/infrastructure/link-softlayer/)
- [Use the Docker Cloud Agent to bring your own host](https://docs.docker.com/docker-cloud/infrastructure/byoh/)

如果是在 swarm 模式下运行（建议用于 Amazon Web Services 或 Microsoft Azure），直接跳到下一部分 创建 swarm。
##3.2 创建 swarm
- 如果使用的是 Amazon Web Services (AWS) 可以 [在 AWS 上自动创建 swarm](https://docs.docker.com/docker-cloud/cloud-swarm/create-cloud-swarm-aws/)

- 如果使用 Microsoft Azure 可以 [在 Azure 上自动创建 swarm](https://docs.docker.com/docker-cloud/cloud-swarm/create-cloud-swarm-azure/)

- 其他情况下，在 Docker Cloud UI 中 [创建节点](https://docs.docker.com/docker-cloud/getting-started/your_first_node/)，然后 [通过 Docker Cloud 使用 SSH](https://docs.docker.com/docker-cloud/infrastructure/ssh-into-a-node/) 运行 `docker swarm init` 命令和 `docker swarm join` 命令 [开启 swarm 模式](https://docs.docker.com/docker-cloud/cloud-swarm/using-swarm-mode/)。最终，点击屏幕顶部的开关开启 swarm 模式并 [注册刚创建的 swarm](https://docs.docker.com/docker-cloud/cloud-swarm/register-swarms/)。

注意：如果 [使用 Docker Cloud Agent 自带主机](https://docs.docker.com/docker-cloud/infrastructure/byoh/)，则此程序不支持 swarm 模式（If you are Using the Docker Cloud Agent to Bring your Own Host, this provider does not support swarm mode）。可以使用 Docker Cloud [注册自己的已存在的 swarm](https://docs.docker.com/docker-cloud/cloud-swarm/register-swarms/)。
##3.3 部署应用到云端
###3.3.1 [通过 Docker Cloud 连接到 swarm](https://docs.docker.com/docker-cloud/cloud-swarm/connect-to-swarm/)
有两种不同的连接方式：

- 在 Swarm 模式下的 Docker Cloud 网页接口中，选择页面顶部的 swarm，点击要连接的 swarm，复制粘贴给定的命令到命令行终端。
![cloud-swarm-connect](http://img.blog.csdn.net/20180224145154022?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 在 Mac 或 Windows 版本的 Docker 中，可以通过 [桌面应用菜单快速连接到 swarm](https://docs.docker.com/docker-cloud/cloud-swarm/connect-to-swarm/#use-docker-for-mac-or-windows-edge-to-connect-to-swarms)。
![cloud-swarm-connect-desktop](http://img.blog.csdn.net/20180224145238884?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

无论使用哪种方式，都将打开一个上下文是本地计算机的终端，但其 Docker 命令会路由到云服务提供商上运行的 swarm。 你可以直接访问本地文件系统和远程 swarm，从而启用纯 docker 命令。
###3.3.2 部署应用到云端
运行 `docker stack deploy -c docker-compose.yml getstartedlab` 命令来部署应用到云端的 swarm。
```
 docker stack deploy -c docker-compose.yml getstartedlab

 Creating network getstartedlab_webnet
 Creating service getstartedlab_web
 Creating service getstartedlab_visualizer
 Creating service getstartedlab_redis
```
现在应用运行在云端了。
###3.3.3 运行 swarm 命令验证部署效果
可以使用 swarm 命令行来浏览和管理 swarm。下面是几个看上去类似的例子：

- 通过 `docker node ls` 查看节点：
```
  [getstartedlab] ~ $ docker node ls
  ID                            HOSTNAME                                      STATUS              AVAILABILITY        MANAGER STATUS
  9442yi1zie2l34lj01frj3lsn     ip-172-31-5-208.us-west-1.compute.internal    Ready               Active              
  jr02vg153pfx6jr0j66624e8a     ip-172-31-6-237.us-west-1.compute.internal    Ready               Active              
  thpgwmoz3qefdvfzp7d9wzfvi     ip-172-31-18-121.us-west-1.compute.internal   Ready               Active              
  n2bsny0r2b8fey6013kwnom3m *   ip-172-31-20-217.us-west-1.compute.internal   Ready               Active              Leader
```
- 通过 `docker service ls` 查看服务：
```
[getstartedlab] ~/sandbox/getstart $ docker service ls
ID                  NAME                       MODE                REPLICAS            IMAGE                             PORTS
x3jyx6uukog9        dockercloud-server-proxy   global              1/1                 dockercloud/server-proxy          *:2376->2376/tcp
ioipby1vcxzm        getstartedlab_redis        replicated          0/1                 redis:latest                      *:6379->6379/tcp
u5cxv7ppv5o0        getstartedlab_visualizer   replicated          0/1                 dockersamples/visualizer:stable   *:8080->8080/tcp
vy7n2piyqrtr        getstartedlab_web          replicated          5/5                 sam/getstarted:part6    *:80->80/tcp
```
- 通过 `docker node ps <service>` 查看服务的任务 task：
```
[getstartedlab] ~/sandbox/getstart $ docker service ps vy7n2piyqrtr
ID                  NAME                  IMAGE                            NODE                                          DESIRED STATE       CURRENT STATE            ERROR               PORTS
qrcd4a9lvjel        getstartedlab_web.1   sam/getstarted:part6   ip-172-31-5-208.us-west-1.compute.internal    Running             Running 20 seconds ago                       
sknya8t4m51u        getstartedlab_web.2   sam/getstarted:part6   ip-172-31-6-237.us-west-1.compute.internal    Running             Running 17 seconds ago                       
ia730lfnrslg        getstartedlab_web.3   sam/getstarted:part6   ip-172-31-20-217.us-west-1.compute.internal   Running             Running 21 seconds ago                       
1edaa97h9u4k        getstartedlab_web.4   sam/getstarted:part6   ip-172-31-18-121.us-west-1.compute.internal   Running             Running 21 seconds ago                       
uh64ez6ahuew        getstartedlab_web.5   sam/getstarted:part6   ip-172-31-18-121.us-west-1.compute.internal   Running             Running 22 seconds ago     
```   
###3.3.4 开启云服务器的端口
此时应用已经在云服务器上作为 swarm 集群部署好了，正如刚刚运行的 docker 命令所证明的那样。但是，您仍然需要在云服务器上打开端口，以便：

- 允许每个 worker 节点上的 redis 和 web 服务互相通信

- 允许到 worker 节点上的 web 服务的入站流量，以使 Hello World 和 Visualizer 可以从浏览器访问。

- 允许到运行 swarm 管理器的服务器上 SSH 服务的入站流量（可能已经有云服务提供商设置好了）。

这些是每个服务需要暴露的端口：
| 服务 | 类型 | 协议 | 端口 |
|-|-|-|-|
|web| HTTP| TCP|  80|
|visualizer|  HTTP| TCP|  8080|
|redis| TCP|  TCP|  6379|
开启端口的方法，参考每个云服务提供商，不再举例。

>用于数据持久化的 redis 服务怎么设置？
要让 redis 服务工作，需要在运行 `docker stack deploy` 命令前通过 ssh 登入运行 swarm 管理器的那台云服务器，并在目录 `/home/docker/` 中创建 `data/` 目录。另一种选择是将 `docker-stack.yml` 文件中的数据路径改为一个运行 swarm 管理器的那台云服务器上已经存在的目录。这个例子中的 redis 因为没有设置路径，所以无法使用。
##3.4 迭代和清理
现在可以使用之前学过的所有知识了。

- 通过改变 `docker-compose.yml` 文件来伸缩应用，执行 `docker stack deploy` 命令就地重新部署应用。

- 通过编辑代码、重新构建并推送新镜像来改变应用程序的表现。

- 通过 `docker stack rm` 命令删除 stack：
```
docker stack rm getstartedlab
```
与在本地 Docker 虚拟机上运行 swarm 的场景不同，不管是否关闭本地主机，swarm 和部署在其上的任何应用程序都将继续在云服务器上运行。
#4. 祝贺!
你已经对使用 Docker 平台进行从开发到部署的完整流程进行了学习。

Docker 平台上，有更多的知识是这里没有提到的，但是你已经对 container、image、service、swarm、stack、scaling、load-balancing、volume、placement constraint 有了基本认识。

想要深入学习吗？下面是我们建议的资源：

- [Samples](https://docs.docker.com/samples/)：包含了多个运行在容器中的流行软件的示例，以及一些教授最佳实践的实验室。
- [User Guide](https://docs.docker.com/engine/userguide/)：用户向导中有几个相对深入的解释网络和存储的例子。
- [Admin Guide](https://docs.docker.com/engine/admin/)：管理员向导中讲述了如何管理容器化的生产环境。
- [Training](https://training.docker.com/)：官方的 Docker 课程，提供面对面的教学和虚拟教室环境。
- [Blog](https://blog.docker.com/)：涵盖了 Docker 最近发生的一切。