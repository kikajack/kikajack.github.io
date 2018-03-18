[原文地址](https://docs.docker.com/engine/security/trust/trust_automation/)

获取并构建镜像的自动化系统也可以使用内容信任。任何自动化环境都必须在处理镜像前手动或通过脚本方式设置 `DOCKER_CONTENT_TRUST`。
# 1. 绕过密码请求
要允许工具打包 docker 并推送可信内容，有两个环境变量允许你提供不需要脚本的密码，或者直接输入密码到：

- `DOCKER_CONTENT_TRUST_ROOT_PASSPHRASE`
- `DOCKER_CONTENT_TRUST_REPOSITORY_PASSPHRASE`

Docker 会尝试使用这些环境变量值作为密钥的密码。例如，镜像发布者可以导出仓库 `target` 和 `snapshot` 快照密码：
```
$  export DOCKER_CONTENT_TRUST_ROOT_PASSPHRASE="u7pEQcGoebUHm6LHe6"
$  export DOCKER_CONTENT_TRUST_REPOSITORY_PASSPHRASE="l7pEQcTKJjUHm6Lpe4"
```
然后，在推送新标签时，Docker 客户端不会请求这些值，而是自动签名：
```
$  docker push docker/trusttest:latest
The push refers to a repository [docker.io/docker/trusttest] (len: 1)
a9539b34a6ab: Image already exists
b3dbab3810fc: Image already exists
latest: digest: sha256:d149ab53f871 size: 3355
Signing and pushing trust metadata
```
直接使用 Notary 客户端时，它使用 [自己的一组环境变量](https://docs.docker.com/notary/reference/client-config/#environment-variables-optional)。
# 2. 构建时使用内容信任
可以使用内容信任进行构建。在运行 `docker build` 命令之前，应该手动或以脚本方式设置环境变量 `DOCKER_CONTENT_TRUST`。考虑下面的简单 Dockerfile。
```
FROM docker/trusttest:latest
RUN echo
```
`FROM` 标签正在获取已签名的镜像。不能构建具有本地或未签名的 `FROM` 的镜像。鉴于标签 `latest` 的内容信任数据存在，以下构建应该成功：
```
$  docker build -t docker/trusttest:testing .
Using default tag: latest
latest: Pulling from docker/trusttest

b3dbab3810fc: Pull complete
a9539b34a6ab: Pull complete
Digest: sha256:d149ab53f871
```
如果启用了内容信任，则依赖于没有信任数据的标记的 Dockerfile 构建，会导致构建命令失败（building from a Dockerfile that relies on tag without trust data, causes the build command to fail）：
```
$  docker build -t docker/trusttest:testing .
unable to process Dockerfile: No trust data for notrust
```
