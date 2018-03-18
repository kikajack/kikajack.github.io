[原文地址](https://docs.docker.com/docker-hub/orgs/)

Docker Hub [organizations](https://hub.docker.com/organizations/) 允许创建团队，以便你可以让同事访问共享镜像仓库。Docker Hub organizations 可以像用户帐户一样包含公共和私人仓库。通过定义用户团队然后将团队权限分配给特定仓库来分配对这些仓库的推送或拉取的访问权限。仓库创建仅限于 organization 所有者组中的用户。这允许你分发访问受限的 Docker 镜像，并选择哪些 Docker Hub 用户可以发布新镜像。
# 1. 创建并查看组织 organization
可以查看你属于哪个组织，并通过点击顶部导航栏的 **Organizations** 来添加新组织。

![orgs](//img-blog.csdn.net/20180318192137571?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2tpa2FqYWNr/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
## 1.1 Organization teams
一个组织中的“Owners”团队中的用户可以创建和修改所有团队的成员。

其他用户只能看到他们所属的团队。

![groups](//img-blog.csdn.net/20180318192207781?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2tpa2FqYWNr/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
## 1.2 仓库团队权限
使用团队来管理可以与仓库交互的人员。

你需要成为组织的“Owners”团队的成员才能创建新团队、Hub 仓库或进行自动构建。作为“Owners”，你随后使用仓库视图的“Collaborators”（协作者）部分将以下仓库访问权限委派给团队。

权限是累积的。例如，如果拥有“写入”权限，则会自动拥有“读取”权限：

- 读权限允许用户以与公共仓库相同的方式查看，搜索和获取私人存储库。
- 写权限允许用户推送到 Docker Hub 上的非自动仓库。
- 管理员权限允许用户修改存储库“Description”，“Collaborators”协作者权限，“Public/Private”可见性和“Delete”。

>注意：尚未验证其电子邮件地址的用户只有对存储库的读取权限，而不管其团队成员给予他们的权利。

![org-repo-collaborators](//img-blog.csdn.net/20180318192257558?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2tpa2FqYWNr/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
