[原文地址](https://docs.docker.com/engine/security/trust/trust_key_mng/)

通过使用密钥来管理镜像标签的信任。Docker 的内容信任使用五种不同类型的密钥：
密钥	|描述
-|-
root key|	镜像标签的内容信任根。启用内容信任后，将创建一次 root 密钥。也称为脱机密钥，因为它应该脱机保存。
targets	|此密钥允许签署镜像标记，以管理 delegation，包括 delegations 密钥或允许的 delegations 路径。也称为存储库密钥，因为此密钥确定可以将哪些标签签入镜像存储库。
snapshot	|标记当前的镜像标签集合，防止混合和匹配攻击。
timestamp	|允许 Docker 镜像存储库具有最新安全保证，而无需在客户端定期刷新内容。
delegation|	可选的标签密钥，允许你将签名镜像标签委托给其他发布者，而无需共享 targets 密钥。

在第一次启用 Content Trust 的情况下执行 `docker push` 时，会自动为镜像存储库生成 root，targets，snapshot 和 timestamp 密钥：

- root 和 targets 密钥在本地生成并存储在客户端。
- timestamp 和 snapshot 安全地生成并存储在与 Docker registry 一起部署的签名服务器中。这些密钥是在不直接暴露于互联网的后端服务中生成的，并且在空闲时进行加密。

Delegation 密钥是可选的，不是作为普通 docker 工作流程的一部分生成的。他们需要手动生成并添加到存储库。

注意：在 Docker Engine 1.11 之前，snapshot 密钥也是在客户端本地生成和存储的。使用 Notary CLI 再次本地管理 snapshot 密钥，以获取使用较新版本的 Docker 创建的存储库。
# 1. 选择一个密码
为 root 密钥和 repository 密钥选择的密码应该随机生成并存储在密码管理器中。使用 repository 密钥允许用户在存储库上签名镜像标签。密码短语用于加密密钥，并确保丢失的笔记本电脑或意外备份不会使密钥材料处于危险之中。
# 2. 备份密钥
所有 Docker 信任的密钥都使用你在创建时提供的密码进行加密存储。即使如此，仍然应该留意备份位置。好的做法是创建两个加密的 USB 密钥。

将密钥备份到安全的位置非常重要。repository 密钥的丢失是可恢复的；root 密钥丢失后无法恢复。

Docker 客户端将密钥存储在 `~/.docker/trust/private` 目录中。在备份之前，应该通过 `tar` 将它们压缩到一个存档中：
```
$ umask 077; tar -zcvf private_keys_backup.tar.gz ~/.docker/trust/private; umask 022
```
# 3. 硬件存储和签名
Docker Content Trust 可以使用 Yubikey 4 的 root 密钥进行存储和签名。Yubikey 优先于存储在文件系统中的密钥。当你使用内容信任初始化新的存储库时，Docker Engine 会在本地查找 root 密钥。如果没有找到密钥并且存在 Yubikey 4，则 Docker Engine 将在 Yubikey 4 中创建一个 root 密钥。请参阅 [Notary 文档](https://docs.docker.com/notary/advanced_usage/#use-a-yubikey) 以获取更多详细信息。

在 Docker Engine 1.11 之前，此功能仅在实验分支中。
# 4. 丢失密钥
如果发布者丢失密钥，这意味着无法为你的存储库签署可信内容。如果丢失密钥，请联系 Docker Support（support@docker.com）以重置存储库状态。

这种损失还需要每个在损失之前获取标签图像的消费者进行手动干预。 镜像使用者会收到他们已经下载的内容的错误讯息：
```
Warning: potential malicious behavior - trust data has insufficient signatures for remote repository docker.io/my/image: valid signatures did not meet threshold
```
为了解决这个问题，他们需要下载一个用新密钥签名的新镜像标签。
