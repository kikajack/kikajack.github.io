[原文地址](https://docs.docker.com/docker-hub/github/)

如果你已经将 Docker Hub 链接到了你的 GitHub 账户，直接跳到第五小节 创建自动构建。
# 1. 将 Docker Hub 链接到 GitHub 账户
>注意：因为 Docker Hub 需要设置 GitHub 的 service hook，自动构建需要读写权限。我们别无选择，这就是 GitHub 管理权限的方式。我们保证不会碰你帐户中的任何其他内容。

要为 GitHub 中的仓库设置自动构建，需要将 [Docker Hub](https://hub.docker.com/account/authorized-services/) 链接到你的 GitHub 账户。这将允许 registry 查看你的 GitHub 仓库。

要增加、删除或查看已经链接的账户，请转至 Hub 配置文件“Settings”的“Linked Accounts & Services”部分。

![authorized-services](//img-blog.csdn.net/20180320210632819?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2tpa2FqYWNr/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

链接时，选择“Public and Private”或“Limited Access”链接。

![add-authorized-github-service](//img-blog.csdn.net/20180320210656739?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2tpa2FqYWNr/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

“Public and Private”选项最容易使用，这会授予 Docker Hub 完整的访问你的仓库的权限。GitHub 也允许你将属于你所在的 GitHub 组织的仓库授权。

选择“Public and Private”选项时，Docker Hub 只会得到访问公开数据和公开仓库的权限。

按照屏幕上的说明授权并将你的 GitHub 帐户链接到 Docker Hub。链接后，可以选择要从中创建自动构建的源仓库。

可以通过访问 GitHub 用户的“Applications”设置来查看和撤销 Docker Hub 的访问权限。

>注意：如果删除用于其中一个自动构建仓库的 GitHub 帐户链接，先前构建的镜像仍可用。如果你稍后重新链接到该 GitHub 帐户，则可以使用 Hub 上的“Start Build”按钮启动自动构建，或者如果 GitHub 仓库上的 webhook 仍然存在，任何后续提交都会触发自动构建。
# 2. 自动构建和限制链接的 GitHub 帐户
如果你选择仅将 GitHub 帐户与“Limited Access”链接关联，那么在创建自动构建之后，需要使用“Start a Build”按钮手动触发 Docker Hub 构建，或者手动添加 GitHub webhook， 如 GitHub Service Hooks 中所述。这仅适用于用户帐户下的仓库，无法使用“Limited Access”链接向公共的 GitHub 组织添加自动构建。
# 3. 改变 GitHub 用户链接
如果你想要删除或更改 GitHub 帐户和 Docker Hub 之间的链接级别，则需要在两个位置执行此操作。

首先，从 Docker Hub 的“Settings”中删除“Linked Account”。然后转到 GitHub 帐户的个人设置，然后在“Applications”部分中的“Revoke access”。

现在可以随时重新链接账户。
# 4. GitHub 组织
GitHub 组织和由组织 fork 的私有仓库可用于使用“Docker Hub Registry”应用程序自动构建，该应用程序需要添加到组织中，然后应用于所有用户。

要检查或请求访问，请转到你的 GitHub 用户的“Setting”页面，从左侧栏中选择“Applications”部分，然后单击“View”按钮以查看“Docker Hub Registry”。

![gh-check-user-org-dh-app-access](//img-blog.csdn.net/2018032021075512?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2tpa2FqYWNr/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

组织的管理员可能需要在“Settings”中转到组织的“Third party access”页面来授权或禁止访问 Docker Hub Registry 应用程序。此更改会用于所有的组织成员。

![gh-check-admin-org-dh-app-access](//img-blog.csdn.net/20180320210815373?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2tpa2FqYWNr/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

可以使用 GitHub 的“People and Teams”界面管理对特定用户和 GitHub 仓库更具体的访问控制。
# 5. 创建自动构建
可以从任何具有 Dockerfile 的公有或私有 GitHub 仓库创建自动构建。

一旦选择了源代码库，你可以配置：

- 仓库构建的 Hub 用户/组织命名空间 - 你的 Docker ID 名称或帐户所在的任何组织的名称
- 镜像构建的 Docker 仓库名称
- 仓库描述
- 如果 Docker 仓库的可见性：“Public”或“Private”你可以在创建仓库后更改可访问性选项。如果你将 Private 仓库添加到 Hub 用户命名空间，那么你只能将其他用户添加为协作者，并且这些用户可以查看并提取该仓库中的所有镜像。要配置更细化的访问权限（例如使用用户团队或允许不同用户访问不同的镜像标签），则需要将私有仓库添加到你有管理员权限的 Hub 组织。
- 当提交被推送到 GitHub 仓库时，启用或禁用重新构建 Docker 镜像。

你也可以选择一个或多个：

- git 分支/标签，
- 用作上下文的仓库子目录，
- Docker 镜像标签名称

可以通过单击仓库视图的“Description”部分来修改仓库的描述。当下一个构建被触发时，“Full Description”被 README.md 文件覆盖。
# 6. GitHub 私有子模块（private submodules）
如果你的 GitHub 仓库包含指向私有子模块的链接，则构建失败。

通常，Docker Hub 会在你的 GitHub 仓库中设置一个部署密钥。不幸的是，GitHub 只允许一个仓库部署密钥访问一个仓库。

要解决此问题，可以在 GitHub 中创建一个专用用户帐户，并附加帐户的自动构建部署密钥。这个专用的构建帐户可以限制为只需访问构建所需的仓库。
#### 1. 首先，在 GitHub 中创建新帐户。它应该被授予对主仓库和所有需要的子模块的只读访问权限。
![gh_org_members](//img-blog.csdn.net/20180320210952110?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2tpa2FqYWNr/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
#### 2. 这可以通过将帐户添加到保存主 GitHub 仓库和所有子模块仓库的组织中的只读团队来完成。
![gh_team_members](//img-blog.csdn.net/20180320211021912?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2tpa2FqYWNr/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
#### 3. 接下来，从主 GitHub 仓库中删除部署密钥。这可以在 GitHub 仓库的“Deploy keys”设置部分完成。
![gh_repo_deploy_key](//img-blog.csdn.net/20180320211043127?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2tpa2FqYWNr/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
#### 4. 自动构建的部署密钥位于“Deploy keys”下的“Build Details”菜单中。
![deploy_key](//img-blog.csdn.net/20180320211106932?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2tpa2FqYWNr/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
#### 5. 在你的专用 GitHub 用户帐户中，从 Docker Hub 自动构建中添加部署密钥。
![gh_add_ssh_user_key](//img-blog.csdn.net/20180320211130779?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2tpa2FqYWNr/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
# 7. GitHub 服务钩子（service hooks）
GitHub 服务钩子允许 GitHub 在某个指定的 git 仓库发送提交操作时通知 Docker Hub。

当从 GitHub 创建一个具有完全“Public and Private”链接的自动构建时，Service Hook 应该会自动添加到你的 GitHub 仓库中。

如果你的 GitHub 帐户到 Docker Hub 的链接是“Limited Access”，那么需要手动添加服务钩子。

要添加、确认或修改服务钩子，请登录到 GitHub，然后导航到仓库，单击“Settings”（齿轮），然后选择“Webhooks & Services”。你必须拥有管理员权限才能查看或修改此设置。

下图显示了“Docker”服务钩子。

![github-side-hook](//img-blog.csdn.net/20180320211152708?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2tpa2FqYWNr/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

如果手动添加“Docker”服务，请确保选中“Active”复选框，然后单击“Update service”按钮以保存更改。
