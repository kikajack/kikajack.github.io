[原文地址](https://jenkins.io/doc/book/pipeline/)

本章涵盖 Jenkins Pipeline 功能的所有建议方面，包括如何：

- [Pipeline 入门](https://jenkins.io/doc/book/pipeline/getting-started) - 如何通过 Blue Ocean、经典的 UI 或 SCM 定义 Jenkins Pipeline
- [创建并使用 Jenkinsfile](https://jenkins.io/doc/book/pipeline/jenkinsfile) - 涵盖了关于如何制作和构建 Jenkinsfile 的用例场景
- 使用 [branches 和 pull 请求](https://jenkins.io/doc/book/pipeline/multibranch)
- [使用 Docker 和 Pipeline](https://jenkins.io/doc/book/pipeline/docker) - 涵盖 Jenkins 如何调用代理/节点（来自 Jenkinsfile）上的 Docker 容器来构建 Pipeline 项目
- [通过 shared libraries 来扩展 Pipeline](https://jenkins.io/doc/book/pipeline/shared-libraries)
- 使用不同的 [开发工具](https://jenkins.io/doc/book/pipeline/development) 以便于创建 Pipeline
- 使用 [Pipeline 语法](https://jenkins.io/doc/book/pipeline/syntax) - 所有声明式 Pipeline 语法的综合参考

有关 Jenkins 用户手册中内容的概述，请参阅 [User Handbook overview](https://jenkins.io/doc/book/pipeline/getting-started)。
#1. Jenkins Pipeline 是什么
Jenkins Pipeline（或者简称为“Pipeline”）是一套插件，支持在 Jenkins 中实施和集成持续交付流水线。

持续交付（CD）流水线是将软件从版本控制发布到用户和客户的过程的自动化表达。对软件的每一次改变（在源代码控制中提交）都会在发布过程中经历一个复杂的过程。这个过程包括以可靠和可重复的方式构建软件，以及通过测试和部署的多个阶段来推进构建的软件（称为“构建”）。

Pipeline 提供了一套可扩展的工具，用于通过  Pipeline domain-specific language（DSL）语法将交付流水线“作为代码”建模。

Jenkins Pipeline 的定义写在称为 Jenkinsfile 的文本文件中，这个文件可以提交到项目的代码控制仓库。这是“Pipeline-as-code”的基础。 将 CD 流水线作为应用程序的一部分进行版本控制，并像任何其他代码一样进行审查。

创建 Jenkinsfile 文件并且提交到版本控制有下面几个好处：

- 对所有的 branches 和 pull 请求自动创建 Pipeline。
- Pipeline 上的代码审查/迭代（along with the remaining source code）。
- 审核追踪 Pipeline
- Pipeline 的单一真实来源，可由项目的多个成员查看和编辑。

虽然用于定义 Pipeline 的语法无论是在 Web UI 中还是在 Jenkinsfile 中是相同的，但在 Jenkinsfile 中定义 Pipeline 并提交到源代码控制中通常被认为是最佳实践。
##声明式与脚本式 Pipeline 语法
Jenkinsfile 文件可以使用两种语法：声明式和脚本式。

声明式与脚本式 Pipeline 的构造完全不同。声明式 Pipeline 是 Jenkins Pipeline 的更新特性，其中：

- 通过脚本 Pipeline 语法提供更丰富的语法功能
- 用来更容易的读写 Pipeline 代码

然而，写入Jenkins文件的许多单独的语法组件（或“步骤”）对于声明式和脚本式 Pipeline 都是通用的。在下面的 Pipeline 概念和语法概述中详细了解这两种类型的语法。
#2. 为何使用 Pipeline
Jenkins 从根本上说是一个支持多种自动化模式的自动化引擎。Pipeline 在 Jenkins 上增加了一套强大的自动化工具，支持从简单的持续集成到全面的 CD 流水线的用例。通过对一系列相关任务建模，用户可以利用 Pipeline 的许多功能：

- Code：Pipeline 以代码实现，通常会被检入到源代码管理中，使团队能够编辑，审阅和迭代其部署流水线。
- Durable：Pipeline 可以在 Jenkins master 被计划重启或意外重启之后继续存在。
- Pausable：Pipeline 在继续运行之前，可以选择停止并等待人员的输入或批准。
- Versatile：Pipeline 支持复杂的真实 CD 需求，包括 fork/join、loop 及并发任务。
- Extensible：Pipeline 插件支持对其 DSL 的自定义扩展，以及与其他插件集成的多个选项。

虽然 Jenkins 一直允许将自由式工作的基本形式连接起来执行连续任务，Pipeline 使这一概念成为 Jenkins 的一流公民。

基于可扩展性这个 Jenkins 的核心价值，Pipeline 也可以由管道共享库用户和插件开发人员扩展。

下面的流程图是一个在 Jenkins Pipeline 中轻松建模的 CD 场景示例：

![realworld-pipeline-flow](http://img.blog.csdn.net/20180302225512534?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#3. Pipeline 概念
下面的概念是 Jenkins Pipeline 的关键方面，与 Pipeline 语法紧密相关（参阅下面的概述）。
##3.1 Pipeline
Pipeline 是一个用户定义的 CD 流水线模式。Pipeline 代码定义了通常包含构建、测试和发布步骤的完整的构建过程。

同时，pipeline 代码块也是声明式 Pipeline 语法的关键特性。
##3.2 Node
node 是一个机器，它是 Jenkins 环境的一部分，并且能够执行 Pipeline。

同时，node 代码块也是脚本式 Pipeline 语法的关键特性。
##3.3 Stage
Stage 块定义了在整个 Pipeline 中执行的概念上不同的任务子集（例如“构建”，“测试”和“部署”阶段），许多插件使用它来可视化或呈现 Jenkins 管道状态/进度。
##3.4 Step
一项任务。从根本上讲，一个步骤告诉 Jenkins 在特定时间点（或过程中的“步骤”）要做什么。例如，使用 sh step：`sh 'make'` 可以执行 `make` 这个 shell 命令。 当一个插件扩展了 Pipeline DSL 时，通常意味着该插件已经实现了一个新的步骤。
#4. Pipeline 语法概述
下面的 Pipeline 代码框架说明了声明式与脚本式 Pipeline 语法之间的基本区别。

注意，上述的阶段（stage）和步骤（step）都是声明式与脚本式 Pipeline 语法的常见元素。
##4.1 声明式 Pipeline 基础
在声明式 Pipeline 语法中，pipeline 块定义了贯穿 Pipeline 的所有的工作。
```
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any  //在任何可用的代理上执行这个 Pipeline 或其任意 stage
    stages {
        stage('Build') { //定义 "Build" stage
            steps {
                // 执行和 "Build" stage 相关的 step
            }
        }
        stage('Test') { // 定义  "Test" stage
            steps {
                // 执行和 "Test" stage 相关的 step
            }
        }
        stage('Deploy') { 
            steps {
                // 
            }
        }
    }
}
```
##4.2 脚本式 Pipeline 基础
在脚本式 Pipeline 语法中，一个或多个 node 块执行贯穿整个 Pipeline 的核心工作。虽然这不是脚本式 Pipeline 语法的强制性要求，但将管道 work 限制在 node 块内部会做两件事情：

- 通过将 item 添加到 Jenkins 队列来运行块中包含的 step。只要执行程序在 node 上空闲，这些步骤就会运行。
- 创建一个工作空间（特定于该特定 Pipeline 的目录），其中可以对从源代码管理检出的文件执行工作。
注意：根据不同的 Jenkins 配置，部分 workspaces 不会在一段不活动时间之后自动清理。查看与 [JENKINS-2111](https://issues.jenkins-ci.org/browse/JENKINS-2111) 相关的标记和讨论以获取更多信息。
```
Jenkinsfile (Scripted Pipeline)
node {  //在任何可用的代理上执行这个 Pipeline 或其任意 stage
    stage('Build') { //定义 "Build" stage
        // 执行和 "Build" stage 相关的 step
    }
    stage('Test') { 
        // 
    }
    stage('Deploy') { 
        // 
    }
}
```
脚本式 Pipeline 中，stage 块是可选的。但是，在脚本式 Pipeline 中实现 stage 块可以更清晰地显示 Jenkins UI 中每个阶段的 tasks/steps。
#5. Pipeline 示例
这个 `Jenkinsfile` 的例子使用声明式 Pipeline 语法：
```
Jenkinsfile (Declarative Pipeline)
pipeline { 
    agent any 
    stages {
        stage('Build') { 
            steps { 
                sh 'make' 
            }
        }
        stage('Test'){
            steps {
                sh 'make check'
                junit 'reports/**/*.xml' 
            }
        }
        stage('Deploy') {
            steps {
                sh 'make publish'
            }
        }
    }
}
```
上面例子对应的脚本式 Pipeline 语法：
```
Jenkinsfile (Scripted Pipeline)
node { 
    stage('Build') { 
        sh 'make' 
    }
    stage('Test') {
        sh 'make check'
        junit 'reports/**/*.xml' 
    }
    stage('Deploy') {
        sh 'make publish'
    }
}
```

更多关于 Pipeline 语法的资料参考 pPipeline Syntax page](https://jenkins.io/doc/book/pipeline/syntax)。

1. [Domain-specific language](https://en.wikipedia.org/wiki/Domain-specific_language)
2. [Source control management](https://en.wikipedia.org/wiki/Version_control)
3. [Single source of truth](https://en.wikipedia.org/wiki/Single_source_of_truth)
4. Additional plugins have been used to implement complex behaviors utilizing Freestyle Jobs such as the Copy Artifact, Parameterized Trigger, and Promoted Builds plugins
5. [GitHub Organization Folder plugin](https://plugins.jenkins.io/github-organization-folder)
6. [Blue Ocean](https://jenkins.io/doc/book/blueocean), [Pipeline: Stage View plugin](https://plugins.jenkins.io/pipeline-stage-view)