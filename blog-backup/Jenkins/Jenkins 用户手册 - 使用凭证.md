[原文地址](https://jenkins.io/doc/book/using/using-credentials/)

有许多第三方网站和应用程序可以与 Jenkins 进行交互，例如程序代码仓库，云存储系统和服务等。

此类应用程序的系统管理员可以在应用程序中配置凭证以专供 Jenkins 使用。通常通过将访问控制应用于这些凭证来完成这项工作，以“锁定”Jenkins可用的应用程序功能区域。一旦 Jenkins 管理员（即管理 Jenkins 站点的 Jenkins 用户）在 Jenkins 中添加/配置这些凭证，Pipeline 项目就可以使用凭证与这些第三方应用程序进行交互。

>注意：在这个页面及相关页面描述的 Jenkins 凭证功能通过 [Credentials Binding 插件](https://plugins.jenkins.io/credentials-binding) 提供 

Jenkins 中保存的凭证可以用于：

- 任何适用于 Jenkins 的任何地方（即全局证书）
- 特定的 Pipeline 项目（更多信息参考 [使用 Jenkinsfile](https://jenkins.io/doc/book/pipeline/jenkinsfile) 来 [处理凭据](https://jenkins.io/doc/book/pipeline/jenkinsfile#handling-credentials) 部分）
- 特定的 Jenkins 用户（就像 [在 Blue Ocean 创建的 Pipeline 项目](https://jenkins.io/doc/book/blueocean/creating-pipelines) 一样）

Jenkins 可以保存下面几种凭证：

- **Secret 文本** - 例如 API token（例如 GitHub 的个人 access token）
- **用户名和密码** - 可以作为单独的组件处理，也可以作为 `username:password` 格式的冒号分隔字符串来处理（请参阅处理凭据中的更多信息）
- **Secret 文件** - 实际上是文件中的秘密内容
- **使用私钥的 SSH 用户名** - 一个 [SSH 密钥对](http://www.snailbook.com/protocols.html)
- **证书** - 一个 [PKCS#12 证书文件](https://tools.ietf.org/html/rfc7292) 和可选的密码
- **Docker 主机证书身份验证凭证**

#1. 凭证安全
为确保安全，Jenkins 中配置的凭证在 Jenkins 主实例中加密存储（通过 Jenkins 实例的 ID 来加密）并且只能通过他们的凭证 ID 在 Pipeline 项目中处理。

这最大限度地减少向 Jenkins 用户暴露实际证书本身的可能性，并且限制了将功能证书从一个 Jenkins 实例复制到另一个 Jenkins 实例的能力。
#2. 配置凭证
这一部分描述了在 Jenkins 中配置证书的步骤。


凭证可以被任何具有 **Credentials > Create** 权限（通过 **Matrix-based security** 设置）的用户添加到 Jenkins。这些权限可以被具有 **Administer** 权限的用户配置。详情参考 [Managing Security](https://jenkins.io/doc/book/managing/security) 中的 [Authorization 章节](https://jenkins.io/doc/book/managing/security/#authorization)。

否则，如果 Jenkins 实例的 **Configure Global Security** （配置全局安全性）设置页面的授权设置设置为默认值，那么任何Jenkins用户都可以添加和配置凭据。登录用户可以执行任何设置，或者任何人都可以执行任何设置。
##2.1 添加新的全局凭证
在 Jenkins 示例中添加新的全局凭证的十步：
###1 如果需要，确保已经登录 Jenkins（作为具有 Credentials > Create 权限的用户）
###2 在 Jenkins 的首页，点击左边的 Credentials > System。
###3 在 System 中，点击 Global credentials (unrestricted) 链接访问默认域名。
###4 点击左侧的 Add Credentials
注意：如果在这个默认域名中没有凭证，那就需要点击 add some credentials 链接（跟点击 Add Credentials 链接类似）
###5 Kind 字段中，选择要添加的凭证的类型
###6 Scope 字段中，选择下面的一个：
- **Global** - 如果要添加的凭证是一个 Pipeline 项目或条目。设置为 Global 后，凭证在这个项目和其后代项目中都有效。
- **System** - 如果要添加的凭证用于 Jenkins 实例本身与系统管理功能（例如电子邮件认证，代理连接等）交互。选择此选项将凭证的范围限制在单个对象。
###7 将凭证添加到所选凭证类型的相应字段中：
- Secret text - 复制 secret 文本并粘贴到 Secret 字段
- Username and password - 在相关字段中指定凭证的 Username 和 Password
- Secret file - 点击 File 字段旁边的 Choose file 按钮，选择 secret 文件并上传到 Jenkins
- SSH Username with private key - 在相关字段中指定凭证的 Username、Private Key 和可选的密码
注意：直接选择回车让您可以复制私钥的文本并将其粘贴到生成的密钥文本框中（原文：Choosing Enter directly allows you to copy the private key’s text and paste it into the resulting Key text box.）
- Certificate - 指定证书和可选的密码。上传 PKCS#12 证书。
- Docker Host Certificate Authentication - 将相应的详细信息复制并粘贴到客户端密钥，客户端证书和服务器 CA 证书字段中。
###8 在 ID 字段中，指定有意义的凭证 ID 值
例如，`jenkins-user-for-xyz-artifact-repository`。为了你的 Jenkins 实例上的用户，最好使用单一且一致的约定来指定凭证 ID。
注意：这个字段可选。如果不指定这个字段的值，Jenkins 会分配一个 globally unique ID（GUID）值作为这个凭证的 ID。注意，一旦凭证设置了 ID，不可更改。
###9 为凭证指定可选的 Description
###10 点击 OK 保存凭证