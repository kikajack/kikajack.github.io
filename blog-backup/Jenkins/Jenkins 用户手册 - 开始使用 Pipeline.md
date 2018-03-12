[原文地址](https://jenkins.io/doc/book/pipeline/getting-started/)

如前所述，Jenkins Pipeline 是一套插件，支持将持续交付流水线实施并集成到 Jenkins 中。Pipeline 提供了一组可扩展的工具，用于通过 Pipeline DSL 将简单到复杂的交付管道“作为代码”进行建模。

本章讲述如何入门并在 Jenkins 中创建你自己的 Pipeline 项目，并介绍创建和保存 Jenkinsfile 的多种方式。
#1. 先决条件
要使用 Jenkins Pipeline 需要：

- Jenkins 2.x 或更高版本（最早支持 1.642.3 的版本，但不推荐使用）
- 已经作为推荐插件安装好的 Pipeline 插件（已经在 Jenkins 的安装后设置向导中配置好）

关于安装和管理插件的更多资料请参考 [Managing Plugins](https://jenkins.io/doc/book/managing/plugins)。
#2. 定义 Pipeline
声明式和脚本式 Pipeline 都是来描述软件交付流水线的各个部分的 DSL。脚本式 Pipeline 是用 [Groovy 语法](http://groovy-lang.org/semantics.html) 的有限形式编写的。

Groovy 语法的相关组件将在本文档中根据需要引入，因此如果你对 Groovy 有所理解会很有帮助，但 Pipeline 并不需要这些知识。

Pipeline 可以通过以下方式创建：

- 通过 Blue Ocean - 在 Blue Ocean 中设置完成 Pipeline 后，可以使用 Blue Ocean UI 来编写 Pipeline 的 Jenkinsfile 并提交到源代码管理中。
- 通过 classic UI - 可以通过 classic UI 直接输入基本的 Pipeline 到 Jenkins 中。
- 在 SCM 中 - 可以手工编写 Jenkinsfile，然后提交到项目的源代码管理仓库。

用上面几种方法定义 Pipeline 的语法是相同的。虽然 Jenkins 支持直接将 Pipeline 输入到 classic UI 中，通但常认为最佳做法是在 Jenkinsfile 中定义 Pipeline，然后 Jenkins 将直接从源代码管理中加载。
##2.1 通过 Blue Ocean
如果你初次接触 Jenkins Pipeline，Blue Ocean UI 可以帮助你 [设置 Pipeline 项目](https://jenkins.io/doc/book/blueocean/creating-pipelines)，并通过图形化 Pipeline 编辑器自动为你创建和编写 Pipeline（例如 Jenkinsfile）。

作为在 Blue Ocean 中设置 Pipeline 项目的一部分，Jenkins 为项目的源代码管理库配置了一个安全且经过适当验证的连接。因此，通过 Blue Ocean 的 Pipeline 编辑器对 Jenkinsfile 所做的任何更改都会自动保存并提交到源代码管理。

关于 Blue Ocean 的更多资料参考 [Blue Ocean 章节](https://jenkins.io/doc/book/blueocean)。
##2.2 通过 classic UI
通过 classic UI 创建的 Jenkinsfile 会直接被 Jenkins 保存（保存在 Jenkins 的根目录）。

要通过 Jenkins classic UI 创建一个基本的 Pipeline：
###2.2.1 如果需要，确保已经登录 Jenkins
###2.2.2 在 Jenkins 首页（例如 classic UI 的 Dashboard），点击左上角的 New Item
![classic-ui-left-column](http://img.blog.csdn.net/20180303105550455?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
###2.2.3 在 Enter an item name 字段，指定 Pipeline 项目名
注意：Jenkins 使用 item 的名字来创建磁盘上的目录。建议名字中不要使用空格，以避免脚本中无法处理路径的 BUG。
###2.2.4 点击 Pipeline，然后点击页面底部的 OK 打开 Pipeline 的配置页面（其 General 选项卡被选中） 
![new-item-creation](http://img.blog.csdn.net/20180303105625630?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
###2.2.5 点击页面顶部的 Pipeline 选项卡，下拉到 Pipeline 部分
注意：相反，如果要在源代码控制中定义 Jenkinsfile，请按照下面的在 SCM 中的说明进行操作。
###2.2.6 在 Pipeline 部分，确保 Definition 字段字段指示 Pipeline script 选项
###2.2.7 在 Script 文本框中输入你的 Pipeline 代码
例如，复制以下声明式 Pipeline 代码（在Jenkinsfile（...）标题下）或其脚本版本等效的代码，并将其粘贴到 Script 文本框中。（下面的示例在本过程的其余部分中使用）
```
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any // agent 指示 Jenkins 为整个 Pipeline 分配执行程序（在 Jenkins 环境中的任何可用代理/节点上）和工作空间
    stages {
        stage('Stage 1') {
            steps {
                echo 'Hello world!'  // echo 在控制台输出中写入简单的字符串
            }
        }
    }
}
```
对应脚本式 Pipeline 代码如下：
```
Jenkinsfile (Scripted Pipeline)
node {  // node 和上面的 agent 相同
    stage('Stage 1') {
        echo 'Hello World'   // echo 在控制台输出中写入简单的字符串
    }
}
```
![example-pipeline-in-classic-ui](http://img.blog.csdn.net/20180303105746341?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
注意：还可以通过 Script 文本框右上角的 **try sample Pipeline** 选项从固定的脚本式 Pipeline 示例中进行选择。注意，此字段中没有可用的固定声明式 Pipeline 示例。
###2.2.8 点击 Save 来打开 Pipeline 的 project/item 页面
###2.2.9 在这个页面中点击左边的 Build Now 来运行 Pipeline
![classic-ui-left-column-on-item](http://img.blog.csdn.net/20180303105900416?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
###2.2.10 在左边的 Build History 中点击 #1 来获取这个特定的 Pipeline 运行详情
###2.2.11 点击 Console Output 查看 Pipeline 运行中的完整输出。下面的输出表明运行成功：
![hello-world-console-output](http://img.blog.csdn.net/20180303105931543?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
注意：

- 也可以通过单击内部编号（例如＃1）左侧的彩色地球仪 globe 直接从 Dashboard 访问控制台输出。
- 通过 classic UI 定义 Pipeline 可方便测试代码片段，或者用于处理简单 Pipeline 或不需要源代码从存储库检出/克隆的 Pipeline。如上所述，不同于通过 Blue Ocean 或源代码控制定义的 Jenkinsfiles，Pipeline 项目中的输入 Script 文本框的 Jenkinsfiles 由 Jenkins 本身存储在 Jenkins 根目录中。因此，为了更好地灵活控制 Pipeline，特别是通过源代码管理的项目可能会增加复杂性，**建议使用 Blue Ocean 或源代码控制来定义 Jenkins 文件**。
##2.3 使用 SCM
通过 classic UI 的 Pipeline 配置页面上的的 Script 文本框难以编写和维护复杂的 Pipeline。

Pipeline 的 Jenkinsfile 文件可以用文本编辑器或 IDE 编写，然后提交到源代码控制系统（可以和 Jenkins 将要构建的应用代码放在一起）。Jenkins 会在 Pipeline 项目的构建过程中从源代码控制系统获取 Jenkinsfile，然后处理并执行这个 Pipeline。

要将你的 Pipeline 项目配置为使用源代码控制系统中：

1. 参考上述通过 classic UI 定义 Pipeline 的过程，直到步骤 5（访问 Pipeline 配置页面上的 Pipeline 部分）。
2. 在 Definition 字段中，将 Pipeline script 改为 SCM option。
3. 在 SCM 字段中，选择包含 Jenkinsfile 文件的仓库的源代码控制系统的类型。
4. 完成针对你的仓库的源代码控制系统类型对应的字段。
知识点：如果你不理解该选哪个，点击 ？ 图标获取更多信息。
5. 在 Script Path 字段中，指定 Jenkinsfile 文件路径（包括文件名）。默认使用仓库根目录中的名为 Jenkinsfile 的文件。

当你更新指定的仓库时，只要 Pipeline 配置了 SCM 轮询触发器（polling trigger），就会触发新的构建。

>由于 Pipeline 代码（特别是脚本式 Pipeline）是用类似 Groovy 的语法编写的，如果你的 IDE 没有正确地语法突出显示 Jenkins 文件，请尝试在 Jenkins 文件的顶部插入 `#!/usr/bin/env groovy` 可能会纠正这个问题。
#3. 内建文档
Pipeline 带有内置的文档功能，可以更轻松地创建各种复杂性的流水线。该内置文档是基于 Jenkins 实例中安装的插件自动生成和更新的。

如果你的 Jenkins 实例运行在本机的 8080 端口，则内建文档可以通过 `localhost:8080/pipeline-syntax/` 全局访问。同样的文档也会链接到每个配置过的 Pipeline 项目的侧边栏中的 **Pipeline Syntax**。
![classic-ui-left-column-on-item](http://img.blog.csdn.net/20180303110108422?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
##3.1 片段生成器
内置的“Snippet Generator”工具有助于为每个单独的 step 创建一些代码，发现插件提供的新的 step 或者针对特定 step 尝试不同参数。

片段生成器动态将可用的 step 列表填充到 Jenkins 实例。可用的 step 数量取决于安装的插件，这些插件明确公开了在 Pipeline 中使用的步骤。

要通过 Snippet Generator 创建一个 step 片段：

1. 在一个配置过的 Pipeline 中导航到 **Pipeline Syntax** 链接，或访问 `localhost:8080/pipeline-syntax`
2. 在 **Sample Step** 下拉菜单中选择需要的 step
3. 使用 **Sample Step** 下拉菜单下方的动态填充区域来配置选中的 step
4. 点击 **Generate Pipeline Script** 来创建可以复制粘贴到 Pipeline 中的的 Pipeline 片段

![snippet-generator](http://img.blog.csdn.net/20180303110136205?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
要获取关于选中的 step 的额外信息或文档，点击帮助图表（上图的红色箭头表示）。
##3.2 全局变量引用
除了仅表示 step 的片段生成器外，Pipeline 还提供内置的“全局变量引用”。就像片段生成器一样，它也是由插件动态填充的。然而，与片段生成器不同，全局变量引用仅包含由 Pipeline 提供的变量的或可用于 Pipelines 的插件的文档。（或者：全局变量引用仅包含由 Pipeline 或用于 Pipeline 的插件提供的变量的文档。原文：the Global Variable Reference only contains documentation for variables provided by Pipeline or plugins, which are available for Pipelines.）

Pipeline 中默认提供的变量是：

- **env**：Environment 环境变量。可以在脚本式 Pipeline 中访问，例如：`env PATH` 或 `env BUILD_ID`。请参阅 [内置的全局变量引用](http://localhost:8080/pipeline-syntax/globals#env)，以获取 Pipeline 中可用的完整和最新的环境变量列表。
- **params**：把为 Pipeline 定义的所有参数暴露为只读的 [Map](http://groovy-lang.org/syntax.html#_maps)，例如：`params.MY_PARAM_NAME`。
- **currentBuild**：可以用来获取关于当前执行的 Pipeline 的信息，包括 `currentBuild.result`，`currentBuild.displayName` 等。请参阅内置的全局变量引用，以获取 currentBuild 的所有可用属性。
#4. 延伸阅读
本节仅仅介绍 Jenkins Pipeline 可以完成的工作，但应该提供了足够的基础来开始体验  Jenkins。

在下一节，Jenkinsfile 中，将讨论更多 Pipeline step 以及用于实施成功的真实世界 Jenkins Pipeline 的模式。
##附加资源
- [Pipeline Steps Reference](https://jenkins.io/doc/pipeline/steps)，包括关于 Jenkins Update Center 分发的插件的所有步骤。
- [Pipeline Examples](https://jenkins.io/doc/pipeline/examples)，一个社区策划的可复制 Pipeline 示例集合。