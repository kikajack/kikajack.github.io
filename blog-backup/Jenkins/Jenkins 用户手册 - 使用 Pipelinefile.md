[原文地址](https://jenkins.io/doc/book/pipeline/jenkinsfile/)

本节基于 [入门章节](http://blog.csdn.net/kikajack/article/details/79428889) 中的信息，并介绍了更多有用的步骤，常见模式，并演示了一些不重要的 Jenkinsfile 示例。

创建 Jenkinsfile 并上传到源代码控制，会带来几个直接的好处：

- 基于 Pipeline 的代码审查和迭代
- 审核追踪 Pipeline
- Pipeline 的来源单一真实，可以由项目的多个成员查看和编辑

Pipeline 支持两种语法，声明式和脚本式。这两种方法都支持构建持续交付流水线，都可以通过 web UI 或 Jenkinsfile 文件来定义 Pipeline（通常认为创建 Jenkinsfile 文件并上传到源代码控制仓库是最佳实践）。
#1. 创建 Jenkinsfile
入门章节中讨论过如何在 SCM 中定义 Pipeline，Jenkinsfile 就是一个包含对 Jenkins Pipeline 定义的文本文件，会上传到版本控制中。下面的 Pipeline 实现了基本的 3 段持续交付流水线。
声明式 Pipeline：
```
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
```
对应的脚本式 Pipeline：
```
Jenkinsfile (Scripted Pipeline)
node {
    stage('Build') {
        echo 'Building....'
    }
    stage('Test') {
        echo 'Building....'
    }
    stage('Deploy') {
        echo 'Deploying....'
    }
}
```
注意，所有的 Pipeline 都会有这三个相同的 stage，可以在所有项目的一开始就定义好它们。下面演示在 Jenkins 的测试安装中创建和执行一个简单的 Pipeline。

假设项目已经设置好了源代码控制仓库，并且已经按照入门章节的描述在 Jenkins 中定义好了 Pipeline。

使用文本编辑器（最好支持 Groovy 语法高亮显示），在项目根目录中创建 Jenkinsfile。

上面的声明式 Pipeline 示例包含了实现一个持续交付流水线所需的最少步骤。必选指令 [agent](https://jenkins.io/doc/book/pipeline/syntax/#agent) 指示 Jenkins 为 Pipeline 分配执行程序和工作空间。没有 agent 指令的话，声明式 Pipeline 无效，无法做任何工作！默认情况下 agent 指令会确保源代码仓库已经检出，并且可用于后续步骤。

stage 和 step 指令在声明式 Pipeline 中也是必须的，用于指示 Jenkins 执行什么及在哪个 stage 中执行。

对于脚本式 Pipeline 的更高级用法，上面的示例节点是至关重要的第一步，因为它为 Pipeline 分配了一个执行程序和工作空间。如果没有 node，Pipeline 不能做任何工作！在 node 内，业务的第一阶段是检出此项目的源代码。由于 Jenkinsfile 是直接从源代码控制中提取的，因此 Pipeline 提供了一种快速简单的方法来访问源代码的正确版本：
```
Jenkinsfile (Scripted Pipeline)
node {
    checkout scm 
    /* .. snip .. */
}
```
这个 checkout 步骤会从源代码控制中检查代码，scm 是特殊变量，它指示运行检出步骤，复制触发了这次 Pipeline 运行的指定版本。
##1.1 Build 构建
对于大多数项目，Pipeline 流水线的第一步是构建。通常在这一步中，源代码会被组装、编译或打包。Jenkinsfile 不是现有构建工具比如 GNU/Make、Maven、Gradle 等的替代品，而是可以被视为粘合层来将项目开发生命周期的多个阶段（构建，测试，部署等）绑定在一起。

Jenkins 有一系列插件可以调用几乎所有常用的构建工具，这个例子中会在 shell （sh）步骤中调用 make。`sh` 步骤假设系统是基于 Unix/Linux，对于基于 Windows 的系统，可以使用 `bat`。

声明式 Pipeline：
```
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'make' 
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true 
            }
        }
    }
}
```
对应的脚本式 Pipeline：
```
Jenkinsfile (Scripted Pipeline)
node {
    stage('Build') {
        sh 'make' // 调用 make 命令
        archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true  // 匹配并保存文件供以后检索
    }
}
```
- `sh` 步骤调用 make 命令，并且只有在 make 命令返回代码是 0 时才会继续。任何非 0 的返回代码都会使 Pipeline 失败。
- `archiveArtifacts` 捕捉构建时匹配模式 `**/target/*.jar` 的文件并保存到 Jenkins 主进程供以后检索。
>Jenkins 的文件存档不能代替 Artifactory 或 Nexus 等外部文件存储库，应仅用于基本报告和文件存档。
##1.2 Test 测试
自动化测试是任何成功的持续交付流程的重要部分。为此，Jenkins 通过一系列插件提供测试记录、报告和可视化功能。在基本层面上，当测试失败时，Jenkins 会记录失败并在 Web UI 中可视化。下面的例子使用由 JUnit 插件提供的 junit step。

下面的例子中，如果测试失败，Pipeline 会被标记为不稳定“unstable”，web UI 中会标记黄球。根据记录的测试报告，Jenkins 还可以提供历史趋势分析和可视化。

声明式 Pipeline：
```
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                /* 测试失败时，`make check` 返回非 0 值
                *  尽管如此，使用 `true` 使 Pipeline 继续
                */
                sh 'make check || true' 
                junit '**/target/*.xml' 
            }
        }
    }
}
```
对应的脚本式 Pipeline：
```
Jenkinsfile (Scripted Pipeline)
node {
    /* .. snip .. */
    stage('Test') {
        /* `make check` returns non-zero on test failures,
         * using `true` to allow the Pipeline to continue nonetheless
         */
        sh 'make check || true' 
        junit '**/target/*.xml' 
    }
    /* .. snip .. */
}
```
- `sh 'make check || true'`：使用内联 shell 条件（`sh 'make || true'`）可确保 `sh` 步骤总是得到值为 0 的退出代码，从而为 junit 步骤提供捕获和处理测试报告的机会。下面的处理失败部分将更详细地介绍这方面的其他方法。
- `junit '**/target/*.xml'`：junit 捕获并关联与包含模式匹配的 JUnit XML 文件（`**/target/*.xml`）
##1.3 Deploy 部署
根据项目或组织的需求，部署可能意味着各种步骤，可能是将构建的文件发布到 Artifactory 服务器，或将代码推送到生产系统等。

在 Pipeline 示例的这一步，构建和测试阶段都已经成功完成了。实质上，“部署”阶段只会在前一阶段成功完成的情况下执行，否则管道将提早退出。

声明式 Pipeline：
```
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any

    stages {
        stage('Deploy') {
            when {
              expression {
                currentBuild.result == null || currentBuild.result == 'SUCCESS' // 判断是否发生测试失败
              }
            }
            steps {
                sh 'make publish'
            }
        }
    }
}
```
对应的脚本式 Pipeline：
```
Jenkinsfile (Scripted Pipeline)
node {
    /* .. snip .. */
    stage('Deploy') {
        if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {  // 判断是否发生测试失败
            sh 'make publish'
        }
    }
    /* .. snip .. */
}
```
- 访问 `currentBuild.result` 变量使得 Pipeline 可以判断是否发生测试失败的情况。失败时这个变量的值是 UNSTABLE。

假设 Jenkins Pipeline 示例中的所有内容都已成功执行，每个成功的 Pipeline 运行都将存档关联的构建文件、测试结果的报告以及 Jenkins 中完整的控制台输出。

>脚本式 Pipeline 可以报考测试条件（上面例子中有）、循环、try/catch/finally 代码块和函数。下一部分会讲解脚本式 Pipeline 高级语法。
#2. 使用 Jenkinsfile
以下部分提供详细信息用于处理：

- Jenkinsfile 文件中的具体 Pipeline 语法
- Pipeline 语法的特性和函数，这对于构建应用程序或 Pipeline 项目非常重要。
##2.1 字符串插值（interpolation）
Jenkins Pipeline 使用与 Groovy 相同的规则进行字符串插值。Groovy的字符串插值支持可能会让许多这门语言的新手感到困惑。虽然 Groovy 支持使用单引号或双引号来声明一个字符串，例如：
```
def singlyQuoted = 'Hello'
def doublyQuoted = "World"
```
只有双引号字符串才支持基于美元符号（$）的字符串插值，例如：
```
def username = 'Jenkins'
echo 'Hello Mr. ${username}'
echo "I said, Hello Mr. ${username}"
```
结果是：
```
Hello Mr. ${username}
I said, Hello Mr. Jenkins
```
了解如何使用字符串插值对于使用 Pipeline 的一些更高级功能至关重要。
##2.2 使用环境变量
Jenkins Pipeline 通过全局变量 `env` 暴露环境变量，该变量可以在 Jenkinsfile 中的任何地方使用。假设 Jenkins 主机在 `localhost:8080` 运行，在 Jenkins Pipeline 中可访问的环境变量的完整列表记录在 `localhost:8080/pipeline-syntax/globals#env` 中，包括：

- **BUILD_ID**：当前的构建 ID，对于在 Jenkins 1.597 以上的版本中创建的构建，与 `BUILD_NUMBER` 相同。
- **JOB_NAME**：当前构建的项目名，例如“foo”或“foo/bar”。
- **JENKINS_URL**：完整的 Jenkins URL，例如 `example.com:port/jenkins/`（注意：只有在“System Configuration”中设置了 Jenkins URL 后才可用）。

引用或使用这些环境变量可以像访问 Groovy [Map](http://groovy-lang.org/syntax.html#_maps) 中的任何键一样，例如：

声明式 Pipeline：
```
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
            }
        }
    }
}
```
对应的脚本式 Pipeline：
```
Jenkinsfile (Scripted Pipeline)
node {
    echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
}
```
###2.2.1 设置环境变量
在 Jenkins Pipeline 中设置环境变量时，声明式和脚本式 Pipeline 中完全不同。

声明式 Pipeline 支持 [environment](https://jenkins.io/doc/book/pipeline/syntax/#environment) 指令，而脚本式 Pipeline 中必须使用 `withEnv` 步骤。

声明式 Pipeline：
```
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    environment { // 顶级上下文中的 environment 指令会用于 Pipeline 中的所有步骤
        CC = 'clang'
    }
    stages {
        stage('Example') {
            environment { // stage 中的 environment 指令仅用于当前 stage 中的步骤
                DEBUG_FLAGS = '-g'
            }
            steps {
                sh 'printenv'
            }
        }
    }
}
```
对应的脚本式 Pipeline：
```
Jenkinsfile (Scripted Pipeline)
node {
    /* .. snip .. */
    withEnv(["PATH+MAVEN=${tool 'M3'}/bin"]) {
        sh 'mvn -B verify'
    }
}
```
##2.3 处理凭证
[Jenkins 中配置的凭证](https://jenkins.io/doc/book/using/using-credentials#configuring-credentials) 可以在 Pipeline 中处理用于临时用途。关于在 Jenkins 中使用凭证的资料请参考 [Using credentials](https://jenkins.io/doc/book/using/using-credentials) 页面。
###2.3.1 处理密码文本、用户名密码和密码文件
声明式 Pipeline 语法使用支持密码文本、用户名密码和密码文件的 `credentials()` 辅助函数（在 `environment` 指令中使用）。如果要处理其他类型的凭证，参考下一部分。
####2.3.1.1 密码文本（Secret text）
下面的 Pipeline 代码示例显示了如何使用和密码文本凭证相关的环境变量创建 Pipeline。

在这个例子中，两个密码文本凭证被分配到不同的环境变量来访问 Amazon Web Services (AWS)。这些凭证将在 Jenkins 中以其各自的证书 ID `jenkins-aws-secret-key-id` 和 `jenkins-aws-secret-access-key` 进行配置。
```
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent {
        // 定义代理详情
    }
    environment {
        AWS_ACCESS_KEY_ID     = credentials('jenkins-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-secret-access-key')
    }
    stages {
        stage('Example stage 1') {
            steps {
                // 
            }
        }
        stage('Example stage 2') {
            steps {
                // 
            }
        }
    }
}
```
可以引用这两个环境变量凭证（在这个 Pipeline 的 `environment` 指令中定义），在这个阶段（stege）的步骤（step）中使用 `$AWS_ACCESS_KEY_ID` 和 `$AWS_SECRET_ACCESS_KEY`。例如，可以授权 AWS 使用和这个凭证变量相关的密码文本凭证。
要维护这些证书的安全性和匿名性，如果你尝试从 Pipeline 内检索这些凭证变量的值（例如，`echo $AWS_SECRET_ACCESS_KEY`），则 Jenkins 仅返回值 `****` 以防止秘密信息写入到控制台输出和任何日志。凭证 ID 本身的任何敏感信息（例如用户名）在 Pipeline 运行的输出中也以 `****` 的形式返回。
在这个 Pipeline 示例中，分配到两个 `AWS_...` 环境变量的凭证都对 Pipeline 有全局作用域，因此这些凭证变量可以在这个阶段的每个步骤中使用。然而，如果这个 Pipeline 中的 `environment` 指令被移到某个特定的 stage 中（参考下面用户名和密码 Pipeline 示例），那这些  `AWS_...` 环境变量将只能在这个 stage 中使用。
####2.3.1.2 用户名密码
下面的 Pipeline 代码片段是关于如何创建使用与用户名密码相关的环境变量的 Pipeline 示例。

在此示例中，用户名和密码凭证分配给环境变量，以访问组织中常用帐户或团队中的 Bitbucket 仓库；这些凭证将在 Jenkins 中用凭证 ID `jenkins-bitbucket-common-creds` 进行配置。

在 `environment` 指令中设置凭证环境变量时：
```
environment {
    BITBUCKET_COMMON_CREDS = credentials('jenkins-bitbucket-common-creds')
}
```
这实际上设置了三个环境变量：

- `BITBUCKET_COMMON_CREDS` - 包含用冒号分割的用户名和密码，格式为 `username:password`
- `BITBUCKET_COMMON_CREDS_USR` - 只包含用户名的额外变量
- `BITBUCKET_COMMON_CREDS_PSW` - 只包含密码的额外变量

>按照惯例，环境变量的变量名通常以大写形式指定，单个单词由下划线分隔。但是，也可以使用小写字符指定任何合法的变量名称。请记住，由`credentials()` 方法创建的附加环境变量将始终附加 `_USR` 和 `_PSW`（下划线后跟三个大写字母）。

下面的代码片段显示了完整的 Pipeline：
```
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent {
        // Define agent details here
    }
    stages {
        stage('Example stage 1') {
            environment {
                BITBUCKET_COMMON_CREDS = credentials('jenkins-bitbucket-common-creds')
            }
            steps {
                // 
            }
        }
        stage('Example stage 2') {
            steps {
                // 
            }
        }
    }
}
```
凭证环境变量（在这个 Pipeline 的 `environment` 指令中定义的）可以在所有阶段的每个 step 中用下面的语法引用：

- `$BITBUCKET_COMMON_CREDS`
- `$BITBUCKET_COMMON_CREDS_USR`
- `$BITBUCKET_COMMON_CREDS_PSW`

例如，在这里可以用分配给这些凭证变量的用户名和密码对 Bitbucket 进行身份验证。

为了维护这些凭证的安全性和匿名性，如果你尝试在 Pipeline 内检索这些凭证变量的值，则以上密码文本示例中描述的相同行为也适用于这些用户名和密码凭证变量类型（即返回 `****`）。

在这个 Pipeline 示例中，分配给这三个环境变量的凭证的作用域仅限于当前阶段。然而，如果将当前 Pipeline 中的 `environment` 指令移到顶级上下文中，那就可以在所有的 stage 阶段中使用了。
####2.3.1.3 密码文件（Secret files）
就管道而言，密码文件的处理方式与上文中的密码文本完全相同。

密码文本和密码文件凭证之间的唯一区别：对于密码文本，凭证本身直接输入 Jenkins，而对于密码文件，凭证最初存储在文件中，然后上传到 Jenkins。

与密码文本不同，密码文件符合以下凭证：

- 过于笨重，无法直接传到 Jenkins
- 二进制格式，例如 GPG 文件
###2.3.2 其他类型的凭证
如果需要在 Pipeline 中为密码文本、用户名密码或密码文件以外的任何内容设置凭据 - 即 SSH 密钥或证书，则可以通过 Jenkins 的 classic UI 使用 Jenkins 的片段生成器功能。


要为你的项目使用 Snippet Generator 片段生成器：

1. 在 Jenkins 的首页（例如 Jenkins classic UI 的 Dashboard）点击你的 Pipeline 项目名。
2. 点击左侧的 **Pipeline Syntax** 并确保 **Snippet Generator** 链接在左上角以粗体显示。（如果没有，请点击链接）
3. 从 **Sample Step** 字段中，选择 **withCredentials: Bind credentials to variables**。
4. 在 **Bindings** 中点击 **Add**，从下拉列表中选择：
- **SSH User Private Key** - 处理 SSH 公私钥对凭证，可以指定为：
    - **Key File Variable** - 要被绑定到这些凭证的环境变量的名字。Jenkins 实际上将此临时变量分配给 SSH 公私钥对认证过程中所需的私钥文件的安全位置。
    - **Passphrase Variable** - 可选，要被绑定到与 SSH 公私钥对关联的密码的环境变量的名称。
    - **Username Variable** - 可选，要被绑定到与 SSH 公私钥对关联的用户名的环境变量的名称。
    - **Credentials** - 选择存储在 Jenkins 中的 SSH 公私钥凭证。 该字段的值是由 Jenkins 写入生成的代码片段中的凭证 ID。
- **Certificate** - 处理 PKCS#12 证书，可以指定为：
    - **Keystore Variable** - 要被绑定到这些凭据的环境变量的名称。Jenkins 实际上将此临时变量分配给证书认证过程中所需的证书密钥库的安全位置。
    - **Password Variable** - 可选，要被绑定到和证书相关的密码的环境变量的名字。
    - **Alias Variable** - 可选，要被绑定到和证书相关的唯一别名的环境变量的名字。
    - **Credentials** - 选择 Jenkins 中存储的证书凭证。 该字段的值是由 Jenkins 写入生成的代码片段中的凭证 ID。
- **Docker client certificate** - 处理 Docker 主机证书认证。

5. 点击 **Generate Pipeline Script** ，Jenkins 会为你指定的凭证生成，可以复制粘贴到你的声明式或脚本式 Pipeline 代码中。
注意：

- **Credentials** 字段显示了 Jenkins 中配置的凭证名称。然而，在点击 **Generate Pipeline Script** 之后，这些值会转换为凭证 ID。
- 要在一个 `withCredentials( ... ) { ... }` Pipeline 步骤的代码片段整合多个凭证，参考下一小节。

**SSH User Private Key 示例**
```
withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'jenkins-ssh-key-for-abc', \
                                             keyFileVariable: 'SSH_KEY_FOR_ABC', \
                                             passphraseVariable: '', \
                                             usernameVariable: '')]) {
  // some block
}
```
The optional passphraseVariable and usernameVariable definitions can be deleted in your final Pipeline code.

**Certificate 示例**
```
withCredentials(bindings: [certificate(aliasVariable: '', \
                                       credentialsId: 'jenkins-certificate-for-xyz', \
                                       keystoreVariable: 'CERTIFICATE_FOR_XYZ', \
                                       passwordVariable: 'XYZ-CERTIFICATE-PASSWORD')]) {
  // some block
}
```
可选的 `aliasVariable` 和 `passwordVariable` 变量定义可以在你最终的 Pipeline 代码中删除。

下面的代码片段是完整的 Pipeline 示例，包括了上面提到的 **SSH User Private Key** 和 **Certificate** 片段：
```
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent {
        // define agent details
    }
    stages {
        stage('Example stage 1') {
            steps {
                withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'jenkins-ssh-key-for-abc', \
                                                             keyFileVariable: 'SSH_KEY_FOR_ABC')]) {
                  // ①
                }
                withCredentials(bindings: [certificate(credentialsId: 'jenkins-certificate-for-xyz', \
                                                       keystoreVariable: 'CERTIFICATE_FOR_XYZ', \
                                                       passwordVariable: 'XYZ-CERTIFICATE-PASSWORD')]) {
                  // ②
                }
            }
        }
        stage('Example stage 2') {
            steps {
                // ③
            }
        }
    }
}
```
①：在这个步骤中，可以通过 `$SSH_KEY_FOR_ABC` 引用凭证环境变量。例如，可以使用其已经配置的 SSH 公私钥对凭证向 ABC 应用授权，该应用的 **SSH User Private Key** 文件已经分配到了 `$SSH_KEY_FOR_ABC` 变量。
②：在这个步骤中，可以通过 `$CERTIFICATE_FOR_XYZ` 和 `$XYZ-CERTIFICATE-PASSWORD` 引用凭证环境变量。例如，可以使用其已经配置的证书凭证向 XYZ 应用授权，其证书的密钥库文件和密码分别分配给变量 `$CERTIFICATE_FOR_XYZ` 和 `$XYZ-CERTIFICATE-PASSWORD`。
③：在这个 Pipeline 示例中，分配给 `$SSH_KEY_FOR_ABC`、`$CERTIFICATE_FOR_XYZ` 和
`$XYZ-CERTIFICATE-PASSWORD` 环境变量的凭证仅作用域对应的 `withCredentials( ... ) { ... } ` 步骤。因此无法在第二个阶段中使用。

为了维护这些凭证的安全性和匿名性，如果你尝试在 `within these withCredentials( ... ) { ... }` 内检索这些凭证变量的值，则以上密码文本示例中描述的相同行为也适用于这些用户名和密码凭证变量类型（即返回 `****`）。

>- 在代码段生成器 **Snippet Generator** 中使用 **Sample Step** 字段的 **withCredentials: Bind credentials to variables** 选项时，只能从凭证字段列表中选择当前 Pipeline 项目有权访问的凭证。虽然可以手动为 Pipeline 编写 `withCredentials( ... ) { ... }` 步骤，但建议使用 **Snippet Generator** 以避免指定超出此 Pipeline 项目范围的凭据， 这在运行时会使这个步骤失败。
- 也可以使用 **Snippet Generator** 来生成 `withCredentials( ... ) { ... }` 来处理密码文本、用户名密码和密码文件。然而，如果只需要处理这些类型的凭证，建议为了 Pipeline 代码的可读性使用上一章节描述的相关过程。
####2.3.2.1 一个步骤中整合多个凭证
使用 **Snippet Generator** 时，可以通过下面步骤在一个 `withCredentials( ... ) { ... }` 步骤中使用多个凭证：

1. 在 Jenkins 首页（例如 classic UI 的 Dashboard），点击你的 Pipeline 项目名。
2. 点击左侧的 **Pipeline Syntax** 并确保 **Snippet Generator** 链接在左上角变成粗体（若没有，点击这个链接）。
3. 在 **Sample Step** 字段中选择 **withCredentials: Bind credentials to variables**。
4. 点击 **Bindings** 下面的 **Add**。
5. 通过下列列表选择要添加到 `withCredentials( ... ) { ... }` 步骤的凭证类型。
6. 指定凭证的 **Bindings** 详情。参考上面部分的凭证类型。
7. 为每个要添加到 `withCredentials( ... ) { ... }` 步骤的凭证重复 **Click Add …**。
8. 点击 **Generate Pipeline Script** 生成用于 `withCredentials( ... ) { ... }` 步骤的最终片段。
##2.4 处理参数
声明式 Pipeline 支持开箱即用的参数，允许 Pipeline 通过 [parameters 指令](https://jenkins.io/doc/book/pipeline/syntax/#parameters) 在运行时接受用户指定的参数。使用脚本式 Pipeline 配置参数是通过 `properties` 步骤完成的，该步骤可以在代码片段生成器中找到。

如果通过 **Build with Parameters** 选项来配置你的流水线接受参数，那些参数可以作为 `params` 变量的成员来访问。

假设在 Jenkinsfile 中已经配置了一个名为“Greeting”的字符串参数，可以通过 `${params.Greeting}` 来访问该参数：

声明式 Pipeline：
```
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    parameters {
        string(name: 'Greeting', defaultValue: 'Hello', description: 'How should I greet the world?')
    }
    stages {
        stage('Example') {
            steps {
                echo "${params.Greeting} World!"
            }
        }
    }
}
```
对应的脚本式 Pipeline：
```
Jenkinsfile (Scripted Pipeline)
properties([parameters([string(defaultValue: 'Hello', description: 'How should I greet the world?', name: 'Greeting')])])

node {
    echo "${params.Greeting} World!"
}
```
##2.5 处理失败
声明式 Pipeline 默认通过 [post 部分](https://jenkins.io/doc/book/pipeline/syntax/#post) 支持强大的故障处理，post 中支持 `always`、`unstable`、`success`、`failure` 和 `changed` 等各种不同的结果状态（post conditions）。教程的 [Pipeline 语法部分](https://jenkins.io/doc/book/pipeline/jenkinsfile/#syntax) 提供了关于如何使用各种 post condition 的更多细节。

声明式 Pipeline：
```
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'make check'
            }
        }
    }
    post {
        always {
            junit '**/target/*.xml'
        }
        failure {
            mail to: team@example.com, subject: 'The Pipeline failed :('
        }
    }
}
```
对应的脚本式 Pipeline：
```
Jenkinsfile (Scripted Pipeline)
node {
    /* .. snip .. */
    stage('Test') {
        try {
            sh 'make check'
        }
        finally {
            junit '**/target/*.xml'
        }
    }
    /* .. snip .. */
}
```
脚本式 Pipeline 依赖于 Groovy 的内置 try/catch/finally 语义来处理 Pipeline 执行过程中的失败。

上面的一个示例中，`sh` 步骤被设置为永不返回非 0 代码（`sh 'make check || true'`）。这种方法虽然有效，但意味着以下阶段需要检查 `currentBuild.result` 以确定是否存在测试失败。

处理这个问题的另一种方法是通过使用一系列 `try/finally` 代码块保留 Pipeline 中失败的早期退出行为，同时仍然给 junit 提供捕获测试报告的机会。
##2.6 使用多个代理
前面的所有例子都只使用了一个代理。这意味着 Jenkins 会在任何可用的地方分配一个执行者，而不管它是如何标记或配置的。这种行为可以被重写，而且 Pipeline 允许在同一个 Jenkinsfile 中使用 Jenkins 环境中的多个代理，这对于更高级的用例（例如在多个平台上执行构建/测试）可能会有所帮助。

在下面的例子中，将在一个代理上执行“构建”阶段，而在“测试”阶段期间，将分别在两个后续代理上标记“linux”和“windows”使用这个构建的结果。

声明式 Pipeline：
```
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent none
    stages {
        stage('Build') {
            agent any
            steps {
                checkout scm
                sh 'make'
                stash includes: '**/target/*.jar', name: 'app'  // ①
            }
        }
        stage('Test on Linux') {
            agent {  // ②
                label 'linux'
            }
            steps {
                unstash 'app' // ③
                sh 'make check'
            }
            post {
                always {
                    junit '**/target/*.xml'
                }
            }
        }
        stage('Test on Windows') {
            agent {
                label 'windows'
            }
            steps {
                unstash 'app'
                bat 'make check' // ④
            }
            post {
                always {
                    junit '**/target/*.xml'
                }
            }
        }
    }
}
```
对应的脚本式 Pipeline：
```
Jenkinsfile (Scripted Pipeline)
stage('Build') {
    node {
        checkout scm
        sh 'make'
        stash includes: '**/target/*.jar', name: 'app' // ①
    }
}

stage('Test') {
    node('linux') { // ②
        checkout scm
        try {
            unstash 'app' // ③
            sh 'make check'
        }
        finally {
            junit '**/target/*.xml'
        }
    }
    node('windows') {
        checkout scm
        try {
            unstash 'app'
            bat 'make check' // ④
        }
        finally {
            junit '**/target/*.xml'
        }
    }
}
```
①：`stash` 这一步允许捕获匹配模式（`**/target/*.jar`）的文件以在同一 Pipeline 内重用。一旦 Pipeline 完成执行，隐藏的文件将从 Jenkins 主文件中删除。
②：`agent/node` 中的参数允许任何有效的 Jenkins 标签表达式。更多详细信息，请参阅 [管道语法部分](https://jenkins.io/doc/book/pipeline/syntax/)。
③：`unstash` 将从主 Jenkins 中检索名为“stash”并放入 Pipeline 的当前工作空间（unstash will retrieve the named "stash" from the Jenkins master into the Pipeline’s current workspace）。
④：`bat` 脚本允许在基于 Windows 的平台上执行批处理脚本。
##2.7 可选的步骤参数
Pipeline 遵循 Groovy 语言约定，允许在方法参数周围省略括号。

许多 Pipeline 步骤还使用命名参数（named-parameter）语法作为在 Groovy 中创建 Map 的简写，使用的语法为 `[key1：value1，key2：value2]`。Making statements like the following functionally equivalent:
```
git url: 'git://example.com/amazing-project.git', branch: 'master'
git([url: 'git://example.com/amazing-project.git', branch: 'master'])
```
为方便起见，当调用只有一个参数的步骤（或只有一个必需参数）时，参数名称可以省略，例如：
```
sh 'echo hello' /* short form  */
sh([script: 'echo hello'])  /* long form */
```
##2.8 高级脚本式 Pipeline
脚本式 Pipeline 是一种基于 Groovy 的领域特定语言，大多数 Groovy 语法都可以不用修改直接用于脚本式 Pipeline。
###2.8.1 并行执行
上一个示例以线性顺序跨两个不同平台的测试执行。在实践中，如果执行 `make check` 需要30分钟完成，那么“测试”阶段现在需要 60 分钟才能完成！

幸运的是，Pipeline 具有内置的功能，可以通过适当命名的并行步骤并行执行部分脚本式 Pipeline。

重构上面的例子以使用并行步骤：
```
Jenkinsfile (Scripted Pipeline)
stage('Build') {
    /* .. snip .. */
}

stage('Test') {
    parallel linux: {
        node('linux') {
            checkout scm
            try {
                unstash 'app'
                sh 'make check'
            }
            finally {
                junit '**/target/*.xml'
            }
        }
    },
    windows: {
        node('windows') {
            /* .. snip .. */
        }
    }
}
```
假设在 Jenkins 环境中资源足够，它们现在将并行执行“linux”和“windows”标记节点上的测试。