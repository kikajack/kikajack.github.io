[原文地址](https://docs.docker.com/engine/security/trust/trust_delegation/)

Docker Engine 支持使用 `targets/releases` delegation （委托，授权）作为可信镜像标记的规范来源。

使用此 delegation 可让你与其他发布者协作，而不共享你仓库密钥，这是你的 `targets` 和 `snapshot` 密钥的组合。有关更多信息，请参阅 管理内容信任的密钥。合作者可以保留自己的 delegation 密钥。

`targets/releases` delegation 目前是可选特性 - 要设置 delegation，就必须使用 Notary CLI：
#### 1. [下载客户端](https://github.com/docker/notary/releases) 并确保它在你的路径上可用
#### 2. 用下面的内容创建配置文件 `~/.notary/config.json`：
```
 {
   "trust_dir" : "~/.docker/trust",
   "remote_server": {
     "url": "https://notary.docker.io"
   }
 }
```
这告诉 Notary Docker内容信任数据的存储位置，以及在 Docker Hub 中使用用于镜像的 Notary 服务器。

有关如何在默认 Docker Content Trust 用例外使用 Notary 的更多详细信息，请参阅 [Notary CLI 文档](https://docs.docker.com/notary/getting_started/)。

使用 Notary 客户端发布和列出 delegation 更改时，你的 Docker Hub 凭据是必需的。
# 1. 创建 delegation 密钥
你的协作者需要生成一个私钥（RSA 或 ECDSA）并为你提供公钥，以便将其添加到 `targets/releases` delegation 中。

他们生成这些密钥的最简单方法是使用 OpenSSL。以下是如何生成 2048 位 RSA 部分密钥的示例（所有 RSA 密钥必须至少为 2048 位）：
```
$ openssl genrsa -out delegation.key 2048
Generating RSA private key, 2048 bit long modulus
....................................................+++
............+++
e is 65537 (0x10001)
```
他们应该保证 `delegation.key` 的私密性，因为这是用来加密标签的。

然后他们需要生成一个包含公钥的 x509 证书，这是你需要的。这是生成 CSR（certificate signing request，证书签名请求）的命令：
```
$ openssl req -new -sha256 -key delegation.key -out delegation.csr
```
然后他们可以将它发送给任何你信任的 CA 来签署证书，或者他们可以自行签署证书（在此示例中，创建有效期为1年的证书）：
```
$ openssl x509 -req -sha256 -days 365 -in delegation.csr -signkey delegation.key -out delegation.crt
```
然后，他们需要给你 `delegation.crt` 文件，不管是自行签名还是由 CA 签名。
# 2. 为已存在的仓库添加 delegation 密钥
如果你的存储库是使用 1.11 之前的 Docker Engine 版本创建的，那么在添加任何 delegation 之前，应该将快照密钥 rotate 到服务器，以便协作者不需要快照密钥来签署和发布 tag：
```
$ notary key rotate docker.io/<username>/<imagename> snapshot -r
```
这会告知 Notary 去为你指定的镜像仓库 rotate 密钥。`docker.io/` 前缀是必须的。`snapshot -r` 指定要旋转的快照密钥并且希望服务器对其进行管理（-r 代表“remote”）（specifies that you want to rotate the snapshot key and that you want the server to manage it）。

在添加 delegation 时，必须使用你希望委派给协作者的公钥来获取 PEM 编码的 x509 证书。

假设你拥有证书 `delegation.crt`，则可以为该用户添加一个 delegation，然后发布 delegation 更改：
```
$ notary delegation add docker.io/<username>/<imagename> targets/releases delegation.crt --all-paths
$ notary publish docker.io/<username>/<imagename>
```
上例演示了将 `targets/releases` delegation 添加到镜像仓库的请求（如果它不存在）。 确保使用 `targets/releases` - Notary 支持多个 delegation 角色，因此如果输错 delegation 名称，Notary CLI 不会报错。但是，Docker 引擎仅支持从 `targets/releases` 进行读取。

它还将协作者的公钥添加到 delegation 中，使他们能够签署 `targets/releases` 委派，只要他们拥有与此公钥相对应的私钥。`--all-paths` 标志告诉 Notary 不要限制可以登录到 `targets/releases` 的标签名称，强烈建议使用。

发布更改会告诉服务器有关 `targets/releases` delegation 的更改。

发布后，查看 delegation 信息以确保正确地将密钥添加到 `targets/releases`：
```
$ notary delegation list docker.io/<username>/<imagename>

      ROLE               PATHS                                   KEY IDS                                THRESHOLD
---------------------------------------------------------------------------------------------------------------
  targets/releases   "" <all paths>  729c7094a8210fd1e780e7b17b7bb55c9a28a48b871b07f65d97baf93898523a   1
```
可以通过其路径和刚刚添加的密钥 ID 来查看 `targets/releases`。

Notary 目前不会将协作者名称映射到密钥，因此我们建议你一次添加并列出 delegation 密钥，并在需要删除协作者时自己将密钥 ID 映射到协作者。
# 3. 从已存在的仓库中删除 delegation 密钥
要撤消协作者为镜像仓库签署标签的功能，你需要从 `targets/releases` delegation 中删除其密钥。要做到这一点，你需要他们的密钥的 ID。
```
$ notary delegation remove docker.io/<username>/<imagename> targets/releases 729c7094a8210fd1e780e7b17b7bb55c9a28a48b871b07f65d97baf93898523a

Removal of delegation role targets/releases with keys [729c7094a8210fd1e780e7b17b7bb55c9a28a48b871b07f65d97baf93898523a], to repository "docker.io/<username>/<imagename>" staged for next publish.
```
撤销在提交后立刻生效：
```
$ notary publish docker.io/<username>/<imagename>
```
通过从 `targets/releases` delegation 中删除所有密钥，委派（以及任何登录到其中的标签）都将被删除。这意味着这些标签全部被删除，并且你最终可能会收到由目标密钥直接签名的较旧的旧版标签。
原文：
By removing all the keys from the targets/releases delegation, the delegation (and any tags that are signed into it) is removed. That means that these tags are all deleted, and you may end up with older, legacy tags that were signed directly by the targets key.
# 4. 从仓库中彻底删除 targets/releases delegation
如果确定 delegation 不适合你，则可以完全删除 `targets/releases` delegation。但是，这也会删除当前位于  `targets/releases` 中的所有标签，并且最终可能会使用由目标键直接签名的旧版标签。

要删除 `targets/releases` delegation：
```
$ notary delegation remove docker.io/<username>/<imagename> targets/releases

Are you sure you want to remove all data for this delegation? (yes/no)
yes

Forced removal (including all keys and paths) of delegation role targets/releases to repository "docker.io/<username>/<imagename>" staged for next publish.

$ notary publish docker.io/<username>/<imagename>
```
# 5. 以合作者身份发布可信数据
作为已将私钥添加到仓库中的 `targets/releases` delegation 的协作者，需要将你生成的私钥导入到 Content Trust 中。

可以通过下面命令实现：
```
$ notary key import delegation.key --role user
```
其中 `delegation.key` 是包含你的 PEM 编码私钥的文件。

完成之后，在包含你的 `targets/releases` delegation 中密钥的任何仓库上运行 `docker push` 会自动使用此导入的密钥来签名标签。
# 6. `docker push` 行为
当使用 Docker Content Trust 运行 `docker push` 时，Docker Engine 会尝试使用 `targets/releases` delegation 进行签名和推送（如果存在）。如果没有，则如果密钥可用，则使用目标密钥来签署标签。
# 7. `docker pull` and `docker build` 行为
当使用 Docker Content Trust 运行 `docker pull` 或 `docker build` 时，Docker Engine 会拉取仅由 `targets/releases` delegation 角色签署的标签或直接用目标密钥签名的旧版标签。
