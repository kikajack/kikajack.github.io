[原文地址](https://docs.docker.com/develop/develop-images/image_management/)

要使你的镜像可以被其他人使用，最简单的办法是使用 Docker Registry，比如 Docker Hub、Docker Trusted Registry，或者运行你自己的私有 registry。
#1. Docker Hub
[Docker Hub](https://docs.docker.com/docker-hub/) 是一个由 Docker 公司管理的公共的注册处。它集中了有关组织、用户帐户和镜像的信息。它包括 Web UI，使用组织的身份验证和授权，使用类似  `docker login`、`docker pull` 和 `docker push` 这些命令的 CLI 和 API 访问，评论，点赞，搜索等。Docker Hub 也集成到 [Docker Store](https://docs.docker.com/docker-store/) 中，Docker Store 是一个允许购买和销售非免费镜像的市场。
#2. Docker Registry
Docker Registry 是 Docker 生态系统的一个组件。registry 是一个存储和内容分发系统，可以保存命名过的 Docker 镜像，通过版本标记获取。例如，镜像 `distribution/registry` 可以包含标记 `2.0` 和 `latest`。用户通过使用 docker push 和 docker pull 命令来和 registry 交互：`docker pull myregistry.com/stevvooe/batman:voice`。

Docker Hub 是 Docker Registry 的一个实例。
#3. Docker Trusted Registry
[Docker Trusted Registry](https://docs.docker.com/datacenter/dtr/2.1/guides/) 是 Docker Enterprise Edition 的一部分，是一个私有的、安全的 Docker Registry，包含了镜像签名和内容可信（image signing and content trust）、基于角色的访问控制（role-based access controls）和其他企业级特性。
#4. Content Trust
在联网系统间传输数据时，信任是一个核心问题。特别是，当通过互联网等不受信任的媒介进行通信时，确保系统操作的所有数据的完整性和发布者至关重要。使用 Docker 将镜像（数据）上传到 registry 或从 registry 下载。Content Trust 使得能够验证通过任何渠道从 registry 接收的所有数据的完整性和发布者。


查看 [Content trust](https://docs.docker.com/engine/security/trust/) 了解关于在客户端配置和使用这个特性的信息。