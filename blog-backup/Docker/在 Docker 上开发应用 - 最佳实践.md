[原文地址](https://docs.docker.com/develop/dev-best-practices/)

下面的开发模式对通过 Docker 构建应用程序的开发人员是有用的。
#1. 使镜像尽可能小
小镜像可以更快的通过网络传输，在启动容器或服务时更快的加载到内存中。下面是几个保持镜像的小体积的经验法则：

##1.1 从合适的基础镜像开始
例如，如果你需要 JDK，可以考虑用官方的 `openjdk` 镜像做基础镜像，而不是从通用的 `ubuntu` 镜像开始，安装 `openjdk` 并将其作为 Dockerfile 的一部分。
##1.2 使用多阶段构建
例如，可以使用 maven 镜像来构建 Java 应用程序，然后重置 tomcat 镜像并将 Java 文件复制到正确的位置以部署应用程序，在同一个 Dockerfile 中完成这些事情。这意味着最终的镜像不包含构建时引入的所有库和依赖，只包含项目文件和运行时所需的环境。

如果需要使用 Docker 的不包含多阶段构建的版本，可以尝试减少 Dockerfile 文件中独立的 `RUN` 命令来减少镜像的层数。可以通过将多个命令整合到一个 `RUN` 行中并且使用 shell 的机制将这些命令组合到一起来实现镜像层数的减少。思考以下两个片段。第一个在镜像中创建两个层，而第二个创建一个层。
```
RUN apt-get -y update
RUN apt-get install -y python
```
```
RUN apt-get -y update && apt-get install -y python
```
##1.3 创建合适的基础镜像
如果你有多个镜像，这些镜像又有很大部分相同，可以用公共部分 [创建自己的基础镜像](https://docs.docker.com/develop/develop-images/baseimages/)，每个独立镜像都可以以此为基础创建。
Docker 只需要加载一次公共层，然后缓存起来。这意味着你的衍生镜像更有效地使用 Docker 主机上的内存并更快地加载。
##1.4 生产镜像精简
要保持生产镜像精简且允许调试，可以考虑使用生产镜像作为调试镜像的基本镜像。可以在生产镜像上添加其他测试或调试工具。
##1.5 所有镜像打标记
构建镜像时，始终添加有意义的标记来编码版本信息、用途（`prod` 或 `test`）、稳定性 stability 或其他在不同环境部署应用程序时有用的信息。不要依赖自动创建的 `latest` 标记。
#2. 应用数据持久化
- 避免使用 [存储驱动程序](https://docs.docker.com/engine/userguide/storagedriver/) 将应用数据存储在容器的可写层中。这增加了容器的大小，并且从 I/O 角度来看，与使用卷或绑定挂载 bind mounts 相比效率较低。

- 相反，用卷 [volume](https://docs.docker.com/engine/admin/volumes/volumes/) 来存储数据。

- 适合使用 [绑定挂载](https://docs.docker.com/engine/admin/volumes/bind-mounts/) 的一种情况是在开发过程中，可能需要挂载源目录或刚刚构建到容器中的二进制文件。在生产环境中，使用卷来代替绑定挂载，挂载到开发环境通过绑定挂载挂载到的相同位置。

- 在生产环境中，存储服务使用的敏感应用程序数据时使用 [secrets](https://docs.docker.com/engine/swarm/secrets/)，对于配置文件这样的非敏感数据使用 [configs](https://docs.docker.com/engine/swarm/configs/) 。如果你当前使用独立容器，请考虑迁移以使用单一副本服务（single-replica services），以便您可以利用这些仅限于服务的功能。
#3. 尽量使用 swarm service
- 可能的话，使用包含应用伸缩能力的 swarm service 来设计应用程序。
- 即使只需要运行应用程序的唯一一个实例，swarm service 相比独立容器也提供了更多优点。service 的配置是声明式的，Docker 一直尽量使期望的和实际的状态保持同步。
- 网络和卷可以从 swarm service 连接和断开，并且 Docker 可以以不中断的方式重新部署各个服务容器。配置变更时独立容器需要人工停止、删除和重新创建。
- 部分特征只有 service 可以使用，比如存储 secrets 和 configs 的能力。这些特征可以让你的镜像尽可能通用，防止在 Docker 镜像和容器中存储敏感数据。
- 通过 `docker stack deploy` 命令来处理镜像的获取，而不要使用 `docker pull` 命令，因为通过后者部署时不会尝试从其他已经下载镜像的节点获取镜像。当新节点添加到 swarm 时，镜像会自动获取。

swarm 服务中的节点共享数据时有限制。如果你使用 [Docker for AWS](https://docs.docker.com/docker-for-aws/persistent-data-volumes/) 或 [Docker for Azure](https://docs.docker.com/develop/docker-for-azure/persistent-data-volumes/)，可以使用 Cloudstor 插件在 swarm 服务的节点中共享数据。也可以把应用数据写入一个独立的支持同时更新的数据库中。
#4. 测试和部署时使用 CI/CD
- 在检查对源代码的更改或创建拉取请求时，使用 [Docker Cloud](https://docs.docker.com/docker-cloud/builds/automated-build/) 或其他 CI/CD 管道来自动构建并标记 Docker 镜像并对其进行测试。Docker Cloud 也可以将测试过的应用直接部署到生产环境中。

- 通过 [Docker EE](https://docs.docker.com/enterprise/) 可以要求开发，测试和安全团队在将镜像部署到生产环境中之前分别对其进行签名。通过这种方式，可以确保镜像在部署到生产环境之前已经通过了开发，质量和安全团队的测试和签名。
#5. 开发环境和生产环境的区别
|Development| Production|
|-|-|
|通过绑定挂载 bind mounts 使容器访问源代码|使用卷 volume 来存储容器数据
|使用 Mac 或 Windows 版本的 Docker |如果可能的话，使用Docker EE，通过用户映射将Docker进程与主进程隔离开来
|不用担心时间漂移|始终在 Docker 主机上和每个容器进程中运行 NTP 客户端，全部同步到同一个 NTP 服务器。如果使用 swarm 服务，还要确保每个 Docker 节点将其时钟与容器同步到同一时间源