[原文地址](https://docs.docker.com/docker-hub/repos/)

Docker Hub 仓库允许你与同事，客户或整个 Docker 社区共享镜像。如果你在内部构建镜像，无论是在你自己的 Docker 守护进程中还是使用自己的持续集成服务，都可以将它们推送到你添加到 Docker Hub 用户或组织帐户中的 Docker Hub 仓库。

或者，如果 Docker 镜像的源代码位于 GitHub 或 Bitbucket 上，则可以使用由 Docker Hub 服务构建的“Automated build”仓库。请参阅 [自动构建文档](https://docs.docker.com/docker-hub/builds/) 以了解这些服务提供的额外功能。

![repos](//img-blog.csdn.net/20180318214328553?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2tpa2FqYWNr/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
# 1. 搜索镜像
可以通过搜索界面或命令行界面搜索 Docker Hub registry。搜索可以通过镜像名称，用户名或描述来查找：
```
$ docker search centos
NAME                                 DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
centos                               The official build of CentOS.                   1034      [OK]
ansible/centos7-ansible              Ansible on Centos7                              43                   [OK]
tutum/centos                         Centos image with SSH access. For the root...   13                   [OK]
...
```
可以看到两个示例结果：`centos` 和 `ansible/centos7-ansible`。第二个结果显示它来自用户的公共仓库，名为 `ansible/`，而第一个结果 `centos` 没有明确列出仓库，这意味着它来自官方仓库的顶级命名空间。`/` 字符将用户的仓库与镜像名称分开。

一旦找到了想要的镜像，可以用 `docker pull <imagename>`下载：
```
$ docker pull centos
latest: Pulling from centos
6941bfcbbfca: Pull complete
41459f052977: Pull complete
fd44297e2ddb: Already exists
centos:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
Digest: sha256:d601d3b928eb2954653c59e65862aabb31edefa868bd5148a41fa45004c12288
Status: Downloaded newer image for centos:latest
```
现在你就有了一个可以用来运行容器的镜像。
# 2. 查看仓库标签
Docker Hub 的仓库“Tags”视图显示可用标签及相关镜像的大小。

镜像大小是镜像及其所有父镜像占用的累积空间。这也是在通过 `docker save` 保存镜像时创建的 Tar 文件所占的磁盘空间。

![busybox-image-tags](//img-blog.csdn.net/20180318214424322?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2tpa2FqYWNr/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
# 3. 在 Docker Hub 上创建新仓库
初次创建 Docker Hub 用户时，会看到“Get started with Docker Hub”页面，可用直接在页面上点击“Create Repository”。也可以从菜单“Create ▼”进入到“Create Repository”。

创建新仓库时，可以将其放在你的 Docker ID 命名空间中，或者你属于“Owners”团队的任何组织中。命名空间中的仓库名需要独一无二，且只能由小写字母、数字、中横线和下划线组成。

“Short Description”的长度在 100 个字符之内，用于搜索。“Full Description”用于仓库的 Readme 简介，支持 Markdown 语法。

在点击“Create”按钮之后，需要通过 `docker push` 命令将镜像传到基于该 Hub 的仓库。
# 4. 将仓库镜像上传到 Docker Hub
要将仓库上传到 Docker Hub，需要使用 Docker Hub 命名空间命名你的本地镜像和上一步创建的仓库名。通过指定特殊的 `:<tag>` 可以在一个仓库中添加多个镜像（例如 `docs/base:testing`）。如果没有指定，会使用 `latest` 这个默认的 tag。可以在构建镜像的同时通过 `docker build -t <hub-user>/<repo-name>[:<tag>]` 命名镜像，通过 `docker tag <existing-image> <hub-user>/<repo-name>[:<tag>]` 重新为已存在的镜像打标签，或者使用 `docker commit <exiting-container> <hub-user>/<repo-name>[:<tag>]` 来提交变化。

现在，可以将此仓库推送到由其名称或标记指定的 registry 中。
```
$ docker push <hub-user>/<repo-name>:<tag>
```
镜像会上传，之后你的团队或社区就可以使用了。
# 5. 星标 Stars
你的仓库可以被别人加星标，并且你也可以给仓库加星标。星标是表明你喜欢存储库的一种方式。它们也是为收藏夹添加书签的简单方法。
# 6. 注释 Comments
可以通过在仓库上留下评论来与 Docker 社区的其他成员和维护人员进行交互。如果发现任何不合适的评论，可以举报。
# 7. 协作者及其角色
协作者是得到你的授权，可以访问你的私有仓库的人。一旦指定，他们可以 push 并 pull 你的仓库。他们无法执行任何管理任务，例如删除存储库或将其状态从私有状态更改为公共状态。

>注意：协作者不能添加其他协作者。只有存储库的所有者具有管理访问权限。

还可以使用组织和团队在 Docker Hub 上分配更细致的协作者权限（“Read”，“Write”或“Admin”）。更多信息请参阅 [组织和团队](http://blog.csdn.net/kikajack/article/details/79604987)。
# 8. 私有仓库
私有仓库可以包含私有镜像，只有你自己的帐户或者指定的组织或团队成员才可以访问。

要在 Docker Hub 上使用私有仓库，需要先通过 [Add Repository](https://hub.docker.com/add/repository/) 按钮添加一个。可以通过 Docker Hub 用户帐户免费获得一个私人仓库（不适用于你所属的组织）。如果需要更多帐户，则可以升级 Docker Hub 计划。

创建私有存储库后，可以使用 Docker 获取或上传镜像。

>注意：需要登录并有权使用私人存储库。

私有仓库类似公共仓库。但是，无法在公共 registry 中浏览它们或搜索其内容，并且它们不像公共仓库那样被缓存。

可以指定协作者并从该仓库的“Settings”页面管理他们对私有仓库的访问。如果有足够的可用仓库位置，则还可以在公共和私有之间切换仓​库的状态。否则，可以升级 Docker Hub 计划。
# 9. Webhooks
webhook 是由特定事件触发的 HTTP 回调。将新镜像推送到仓库之后，可以使用 Hub 仓库的 webhook 通知人员、服务和其他应用程序（适用于自动构建）。例如，只要镜像可用，就可以触发自动化测试或部署。

要添加 webhooks，请转至 Hub 中所需的存储库，然后单击“Settings”框下的“Webhooks”。只有在成功 push 后才会调用 webhook。webhook 调用是一个 HTTP POST 请求，具有类似于下面所示的 JSON 有效负载。

webhook JSON 负载示例：
```
{
  "callback_url": "https://registry.hub.docker.com/u/svendowideit/busybox/hook/2141bc0cdec4hebec411i4c1g40242eg110020/",
  "push_data": {
    "images": [
        "27d47432a69bca5f2700e4dff7de0388ed65f9d3fb1ec645e2bc24c223dc1cc3",
        "51a9c7c1f8bb2fa19bcd09789a34e63f35abb80044bc10196e304f6634cc582c",
        "..."
    ],
    "pushed_at": 1.417566822e+09,
    "pusher": "svendowideit"
  },
  "repository": {
    "comment_count": 0,
    "date_created": 1.417566665e+09,
    "description": "",
    "full_description": "webhook triggered from a 'docker push'",
    "is_official": false,
    "is_private": false,
    "is_trusted": false,
    "name": "busybox",
    "namespace": "svendowideit",
    "owner": "svendowideit",
    "repo_name": "svendowideit/busybox",
    "repo_url": "https://registry.hub.docker.com/u/svendowideit/busybox/",
    "star_count": 0,
    "status": "Active"
  }
}
```
>注意：如果你想测试你的 webhook，推荐使用像 [requestb.in](http://requestb.in/) 这样的工具。另请注意，Docker Hub 服务器无法通过 IP 地址进行过滤。
## 9.1 Webhook 链
Webhook 链允许你将调用链接到多个服务。例如，只有在容器成功测试后，才可以触发容器的部署，然后在部署完成后更新各自的 Changelog。点击“Add webhook”按钮后，只需在链中添加尽可能多的网址即可。

成功 push 后，链中的第一个 webhook 将被调用。在回调请求验证通过后会继续后续的 URL。
## 9.2 激活回调
要验证 webhook 链中的回调，需要

1. 检索请求的 JSON 负载中的 `callback_url` 值。
2. 发送一个包含有效 JSON 数据的 POST 请求到这个 URL。

>注意：只有在最后一次回调被验证，链式请求才被视为完成。

为帮助调试或简单查看 webhook 的结果，请在可用的 webhook 的设置页面上查看“History”。

#### 回调的 JSON 数据
回调数据中的以下参数会被识别：

- `state`（必需）：接受的值是`success`、`failure`和 `error`。如果状态不成功，则 webhook 链会中断。
- `description`：包含 Docker Hub 上各种可用信息的字符串。最多 255 个字符。
- `context`：包含操作上下文的字符串。可以在 Docker Hub 中检索。最多 100 个字符。
- `target_url`：可找到操作结果的URL（The URL where the results of the operation can be found）。可以在 Docker Hub 上检索。

回调的负载数据示例：
```
{
  "state": "success",
  "description": "387 tests PASSED",
  "context": "Continuous integration by Acme CI",
  "target_url": "http://ci.acme.com/results/afd339c1c3d27"
}
```
