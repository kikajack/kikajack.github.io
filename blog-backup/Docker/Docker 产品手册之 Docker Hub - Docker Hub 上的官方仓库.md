[原文地址](https://docs.docker.com/docker-hub/official_repos/)

Docker [官方仓库](https://hub.docker.com/official/) 是托管在 Docker Hub 上的一系列 Docker 仓库。它们旨在：

- 提供必要的基础操作系统仓库（例如，[ubuntu](https://hub.docker.com/_/ubuntu/)，[centos](https://hub.docker.com/_/centos/)），作为大多数用户的起点。
- 为流行的编程语言运行时，数据存储和其他服务提供插件解决方案，类似于平台即服务（PAAS）所提供的解决方案。
- 举例说明 Dockerfile 的最佳实践，并提供明确的文档以作为其他 Dockerfile 作者的参考。
- 确保及时进行安全更新。这一点特别重要，因为许多官方仓库都是 Docker Hub 上最受欢迎的。

Docker 公司有专门的团队负责审查和发布官方仓库中的所有内容。该团队与上游软件维护人员，安全专家及更广泛的 Docker 社区合作。

虽然最好由上游软件作者维护其相应的官方仓库，但这不是一个严格的要求。为官方仓库创建和维护进行是一个公共过程。它在 GitHub 上公开发布，鼓励参与。任何人都可以提供反馈，提供代码，建议流程更改，甚至提出新的官方存储库。
# 1. 我该使用官方仓库吗？
建议 Docker 新用户在项目中使用官方仓库。这些仓库有清晰的文档，践行最佳实践，并为最通用的使用案例而设计。建议高级用户查看官方仓库作为 Dockerfile 学习过程的一部分。

不建议使用官方仓库的一个常见理由是优化镜像大小。例如，许多编程语言的堆栈镜像都包含完整的构建工具链，以支持依赖于优化过的代码的模块的安装。高级用户可以使用必要的预编译库来构建自定义镜像以节省空间。

许多语言堆栈如 [python](https://hub.docker.com/_/python/) 和 [ruby](https://hub.docker.com/_/ruby/) 都有 `-slim` 标签变体，用于满足优化需求。即使这些“slim”变体不满足要求，仍建议继承官方镜像中的基本操作系统镜像以利用正在进行的维护工作，而不是重复造轮子。
# 2. 我如何知道官方存储库是安全的？
官方仓库中的每个镜像都通过了 Docker Cloud 的安全扫描服务的扫描。这些安全扫描的结果提供了有关哪些镜像包含安全漏洞的宝贵信息，并允许你选择符合安全标准的镜像。

查看 Docker 安全扫描结果：

- 确保已登录到 Docker Hub。即使在退出时也可以查看官方镜像，但扫描结果仅在登录后才可用。
- 导航到要查看其安全扫描的官方仓库。
- 点击 `Tags` 页签查看标签列表及其安全扫描摘要。

![scan-drilldown](//img-blog.csdn.net/20180320223143951?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2tpa2FqYWNr/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

可以单击标签的详细信息页面，查看有关镜像中哪些层以及层中哪些组件易受攻击的详细信息。当单击单个易受攻击的组件时，会显示详细信息，包括指向该漏洞的官方 CVE 报告的链接。
# 3. 我如何参与？
所有官方仓库都在其文档中包含用户反馈部分，其中包含该特定仓库的详细信息。大多数情况下，包含用于官方存储库的 Dockerfiles 的 GitHub 存储库也具有活动问题跟踪器（active issue tracker）。一般反馈和支持问题应该直接发送到 Freenode IRC上的 `＃docker-library`。
# 4. 我如何创建一个新的官方仓库？
From a high level, an Official Repository starts out as a proposal in the form of a set of GitHub pull requests. Detailed and objective proposal requirements are documented in the following GitHub repositories:
从较高的层面来看，官方仓库是以一组 GitHub pull 请求的形式作为提案开始的。详细和客观的提案要求记录在以下 GitHub 存储库中：

- [docker-library/official-images](https://github.com/docker-library/official-images)
- [docker-library/docs](https://github.com/docker-library/docs)

官方仓库团队在社区贡献者的帮助下，正式审查每个提案并向作者提供反馈。在提案被接受之前，这个初步审核过程可能需要一些来回。

在审查过程中也有主观的考虑。这些主观问题归结为一个基本问题：“这个镜像通常有用吗？”例如，[python](https://hub.docker.com/_/python/) 官方仓库对 Python 开发者社区“普遍有用”，而上周用 Python 编写的一个晦涩的文本冒险游戏则不是。

一旦新提案被接受，作者负责保持其镜像的最新状态并回应用户的反馈意见。官方仓库团队负责在 Docker Hub 上发布镜像和文档。对官方仓库的更新遵循相同的拉取请求流程，尽管评论较少。官方仓库团队最终扮演所有变更的守门人的角色，这有助于缓解引入质量和安全问题的风险。
