[原文地址](https://docs.docker.com/engine/security/trust/deploying_notary/)

部署 Notary 服务器的最简单方法是使用 Docker Compose。要按照此页面上的过程，你必须已经安装了 Docker Compose。
## Clone Notary 仓库
```
git clone git@github.com:docker/notary.git
```
## 用简单证书构建并启动 Notary 服务器
```
docker-compose up -d
```
有关如何部署 Notary 服务器的详细文档，请参阅 [运行 Notary 服务的说明](https://docs.docker.com/notary/running_a_service/) 以及 [Notary 仓库](https://github.com/docker/notary) 以获取更多信息。
## 在尝试与 Notary 服务器交互之前，确保 Docker 或 Notary 客户端信任 Notary 服务器的证书

可以根据你使用的 CLI 参考 [Docker](https://docs.docker.com/engine/reference/commandline/cli/#notary) 或 [Notary](https://github.com/docker/notary#using-notary) 手册。
#### 如果你想在生产环境中使用 Notary
在 Notary 服务器有正式的稳定版后，请回到这里查看说明。要在生产中部署 Notary，请参阅 [Notary 仓库](https://github.com/docker/notary)。
