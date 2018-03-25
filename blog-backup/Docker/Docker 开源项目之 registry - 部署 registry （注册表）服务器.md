[原文地址](https://docs.docker.com/registry/deploying/)

在部署 registry 之前需要现在主机上安装 Docker。registry 实际上就是运行在 Docker 中的 registry 镜像的实例。

本主题提供关于部署和配置 registry 的基本信息。要查看配置选项列表，请参考 [配置手册](https://docs.docker.com/registry/configuration/)。

如果你有 air-gapped 数据中心，参考 [air-gapped registries 的注意事项](https://docs.docker.com/registry/deploying/#considerations-for-air-gapped-registries)。
# 1. 运行本地 registry
使用下面命令来启动 registry 容器：
```
$ docker run -d -p 5000:5000 --restart=always --name registry registry:2
```
registry 现在可以使用了。

>警告：前几个示例显示的 registry 配置仅适用于测试环境。生产环境的 registry 必须受 TLS 保护，并且应该使用访问控制机制。继续阅读下一篇 registry 配置指南 以在生产环境中部署 registry。
# 2. 从 Docker Hub 复制镜像到 registry
可以从 Docker Hub 中提取镜像并将其推送到你的 registry。以下示例从 Docker Hub 中取出 `ubuntu:16.04` 镜像，并将其重新标记为 `my-ubuntu`，然后将其放入本地 registry。最后，删除本地的 `ubuntu:16.04` 和 `my-ubuntu` 镜像，并从本地 registry 中取出 `my-ubuntu` 镜像。

#### 1. 从 Docker Hub 中取出 `ubuntu:16.04` 镜像
```
$ docker pull ubuntu:16.04
```
#### 2. 为镜像添加标签 `localhost:5000/my-ubuntu`。这会会已经存在的镜像创建附加标签。标签第一部分是主机名和端口号，执行 push 操作时 Docker 将其解析为 registry 的位置。
```
$ docker tag ubuntu:16.04 localhost:5000/my-ubuntu
```
#### 3. 上传镜像到运行在 `localhost:5000` 的本地 registry：
```
$ docker push localhost:5000/my-ubuntu
```
#### 4. 删除本地缓存中的 `ubuntu:16.04` 和 `my-ubuntu` 镜像
删除后，可以测试从你的本地 registry 中获取镜像。这不会删除你的 registry 中的 `localhost:5000/my-ubuntu` 镜像。
```
$ docker image remove ubuntu:16.04
$ docker image remove localhost:5000/my-ubuntu
```
#### 5. 从你的本地 registry 中获取 `localhost:5000/my-ubuntu` 镜像
```
$ docker pull localhost:5000/my-ubuntu
```
# 3. 停止本地 registry
类似停止其他容器，通过 `docker container stop` 命令可以停止本地 registry：
```
$ docker container stop registry
```
通过 `docker container rm` 命令可以删除容器：
```
$ docker container stop registry && docker container rm -v registry
```
# 4. 基本配置
要配置容器，可以将附加选项或修改选项传递给 `docker run` 命令。

下面提供了配置 registry 的基本指导原则。更多详细信息，请参阅 [registry 配置参考](https://docs.docker.com/registry/configuration/)。
## 4.1 自动启动 registry
如果你希望将 registry 用作永久性基础架构的一部分，则应将其设置为在 Docker 重新启动或退出时自动重新启动。本示例使用 `--restart always` 标志为 registry 设置重新启动策略。
```
$ docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  registry:2
```
## 4.2 自定义发布的端口
如果端口 5000 已经被占用，或者你想运行多个本地 registry 来分隔关注的区域，则可以自定义 registry 的端口。下面的示例在端口 5001 上运行 registry，并将其命名为 `registry-test`。请记住，`-p` 值的第一部分是主机端口，第二部分是容器内的端口。在容器中，registry 默认在端口 5000 上监听。
```
$ docker run -d \
  -p 5001:5000 \
  --name registry-test \
  registry:2
```
如果要在容器中更改 registry 监听的端口（If you want to change the port the registry listens on within the container），可以使用环境变量 `REGISTRY_HTTP_ADDR` 来更改它。此命令使 registry 监听容器内的端口 5001：
```
$ docker run -d \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:5001 \
  -p 5001:5001 \
  --name registry-test \
  registry:2
```
# 5. 存储自定义
## 5.1 自定义存储位置
默认情况下，registry 数据在主机文件系统上作为 [docker卷](https://docs.docker.com/engine/tutorials/dockervolumes/) 持久保存。如果要将 registry 内容存储在主机文件系统上的特定位置，例如挂载到特定目录的 SSD 或 SAN ，则可以使用绑定挂载。绑定挂载更依赖于 Docker 主机的文件系统布局，但在许多情况下性能更高。下面的示例将主机的 `/mnt/registry` 目录绑定到 registry 容器的 `/var/lib/registry/` 目录中。
```
$ docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v /mnt/registry:/var/lib/registry \
  registry:2
```
## 5.2 自定义存储后端
默认情况下，registry 将其数据存储在本地文件系统中，无论你是使用绑定挂载还是使用卷。可以使用 [存储驱动程序](https://docs.docker.com/registry/storage-drivers/) 将 registry 数据存储在 Amazon S3 bucket，Google Cloud Platform 或其他存储后端中。更多信息，请参阅 [存储配置选项](https://docs.docker.com/registry/configuration/#storage)。
# 6. 运行可从外部访问的 registry
运行仅在本地主机上可访问的 registry 具有有限的用处。为了使你的 registry 能够被外部主机访问，必须先开启 TLS。

下面的示例在下一小节 运行 registry 作为服务 中进行了扩展。
## 6.1 获取证书
这些例子假设如下：

- 你的注册网址是 `https://myregistry.domain.com/`。
- 你的 DNS，路由和防火墙设置允许通过端口 443 访问 registry 所在主机。
- 你已经从认证中心（certificate authority，CA）获得证书。

如果你已获得中间证书，请参阅本小节最后一部分 使用中间证书。
#### 1. 创建证书目录
```
$ mkdir -p certs
```
将 `.crt` 和 `.key` 文件从 CA 复制到 `certs` 目录中。以下步骤假定这些文件被命名为 `domain.crt` 和 `domain.key`。
#### 2. 如果 registry 正在运行，则停止 registry
```
$ docker container stop registry
```
#### 3. 重新启动 registry
指示 registry 使用 TLS 证书。该命令将 `certs/` 目录绑定到容器中的 `/cert/` 目录，并设置环境变量来告诉容器在哪里找到 `domain.crt` 和 `domain.key` 文件。registry 在端口 443 上运行，即默认的 HTTPS 端口。
```
$ docker run -d \
  --restart=always \
  --name registry \
  -v `pwd`/certs:/certs \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  -p 443:443 \
  registry:2
```
### 4. Docker 客户端现在可以使用外部地址从 registry 中下载上传镜像：
```
$ docker pull ubuntu:16.04
$ docker tag ubuntu:16.04 myregistry.domain.com/my-ubuntu
$ docker push myregistry.domain.com/my-ubuntu
$ docker pull myregistry.domain.com/my-ubuntu
```
### 使用中间证书
证书颁发者可能会提供中间证书。在这种情况下，必须将你的证书与中间证书连接起来形成一个证书包（certificate bundle）。可以使用 `cat` 命令执行此操作：
```
cat domain.crt intermediate-certificates.pem > certs/domain.crt
```
就像在前面的示例中使用 `domain.crt` 文件一样，可以使用 certificate bundle。
## 6.2 对 Let’s Encrypt 的支持
registry 支持使用 Let's Encrypt 自动获取浏览器信任的证书。有关 Let's Encrypt 的更多信息，请参阅 https://letsencrypt.org/how-it-works/ 和 [registry 配置](https://docs.docker.com/registry/configuration/#letsencrypt) 的相关部分。
## 6.3 使用不安全的 registry（仅适合测试环境）
可以使用自签名证书，或者不安全地使用 registry。除非你已为自签名证书设置验证，否则这仅用于测试。请参阅 [运行不安全的注册表](https://docs.docker.com/registry/insecure/)。
# 7. Run the registry as a service
[Swarm 服务](https://docs.docker.com/engine/swarm/services/) 相对于独立容器有几个优势。他们使用声明式模型（declarative model），这意味着你定义了所需的状态，并且 Docker 可以使你的服务保持在该状态。服务提供自动负载均衡扩展，以及控制服务分布的能力等优势。服务还允许你在 [secrets](https://docs.docker.com/engine/swarm/secrets/) 中存储敏感数据，例如 TLS 证书。

你使用的存储后端决定你是使用完全伸缩的服务还是仅使用单个节点或节点约束的服务。

- 如果使用分布式存储驱动程序（如 Amazon S3），则可以使用完全复制的服务（a fully replicated service）。每个工作人员都可以写入存储后端，而不会导致写入冲突。
- 如果使用本地绑定挂载或卷，则每个工作节点都会写入其自己的存储位置，这意味着每个 registry 都包含一个不同的数据集。可以通过使用单副本服务（single-replica service）和节点约束来解决此问题，以确保只有一个工作节点正在写入绑定挂载。

以下示例将 registry 作为单副本服务启动，该服务可通过任何 swarm 节点的 80 端口访问。假定你使用的是与前面示例中相同的 TLS 证书。

首先，将 TLS 证书和密钥保存为 secrets：
```
$ docker secret create domain.crt certs/domain.crt

$ docker secret create domain.key certs/domain.key
```
接下来，将标签添加到要运行 registry 的节点。可以使用 `docker node ls` 获取节点的名称。用下面的 `node1` 替换节点的名称。
```
$ docker node update --label-add registry=true node1
```
然后创建服务，授予其访问这两个 secrets 的权限，并将其限制为仅在具有标签 `registry=true` 的节点上运行。除了约束之外，还指定一次只能运行一个副本。该示例将 swarm 节点上的 `/mnt/registry` 绑定挂载到容器中的 `/var/lib/registry/`。绑定挂载依赖于预先存在的源目录，因此确保 `node1` 上存在 `/mnt/registry`。在运行以下 `docker service create` 命令之前，可能需要创建它。

默认情况下，secrets 在 `/run/secrets/<secret-name>` 处挂载到服务中。
```
$ docker service create \
  --name registry \
  --secret domain.crt \
  --secret domain.key \
  --constraint 'node.labels.registry==true' \
  --mount type=bind,src=/mnt/registry,dst=/var/lib/registry \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:80 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/run/secrets/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/run/secrets/domain.key \
  --publish published=80,target=80 \
  --replicas 1 \
  registry:2
```
可以访问任何 swarm 节点的 80 端口上的服务。Docker 将请求发送到正在运行该服务的节点。
# 8. 负载均衡的考虑
人们可能想要使用负载均衡器分配负载，终止 TLS 或提供高可用性。虽然完整的负载均衡设置不在本文档的范围内，但有一些注意事项可以使该过程更加流畅。

最重要的方面是，负载均衡的 registry 集群必须共享相同的资源。对于当前版本的 registry，这意味着以下内容必须相同：

- 存储驱动程序
- HTTP Secret
- Redis 缓存（如果已配置）

上述任何差异都会导致服务请求出现问题。例如，如果使用的是文件系统驱动程序，则所有 registry 实例都必须能够访问同一台计算机上的相同文件系统根目录。对于其他驱动程序（如 S3 或 Azure），它们应该访问相同的资源并共享相同的配置。HTTP Secret 协调上传（The HTTP Secret coordinates uploads），跨实例也必须相同。可以配置不同的 redis 实例（编写本文时），但如果实例不共享，则不是最优的，因为此时会导致更多的请求发送到后端服务器而不是使用 redis 缓存。
## 8.1 重要/必需的 HTTP 头
正确获取 header 非常重要。对于“/v2/”地址空间下的任何请求的所有响应，即使是 4xx 响应，`Docker-Distribution-API-Version` 的 header 也应设置为值“registry/2.0”。如果需要，此头允许 docker 引擎快速解析身份验证提示信息并回退到版本 1 的registry。确认此设置是否正确可以帮助避免回退问题。（This header allows the docker engine to quickly resolve authentication realms and fallback to version 1 registries, if necessary. Confirming this is setup correctly can help avoid problems with fallback.）

在同样的思路中，你必须确保正确地将 `X-Forwarded-Proto`，`X-Forwarded-For` 和 `Host` header 发送到它们的“客户端”值。如果不这样做，通常会使 registry 问题重定向到内部主机名或从 https 降级到 http。（Failure to do so usually makes the registry issue redirects to internal hostnames or downgrading from https to http.）

当“/v2/”端点在没有凭证的情况下被访问时，正确安全的 registry 应返回 401。响应应包含 `WWW-Authenticate`，为如何进行身份验证提供指导，例如使用基本身份验证或令牌服务。如果负载平衡器有健康检查，建议将其配置为将 401 响应视为健康状况，其他情况视为关闭。通过确保认证的配置问题不会意外暴露未受保护的 registry，可以确保安全。如果你使用的是不太复杂的负载平衡器（例如亚马逊的 Elastic Load Balancer），它不允许更改健康的响应代码，则健康检查可以定向到“/”，它总是返回 `200 OK` 响应。
# 9. 限制访问
除了在安全的本地网络上运行的 registry 外，应始终对 registry 实施访问限制。
## 9.1 原生基本身份验证 native basic authentication
实现访问限制的最简单方法是通过基本认证（类似于其他 Web 服务器的基本认证机制）。这个例子使用通过 `htpasswd` 来存储 secrets 的原生基本身份认证。

>警告：无法使用以纯文本格式发送凭证的身份验证方案进行身份验证。必须首先配置 TLS 才能使验证生效。
#### 1. 为用户 testuser 创建一个包含密码 testpassword 的密码文件：
```
$ mkdir auth
$ docker run \
  --entrypoint htpasswd \
  registry:2 -Bbn testuser testpassword > auth/htpasswd
```
#### 2. 停止 registry
```
$ docker container stop registry
```
#### 3. 使用基本身份验证启动 registry
```
$ docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v `pwd`/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  -v `pwd`/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry:2
```
#### 4. 尝试从 registry 获取镜像，或上传镜像到 registry。这会全部失败。
#### 5. 登录 registry
```
$ docker login myregistrydomain.com:5000
```
在第一步提供用户名和密码。

测试现在可以从 registry 中获取镜像或将镜像推送到 registry。

> X509 错误：X509 错误通常表明你正在尝试使用自签名证书，但未正确配置 Docker 守护程序。请参阅 [运行不安全的注册表](https://docs.docker.com/registry/insecure/)。
## 9.2 更多高级认证
你可能希望通过在 registry 前使用代理来实现更高级的基本身份验证。见 [recipes](https://docs.docker.com/registry/recipes/) 列表。

注册表还支持委派认证（delegated authentiation），将用户重定向到特定的可信令牌服务器（token server）。这种方法设置起来更加复杂，只有在需要完全配置 ACL 并需要更多控制 registry 与全局授权和身份验证系统的集成时才有意义（only makes sense if you need to fully configure ACLs and need more control over the registry’s integration into your global authorization and authentication systems）。请参阅以下 [背景信息](https://docs.docker.com/registry/spec/auth/token/) 和 [配置信息](https://docs.docker.com/registry/configuration/#auth)。

这种方法需要你使用自己的或第三方认证系统。
# 10. 通过 compose 文件部署 registry
如果 registry 很复杂，那么使用 Docker compose 文件来部署它可能更容易，而不是依赖于特定的 `docker run` 调用。使用以下示例 `docker-compose.yml` 作为模板。
```
registry:
  restart: always
  image: registry:2
  ports:
    - 5000:5000
  environment:
    REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
    REGISTRY_HTTP_TLS_KEY: /certs/domain.key
    REGISTRY_AUTH: htpasswd
    REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
    REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
  volumes:
    - /path/data:/var/lib/registry
    - /path/certs:/certs
    - /path/auth:/auth
```
将 `/path` 替换为包含 `certs/` 和 `auth/` 目录的目录。

通过在包含 `docker-compose.yml` 文件的目录中发出以下命令来启你的 registry：
```
$ docker-compose up -d
```
# 11. 对 air-gapped registry 的考虑
可以在没有互联网连接的环境中运行 registry。但是，如果你依赖任何非本地镜像，则需要考虑以下几点：

- 你可能需要在连接的主机上构建本地 registry 的数据卷，可以在其中运行 `docker pull` 以获取远程可用的任何镜像，然后将 registry 的数据卷迁移到隔离网络（air-gapped network）。
- 某些镜像，如官方 Microsoft Windows 基本镜像，不可分发。这意味着当你将基于这些镜像之一的镜像上传到你的私人 registry 时，不会推送不可分发的层，而是始终从其授权位置获取。这对于连接互联网的主机来说很好，但不能用于空隙设置（air-gapped set-up）。

在 Docker 17.06 及更高版本中，可以配置 Docker 守护程序，以允许在这种情况下将不可分配的层上传到私有 registry。这仅适用于存在不可分配镜像的空隙设置或带宽极端有限的情况下。你有责任确保你符合不可分配的层的使用条款。
#### 1. 编辑 Linux 主机上的 `/etc/docker/` 和 Windows Server 上的 `C:\ProgramData\docker\config\daemon.json` 中的 `daemon.json` 文件。假设文件先前为空，请添加以下内容：
```
{
  "allow-nondistributable-artifacts": ["myregistrydomain.com:5000"]
}
```
值是 registry 地址数组，用逗号分隔。
保存并退出。
#### 2. 重启 Docker
#### 3. 如果 registry 没有自动重启，则重启 registry
#### 4. 当你将镜像上传到 registry 时，它们的不可分发层也会上传到 registry。
>警告：不可分发的构件通常对如何以及在何处分发和共享有限制。只有使用此功能才能将工件推送到私有 registry，并确保你符合包含重新分配不可分配构件的任何条款。
