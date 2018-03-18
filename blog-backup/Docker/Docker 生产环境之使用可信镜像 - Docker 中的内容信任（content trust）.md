[原文地址](https://docs.docker.com/engine/security/trust/content_trust/)

在联网系统间传输数据时，信任是一个核心问题。特别是，当通过互联网等不可信介质进行通信时，确保系统运行的所有数据的完整性和发布者至关重要。使用 Docker Engine 可以将镜像（数据）推送到公共或私有 registry。内容信任使你能够验证通过任何通道从 registry 接收的所有数据的完整性和发布者。
# 1. 关于 Docker 中的信任
内容信任允许使用远程 Docker registry 执行操作，以强制客户端对镜像的标记进行签名和验证。内容信任提供了使用数字签名来收发远程 Docker registry 数据的能力。这些签名允许客户端验证特定镜像标签的完整性和发布者。

目前，内容信任被默认禁用。要启用它，请将 `DOCKER_CONTENT_TRUST` 环境变量设置为 `1`。请参阅 Docker 客户端的 [环境变量](https://docs.docker.com/engine/reference/commandline/cli/#environment-variables) 和 [Notary](https://docs.docker.com/engine/reference/commandline/cli/#notary) 配置以获取更多选项。

一旦启用内容信任，镜像发布者就可以对其镜像进行签名。镜像消费者可以确保他们使用的镜像已经经过签名。发布者和消费者可以是单独的个人或组织中的个人。Docker 的内容信任支持用户和自动化进程，如构建。

当启用内容信任时，如果你使用 Docker CE，则在客户端进行推送和验证后，在客户端上进行签名。如果将 Docker EE 与 UCP 一起使用，并且已将 UCP 配置为要求在部署之前对镜像进行签名，则签名将由 UCP 进行验证。
## 1.1 镜像标签和内容信任
单个镜像记录具有以下标识符：
```
[REGISTRY_HOST[:REGISTRY_PORT]/]REPOSITORY[:TAG]
```
一个特定的镜像 `REPOSITORY` 可以有多个标签。例如，`latest` 和 `3.1.2` 都是 `mongo` 镜像上的标签。镜像发布者可以在每次构建时多次更改镜像来构建镜像和标签组合。

内容信任与镜像的 `TAG` 部分相关联。每个镜像仓库都有一组给镜像发布者使用的用于签署镜像标签的密钥。镜像发布者可以自行决定签署哪些标签。

镜像存储库可以包含同时具有已签名标签和未签名标签的镜像。例如，考虑 [`Mongo` 镜像仓库](https://hub.docker.com/r/library/mongo/tags/)。`latest` 标签是未签名的，而 `3.1.6` 标签是签名的。镜像发布者有责任决定镜像标签是否已签名。下图中，一些镜像标签被签名，而另一些则没有：

![tag_signing](http://img.blog.csdn.net/20180318134659061?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

镜像发布者可以选择是否签名特定标签。这将导致未签名标签的内容与同名的已签名标签的内容可能不匹配。例如，镜像发布者可以将已经标记的镜像推送到 `someimage:latest` 并签名。稍后，同一个发布者可以推送未签名的 `someimage:latest` 镜像。第二次推送替换 `latest` 未签名标签，但不会影响已签名的 `latest` 版本。可以自由选择为哪个标签签名，使得镜像发布者可以在正式签名之前迭代未签名的镜像版本。

镜像使用者可以启用内容信任以确保他们使用的镜像已被签名。如果使用者启用内容信任，则只能使用受信任的镜像进行 pull，run 或 build。启用内容信任就像戴着一副有色眼镜。使用者“看到”只有签名的镜像标签，不想要的、未签名的镜像标签对他们来说是“不可见的”。

![trust_view](http://img.blog.csdn.net/20180318134727257?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

对于尚未启用内容信任的消费者，可以正常使用所有的 Docker 镜像。无论是否签名，每个镜像都可见。
## 1.2 内容信任操作和密钥
启用内容信任时，对标记镜像执行操作的 docker CLI 命令必须具有内容签名或显式内容散列。与内容信任一起运行的命令是：

- push
- build
- create
- pull
- run

例如，开启内容信任后，`docker pull someimage:latest` 命令只有在 `someimage:latest` 确实被签名时才会成功执行。然而，附带显式内容散列的操作只要散列在匹配就会成功执行。
```
$ docker pull someimage@sha256:d149ab53f8718e987c3a3024bb8aa0e2caadf6c0328f1d9d850b2a2a67f2819a
```
对镜像标签的信任通过使用签名密钥来管理。首次调用使用内容信任的操作时会创建密钥集。密钥集由以下几类密钥组成：

- 作为镜像标记的内容信任的根的脱机密钥
- 签名标签的仓库或标签密钥
- 服务器管理的密钥，如时间戳密钥，为仓库提供最新的安全保证

下图描述了各种签名密钥及其关系：

![trust_components](http://img.blog.csdn.net/20180318134801916?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

>警告：丢失根密钥非常难以恢复。纠正这种损失需要 Docker 支持人员的干预来重置仓库状态。这种损失还需要所有使用此仓库中的签名标记的使用者在丢失之前进行手动干预。

应该将根密钥备份到安全的地方。鉴于根密钥仅用于创建新的仓库，最好将其脱机存储在硬件中。有关保护和备份密钥的详细信息，请务必阅读 如何为内容信任管理密钥。
# 2. 典型内容信任操作
本节将学习使用 Docker 镜像执行的典型可信操作。具体来说，我们通过以下步骤来帮助我们执行这些可信操作：

- Build 并 push 未签名镜像
- Pull 未签名镜像
- Build 并 push 签名镜像
- Pull 之前 push 的签名镜像
- Pull 之前 push 的未签名镜像
## 2.1 启用和禁用每个 shell 或每个调用的内容信任
在 shell 中，可以通过设置 `DOCKER_CONTENT_TRUST` 环境变量来启用内容信任。在每个 shell 中启用是非常有用的，因为可以为可信操作配置一个 shell，为不可信操作配置另一个终端 shell。还可以将此声明添加到 shell 配置文件中，以便默认情况下始终打开它。

要在 bash shell 中启用内容信任，请输入以下命令：
```
export DOCKER_CONTENT_TRUST=1
```
一旦设置，每个“tag”操作都需要一个用于获取可信 tag 的密钥。

在设置了 `DOCKER_CONTENT_TRUST` 环境变量的环境中，可以通过 `--disable-content-trust` 标志关闭内容信任后在标记镜像上运行单独的操作。

看看使用不受信任的父镜像的 Dockerfile：
```
$  cat Dockerfile
FROM docker/trusttest:latest
RUN echo
```
要通过这个 Dockerfile 成功构建容器，可以这么做：
```
$  docker build --disable-content-trust -t <username>/nottrusttest:latest .
Sending build context to Docker daemon 42.84 MB
...
Successfully built f21b872447dc
```
其他命令的操作都类似：
```
$  docker pull --disable-content-trust docker/trusttest:latest
...
$  docker push --disable-content-trust <username>/nottrusttest:latest
...
```
要在调用命令时开启内容信任，而不管 `DOCKER_CONTENT_TRUST` 环境变量是如何设置的：
```
$  docker build --disable-content-trust=false -t <username>/trusttest:testing .
```
所有受信任的操作都支持 `--disable-content-trust` 标志。
## 2.2 推送可信的内容
要为特定镜像标记创建签名内容，只需启用内容信任并推送标记的镜像即可。如果这是第一次在系统上使用内容信任推送镜像，则会话如下所示：
```
$ docker push <username>/trusttest:testing
The push refers to a repository [docker.io/<username>/trusttest] (len: 1)
9a61b6b1315e: Image already exists
902b87aaaec9: Image already exists
latest: digest: sha256:d02adacee0ac7a5be140adb94fa1dae64f4e71a68696e7f8e7cbf9db8dd49418 size: 3220
Signing and pushing trust metadata
You are about to create a new root signing key passphrase. This passphrase
will be used to protect the most sensitive key in your signing system. Please
choose a long, complex passphrase and be careful to keep the password and the
key file itself secure and backed up. It is highly recommended that you use a
password manager to generate the passphrase and keep it safe. There will be no
way to recover this key. You can find the key in your config directory.
Enter passphrase for new root key with id a1d96fb:
Repeat passphrase for new root key with id a1d96fb:
Enter passphrase for new repository key with id docker.io/<username>/trusttest (3a932f1):
Repeat passphrase for new repository key with id docker.io/<username>/trusttest (3a932f1):
Finished initializing "docker.io/<username>/trusttest"
```
开启内容信任后第一次推送标记镜像时，docker 客户端会识别出这是你第一次推送，并且会：

- 警告你这会创建一个新的根密钥
- 请求根密钥的密码
- 在 `~/.docker/trust` 目录中生成一个根密钥
- 请求仓库密钥的密码
- 在 `~/.docker/trust` 目录中生成一个仓库密钥

为根密钥和存储库密钥对选择的密码应随机生成并存储在密码管理器中。

>注意：如果省略 `testing` 标记，则跳过内容信任。即使启用了内容信任，即使这是第一次推送，情况也是如此。
```
$ docker push <username>/trusttest
The push refers to a repository [docker.io/<username>/trusttest] (len: 1)
9a61b6b1315e: Image successfully pushed
902b87aaaec9: Image successfully pushed
latest: digest: sha256:a9a9c4402604b703bed1c847f6d85faac97686e48c579bd9c3b0fa6694a398fc size: 3220
No tag specified, skipping trust metadata push
```
它被跳过，因为消息指出，你没有提供镜像 `TAG` 值。在 Docker 内容信任中，签名与标签相关联。

一旦在系统上拥有 root 密钥，创建的后续镜像仓库就可以使用相同的 root 密钥：
```
$ docker push docker.io/<username>/otherimage:latest
The push refers to a repository [docker.io/<username>/otherimage] (len: 1)
a9539b34a6ab: Image successfully pushed
b3dbab3810fc: Image successfully pushed
latest: digest: sha256:d2ba1e603661a59940bfad7072eba698b79a8b20ccbb4e3bfb6f9e367ea43939 size: 3346
Signing and pushing trust metadata
Enter key passphrase for root key with id a1d96fb:
Enter passphrase for new repository key with id docker.io/<username>/otherimage (bb045e3):
Repeat passphrase for new repository key with id docker.io/<username>/otherimage (bb045e3):
Finished initializing "docker.io/<username>/otherimage"
```
新镜像具有自己的仓库密钥和时间戳密钥。`latest` 标签用这两个标签签名。
## 2.3 获取镜像内容
使用镜像的常用方法是 `pull` 镜像。启用内容信任后，Docker 客户端仅允许 `docker pull` 检索已签名的镜像。让我们试着获取你之前签名和推送的镜像：
```
$  docker pull <username>/trusttest:testing
Pull (1 of 1): <username>/trusttest:testing@sha256:d149ab53f871
...
Tagging <username>/trusttest@sha256:d149ab53f871 as docker/trusttest:testing
```
下面的例子中，命令没有指定 tag，所以系统使用默认的 `latest` tag，但是 `docker/trusttest:latest` 并没有签名。
```
$ docker pull docker/trusttest
Using default tag: latest
no trust data available
```
因为 `docker/trusttest:latest`没有签名，所以 `pull` 失败。
