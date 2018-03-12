[原文地址](https://jenkins.io/doc/book/pipeline/multibranch/)

之前的章节中已经实现了 Jenkinsfile，并可以将代码上传到源代码控制。这一章节讲述基于 Jenkinsfile 的多分支 Pipeline 的概念，以便在 Jenkins 中提供更多动态和自动功能。
#1. 创建多分支 Pipeline
##1.1 步骤
多分支 Pipeline 项目类型使你能够为同一项目的不同分支使用不同的 Jenkins 文件。在多分支 Pipeline 项目中，Jenkins 可以为源代码控制中包含 Jenkinsfile 的分支自动发现，管理和执行 Pipeline。

这消除了手动 Pipeline 创建和管理的需要。

要创建多分钟 Pipeline：

- 点击 Jenkins 首页的 **New Item**

![classic-ui-left-column](http://img.blog.csdn.net/20180304173043488?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 输入你的 Pipeline 名字，选择 **Multibranch Pipeline** 并点击 **OK**
注意：Jenkins 使用 Pipeline 名字在磁盘上创建目录，注意不要使用空格等非法字符。

![new-item-multibranch-creation](http://img.blog.csdn.net/20180304173321693?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 添加 **Branch Source**（例如 Git）并输入仓库地址

![multibranch-branch-source](http://img.blog.csdn.net/20180304173428677?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![multibranch-branch-source-configuration](http://img.blog.csdn.net/20180304173454367?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- Save the Multibranch Pipeline project.

保存后，Jenkins 会自动扫描指定的存储库，并为存储库中包含 Jenkinsfile 的每个分支创建适当的项目。

默认情况下，仓库的分支增加或删除时，Jenkins 不会自动重新编制索引（除非使用组织文件夹 Organization Folder），因此配置多分支 Pipeline 以定期在配置中重新索引通常很有用：

![multibranch-branch-indexing](http://img.blog.csdn.net/20180304174111199?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
##1.2 其它环境变量
多分支 Pipeline 通过 `env` 全局变量暴露有关正在构建的分支的其他信息，例如：

- **BRANCH_NAME**：Pipeline 正在执行的分支的名字，例如 `master`。
- **CHANGE_ID**：与某种改变请求相对应的标识符，例如拉取请求的编号。

其他环境变量在 [全局变量参考](http://blog.csdn.net/kikajack/article/details/79428889#t18) 中列出。
##1.3 支持 Pull 请求
通过“GitHub”或“Bitbucket”分支源代码，多分支 Pipeline 可用于验证拉/更改（pull/change）请求。该功能分别由 [GitHub Branch Source](https://plugins.jenkins.io/github-branch-source) 和 [Bitbucket Branch Source](https://plugins.jenkins.io/cloudbees-bitbucket-branch-source) 插件提供。有关如何使用这些插件的更多信息，请参阅他们的文档。
#2. 使用 Organization Folders
通过组织文件夹，Jenkins 可以监视整个 GitHub Organization 或 Bitbucket Team/Project，并为包含分支和包含 Jenkinsfile 的请求的存储库自动创建新的多分支管道。

目前，此功能仅适用于 GitHub 和 Bitbucket，其功能由 GitHub Organization Folder 和 Bitbucket Branch Source 插件提供。