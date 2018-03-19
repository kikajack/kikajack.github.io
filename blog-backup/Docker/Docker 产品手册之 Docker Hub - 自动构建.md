[原文地址](https://docs.docker.com/docker-hub/builds/)

可以从存储在仓库中的构建上下文中自动构建图像。构建上下文是 Dockerfile 和特定位置的任何文件。对于自动构建，构建上下文是包含 Dockerfile 的存储库。

自动构建有几个优点：

- 以这种方式构建的镜像完全符合期望
- 可以访问 Docker Hub 仓库的任何人都可以使用 Dockerfile。
- 代码变化后仓库会自动更新。

GitHub 和 Bitbucket 上的公共和私人仓库都支持自动构建。本文档将指导你完成使用自动构建的过程。
# 1. 先决条件
要使用自动构建，必须在 Docker Hub 和托管仓库（GitHub 或 Bitbucket）上拥有一个帐户。如果之前已链接过你的 Github 或 Bitbucket 帐户，则必须选择公共和私有连接类型。

要查看你当前的连接设置，请登录到 Docker Hub 并选择 **Profile > Settings > Linked Accounts & Services**。
# 2. 限制
- 目前 Docker Hub 不支持 Git LFS（Large File Storage，大文件存储）。如果你的构建上下文中有由 Git LFS 管理的二进制文件，则在自动构建过程中创建的副本中只有大文件对应的指针文件，这并不是你想要的。
订阅 [GitHub issue](https://github.com/docker/hub-feedback/issues/500) 来跟进此限制。
- 不支持构建 Windows 容器
# 3. 链接到托管仓库服务
#### 1. 登录 Docker Hub
#### 2. 依次点击菜单 Profile > Settings > Linked Accounts & Services
#### 3. 选择要链接到的服务
系统会提示你选择“Public”、“Private”和“Limited Access”。如果要使用自动构建，则需要 Public 和 Private 连接类型。

#### 4. 点击 Public 和 Private 连接类型下的 Select
系统会提示你输入服务凭证（Bitbucket 或 GitHub）进行登录。例如，Bitbucket 的提示符如下所示：

![bitbucket_creds](http://img.blog.csdn.net/20180319204305428?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

在授予代码存储库的访问权限之后，系统会跳转回 Docker Hub。链接建立完成。

![linked-acct](http://img.blog.csdn.net/20180319204330229?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
# 4. 创建自动构建
自动构建仓库依靠与代码仓库的集成来构建。不过，也可以使用 `docker push` 命令将已构建的镜像推送到这些仓库。
#### 1. 在 Docker Hub 中选择 Create > Create Automated Build
系统会提示用户/组织和代码仓库列表。
#### 2. 从用户/组织中选择
#### 3. 或者，键入以过滤仓库列表
#### 4. 选择要构建的项目
系统会显示 Create Automated Build 对话框。

![create-dialog1](http://img.blog.csdn.net/20180319204400888?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

该对话框会为可以自定义的值设置默认值。默认情况下，Docker 会为仓库中的每个分支构建镜像。它假定 Dockerfile 位于源代码的根目录。建立镜像时，Docker 用分支名称来标记镜像。
#### 5. 通过点击 Click here to customize 来自定义自动构建

![create-dialog](http://img.blog.csdn.net/20180319204438572?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

指定要从哪个代码分支或代码标签构建。可以通过单击 `+`（加号）来添加新的配置。该对话框接受正则表达式。

![regex-help](http://img.blog.csdn.net/20180319204503535?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#### 6. 点击 Create
系统显示你的自动构建首页。

![home-page](http://img.blog.csdn.net/20180319204527404?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

在 GitHub 中，Docker 集成将出现在你的**项目仓库**的 **Settings > Webhooks＆services** 页面中。

![docker-integration](http://img.blog.csdn.net/20180319204547524?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**附注：目前 GitHub 已经改版，截图如下。**
![GitHub 截图](http://img.blog.csdn.net/20180319224407337?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
对于该代码存储库，类似的页面会出现在 Bitbucket 中。**删除 Docker 集成会导致你的自动构建停止。**
## 4.1 理解构建过程
首次创建自动构建时，Docker Hub 会构建你的镜像。几分钟后，你应该可以在 image dashboard 上看到你的新构建。Build Details 页面显示构建系统的日志：

![first_pending](http://img.blog.csdn.net/20180319204621398?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

在构建过程中，Docker 会将 Dockerfile 的内容复制到 Docker Hub。Docker 社区（用于公共仓库）或批准的团队成员/组织（用于私人仓库）可以在你的仓库页面上查看 Dockerfile。

构建过程会在与 Dockerfile 相同的目录中查找 `README.md`。如果你的仓库中有一个 `README.md` 文件，它将在仓库中用作 full description（完整描述）。如果在构建之后更改 full description，则在下次运行自动构建时将被覆盖。要防止覆盖，请修改 Git 仓库中的 `README.md`。

一次只能触发一次构建，每五分钟不超过一次。如果你已经有一个构建挂起，或者你最近提交了一个构建请求，Docker 会忽略新的请求。
## 4.2 构建状态解释
通过查看 Build Details 页面，可以查看特定仓库的构建状态。如果有正在排队或正在进行的构建，则可以单击 Cancel 来取消构建。

![build-states-ex](http://img.blog.csdn.net/20180319204654856?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

状态有：

- `Queued`：排队等待构建镜像。队列时间取决于你可以使用的并发构建数量。
- `Building`：正在构建镜像。
- `Success`：镜像构建成功，没有问题。
- `Error`: 镜像出了问题。点击对应行进入 Builds Details 页面。页面顶部的 banner 显示日志文件的最后一句话，指明错误是什么。如果需要更多信息，滚动至屏幕底部的日志部分。
# 5. 使用 Build Settings 页面
“Build Settings”页面允许你管理现有的自动构建配置并添加新配置。默认情况下，将新代码合并到源代码库时，会触发 DockerHub 镜像的构建。

![merge_builds](http://img.blog.csdn.net/20180319204726678?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

清除 checkbox 可以关闭这个特性。可以使用这个页面的其他设置来配置并构建镜像。
# 6. 增加并运行新构建
Build 对话框的顶部是配置好的构建列表。可以从代码分支或构建标签来构建。

![build-by](http://img.blog.csdn.net/20180319204754360?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

每当向代码仓库进行 push 时，Docker 都会对列出的所有内容进行构建。如果指定了分支或标记，则可以通过按下“Trigger”来手动构建该镜像。如果使用正则表达式语法（正则表达式）来定义构建分支或标记，Docker 不会提供手动构建的选项。可以按照下面步骤添加新的构建：

#### 1. 点击加号 `+`
#### 2. 选择类型
可以通过代码分支或镜像标签来构建。
#### 3. 输入分支或标签的名字
可以输入特定值或使用正则表达式来选择多个值。要查看正则表达式的示例，请按页面右侧的“Show More”链接。

![regex-help (1)](http://img.blog.csdn.net/20180319204817860?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 4. 输入 Dockerfile 的位置
#### 5. 指定 Tag 名
#### 6. 点击“Save Changes”
如果有地方弄错了或想删除构建，点击减号 `-` 然后点击“Save Changes”即可。
# 7. 仓库链接
仓库链接可将一个自动构建链接到另一个自动构建。如果一个自动构建得到更新，Docker 会触发另一个构建。这可以很容易地确保相关镜像保持同步。可以链接多个镜像存储库。**只需链接两个相关版本的一侧，双方都链接导致无尽的构建循环。**

按照下列步骤创建链接：

#### 1. 在 Docker Hub 中进入要自动构建的仓库的 Build Settings 页面
#### 2. 在 Repository Links 部分，输入镜像仓库名
远程仓库名应该是官方仓库名（如 `ubuntu`）或公共仓库名 `namespace/repoName`。
#### 3. 点击 Add

![repo_links](http://img.blog.csdn.net/20180319204851002?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
# 8. 远程构建触发器
要以编程方式触发自动构建，可以在另一个应用程序（GitHub 或 Bitbucket）中设置远程构建触发器。当激活自动构建的构建触发器时，它会为你提供一个 Token 和一个 URL。

![build-trigger](http://img.blog.csdn.net/20180319204912488?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以通过 curl 来触发构建：
```
$ curl --data build=true -X POST https://registry.hub.docker.com/u/svendowideit/testhook/trigger/be579c
82-7c0e-11e4-81c4-0242ac110020/
OK
```
要验证一切都工作正常，可以查看当前页面的“Last 10 Trigger Logs”。
