[原文地址](https://docs.docker.com/registry/introduction/)

registry 是一个存储和内容交付系统，拥有经过命名的、可以使用不同标签版本的 Docker 镜像。

>示例：`distribution/registry` 镜像，标签为 `2.0` 和 `2.1`。

用户通过使用 docker push 和 pull 命令与 registry 进行交互。

>例如：`docker pull registry-1.docker.io/distribution/registry:2.1`。

存储本身被委托给驱动程序。默认存储驱动程序是本地的 posix 文件系统，适用于开发环境或小范围部署。还支持其他基于云的存储驱动程序，如 S3，Microsoft Azure，OpenStack Swift 和 Aliyun OSS。如果希望使用其他存储后端，可以通过编写自己的驱动程序来实现 [Storage API](https://docs.docker.com/registry/storage-drivers/)。

由于安全访问托管镜像至关重要，因此 Registry 原生支持 TLS 和基本身份验证。

Registry GitHub 仓库包含有关高级身份验证和授权方法的其他信息。预计只有非常大的公共部署才能以这种方式扩展注册表。

最后，Registry 系统提供了强大的通知系统，针对活动调用 webhooks，以及广泛的日志记录和报告功能，这些功能主要用于需要收集指标的大型安装（mostly useful for large installations that want to collect metrics）。
# 1. 理解镜像名称
典型的 docker 命令中使用的镜像名称反映了它们的来源：

- `docker pull ubuntu` 指示 docker 从官方的 Docker Hub 中取出一个名为 `ubuntu` 的镜像。这只是 `docker pull docker.io/library/ubuntu` 命令的缩写
- `docker pull myregistrydomain:port/foo/bar` 指示 docker 访问位于 `myregistrydomain:port` 的 registry 以获取 `foo/bar` 镜像

可以在 [官方 Docker 引擎文档](https://docs.docker.com/engine/reference/commandline/cli/) 中找到更多关于处理镜像的各种 Docker 命令。
# 2. 使用案例
运行你自己的 registry 是与 CI/CD 系统集成和互补的绝佳解决方案。在典型的工作流程中，版本控制系统中的源代码提交时会触发 CI 系统上的构建，如果构建成功，则系统会将新镜像推送到 registry。然后，来自 registry 的通知将触发在暂存环境中的部署，或者通知其他系统有新镜像可用。

如果想要在大型集群上快速部署新镜像，它也是一个重要组件。

最后，这是在隔离网络内分发镜像的最佳方式。
# 3. 需要
必须要熟悉 Docker，特别是上传及下载镜像。必须了解守护进程和 cli 之间的区别，并且至少掌握关于网络的基本概念。

只是启动 registry 是很容易的，但在生产环境中操作它需要操作技能，就像任何其他服务一样。需要熟悉系统可用性和可伸缩性，日志记录和日志处理，系统监控和安全。最好对 http 和网络通信、 golang 或黑客知识有一定的理解。
