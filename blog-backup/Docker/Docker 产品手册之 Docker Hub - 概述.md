[原文地址](https://docs.docker.com/docker-hub/)

[Docker Hub](https://hub.docker.com/) 是一个基于云的 registry（注册表）服务，它允许你链接到代码仓库，构建镜像并测试它们，存储手动推送的镜像以及到 [Docker Cloud](https://docs.docker.com/docker-cloud/) 的链接，以便将镜像部署到主机。它为整个开发流程中的容器镜像发现，分发和变更管理，[用户和团队协作](https://docs.docker.com/docker-hub/orgs/) 以及工作流程自动化提供集中资源。

可以免费通过 [Docker ID](https://docs.docker.com/docker-hub/accounts/) 登录 Docker Hub 和 Docker Cloud。

![getting-started](http://img.blog.csdn.net/20180318183614749?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

Docker Hub 主要提供以下特性：

- [镜像仓库](https://docs.docker.com/docker-hub/repos/)：从社区和官方仓库查找和获取镜像，并从你有权访问的私有镜像库中管理、推送、提取镜像。
- [自动构建](https://docs.docker.com/docker-hub/builds/)：更改源代码仓库时自动创建新镜像。
- [Webhooks](https://docs.docker.com/docker-hub/webhooks/)：自动构建的一个特性，Webhook 让你在成功推送到仓库后触发行为。
- [组织](https://docs.docker.com/docker-hub/orgs/)：创建工作组来管理对镜像仓库的访问。
- GitHub 和 Bitbucket 集成：将 Hub 和 Docker 镜像添加到当前的工作流程中。
# 1. 创建 Docker ID
要探索 Docker Hub，你需要按照 [你的 Docker ID](https://docs.docker.com/docker-hub/accounts/) 中的说明创建一个帐户。

>注意：您可以在不登录的情况下从 Hub 搜索并获取 Docker 镜像，但要 push 镜像则必须登录。

你的 Docker ID 为你提供一个免费的私人 Docker Hub 仓库。如果需要更多私人仓库，则可以从免费帐户升级到付费计划。要了解更多信息，请登录到 Docker Hub 并转到“设置”菜单中的“计费和计划”（[ Billing & Plans](https://hub.docker.com/account/billing-plans/)）。
## 1.1 探索仓库
可以通过两种方式从 Docker Hub 中找到公共存储库和映像。可以从 Docker Hub 网站“搜索”，或者使用 Docker 命令行工具来运行 `docker search` 命令。例如，如果你正在查找 Ubuntu 镜像，则可以运行以下命令行搜索：
```
    $ docker search ubuntu
```
两种方法都列出了 Docker Hub 上与搜索条件匹配的可用公共仓库。

私有仓库不会出现在搜索结果中。要查看你可以访问的所有仓库及其状态，请查看 [Docker Hub](https://hub.docker.com/) 上的“Dashboard”页面。
## 1.2 使用官方仓库
Docker Hub 包含许多 [官方仓库](http://hub.docker.com/explore/)。这些是来自供应商和 Docker 贡献者的公共认证仓库。它们包含 Canonical，Oracle 和 Red Hat 等供应商提供的 Docker 镜像，可以将其用作构建应用程序和服务的基础。

使用官方仓库时，就是使用由专家为你的应用程序构建的优化过的并且是最新的镜像。

>注意：如果你想为你的组织或产品贡献官方存储库，请参阅 [Docker Hub 上的官方存储库文档](https://hub.docker.com/docker-hub/official_repos/) 以获取更多信息。
# 2. 通过 Docker Hub 镜像仓库工作
Docker Hub 为你和你的团队提供了一个构建和发布 Docker 镜像的场所。

可以通过两种方式配置 Docker Hub 仓库：

- [仓库](https://hub.docker.com/docker-hub/repos/)，允许你从本地 Docker 守护进程上传镜像。
- [自动构建](https://hub.docker.com/docker-hub/builds/)，链接到源代码仓库，并且在每次检测到源代码的变化时触发 Docker Hub 上的镜像的重新构建过程。

你可以创建可以被任何其他 Hub 用户访问的公共仓库，也可以创建由你来控制的私有仓库。
## 2.1 Docker 命令和 Docker Hub
Docker 本身通过 `docker search`，`pull`，`login` 和 `push` 命令提供对 Docker Hub 服务的访问。
