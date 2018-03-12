[原文地址](https://jenkins.io/doc/book/installing/)

本教程用于在一台独立的或本地的机器上安装 Jenkins。

Jenkins 通常作为一个独立的应用程序在其自己的进程中运行，内置 [Java servlet](https://stackoverflow.com/questions/7213541/what-is-java-servlet) 容器/应用程序服务器（[Jetty](http://www.eclipse.org/jetty/)）。

Jenkins 也可以在其他的 Java servlet 容器中运行，例如 [Apache Tomcat](http://tomcat.apache.org/) 或 [GlassFish](https://javaee.github.io/glassfish/)。然而，本教程并不会说明如何在这些容器中安装。

注意：虽然本教程讲的是如何本地安装 Jenkins，这部分内容也可以用于设置生产环境中的 Jenkins。
#1. 先决条件
最低硬件配置：

- 256 MB 内存
- 1 GB 剩余磁盘空间，如果将 Jenkins 当做 Docker 容器运行，建议最少 10 GB

为小团队推荐的硬件配置：

- 1 GB+ 内存
- 50 GB+ 剩余磁盘空间

软件要求：

- Java 8 - Java Runtime Environment (JRE) 或 a Java Development Kit (JDK) 都可以
注意：如果将 Jenkins 当做 Docker 容器运行，则不需要满足这个要求。
#2. 安装平台
##2.1 Docker
Docker 是用于在称为容器的隔离环境中运行应用程序的平台。像 Jenkins 这样的应用程序可以被当做只读的镜像下载，镜像在 Docker 中运行为容器。Docker 容器实际上就是镜像的运行中的实例。从这个角度讲，镜像可以持久保存，而容器则是临时的。更多资料参考 [Docker 文档入门教程的第一部分：方向和设置](http://blog.csdn.net/kikajack/article/details/79350391)，官方原版 [在此](https://docs.docker.com/get-started/)。

Docker 的基础平台和容器设计意味着可以在任何支持运行 Docker 的操作系统（macOS、Linux 和 Windows）或云服务（AWS 和 Azure）上运行单个 Docker 镜像（运行任何应用程序，如 Jenkins）。
###2.1.1 安装 Docker
要在你的操作系统上安装 Docker，访问 [Docker store](https://store.docker.com/search?type=edition&offering=community) 网页并点击适合你的操作系统或云服务的 Docker Community Edition 按钮。安装 Docker 官网提示进行安装。

Jenkins 也可以在 Docker Enterprise Edition 企业版上运行。

如果你在基于 Linux 的操作系统上安装 Docker，确保将 Docker 配置为可以被非 root 用户管理。更多资料参考 [Docker’s Post-installation steps for Linux](https://docs.docker.com/engine/installation/linux/linux-postinstall/) 页面，这个页面也描述了如何将 Docker 配置为开机启动。
###2.1.2 在 Docker 中下载并运行 Jenkins
有好几个可用的 Jenkins 镜像。

建议使用 [jenkinsci/blueocean 镜像](https://hub.docker.com/r/jenkinsci/blueocean/)（在 Docker Hub 仓库）。这个镜像包含 Jenkins 的适用于生产环境的 [长期支持版本](https://jenkins.io/download)，并集成了 Blue Ocean 插件和特性。这意味着不需要再单独安装 Blue Ocean 了。

每次 Blue Ocean 发布新版本时，jenkinsci/blueocean 镜像都会发布。可用在 [tags](https://hub.docker.com/r/jenkinsci/blueocean/tags/) 页面看到之前发布的镜像列表。

当然也有其他的可用的 Jenkins 镜像（在 Docker Hub 的 [jenkins/jenkins](https://hub.docker.com/r/jenkins/jenkins/) 中）。然而，那些不包含 Blue Ocean，需要通过 Jenkins 的 `[Manage Jenkins](https://jenkins.io/doc/book/managing)` > `[Manage Plugins](https://jenkins.io/doc/book/managing/plugins)`  页面安装。更多资料参考 [Getting started with Blue Ocean](https://jenkins.io/doc/book/blueocean/getting-started)。
####2.1.2.1 在 macOS 和 Linux 上
#####1. 打开终端窗口
#####2. 下载 [jenkinsci/blueocean 镜像](https://hub.docker.com/r/jenkinsci/blueocean/)并用 [`docker run`](https://docs.docker.com/engine/reference/commandline/run/)命令在 Docker 中运行为容器：
```
docker run \
  -u root \
  --rm \  
  -d \ 
  -p 8080:8080 \ 
  -p 50000:50000 \ 
  -v jenkins-data:/var/jenkins_home \ 
  -v /var/run/docker.sock:/var/run/docker.sock \ 
  jenkinsci/blueocean 
```
参数：

- `--rm`：可选。在 Docker 容器（jenkinsci/blueocean 的实例）关闭时自动删除。
- `-d`：可选，`detached` 后台模式。后台运行 jenkinsci/blueocean 容器并输出容器的 ID。如果不指定这个选项，Docker 会把日志输出到终端窗口。
- `-p 8080:8080`：`publishes` 发布端口。将发布的 jenkinsci/blueocean 容器的 8080 端口映射到宿主机的 8080 端口。第一个数字代表宿主机的端口，最后一个数字代表容器的端口。因此如果你指定 `-p 49000:8080`，表示可以通过访问本地主机的 49000 端口来访问 Jenkins。
- `-p 50000:50000`：可选。将发布的 jenkinsci/blueocean 容器的 50000 端口映射到宿主机的 50000 端口。只有在其他主机设置了一个或多个基于 JNLP 的 Jenkins 代理程序，而这些代理程序又与 `jenkinsci/blueocean` 容器（作为主 Jenkins 服务器“Jenkins master”）进行交互时才需要这个配置。默认情况下，基于 JNLP 的 Jenkins 代理通过 TCP 端口 50000 与 Jenkins master 进行通信。可以参考 [Configure Global Security](https://jenkins.io/doc/book/managing/security/) 页面更改 Jenkins master 上的端口号。如果要更改 Jenkins 主机的 JNLP 代理的 TCP 端口值（假设是 51000），那就需要重新运行Jenkins（通过 `docker run ...` 命令）并指定此“publishes”选项 `-p 52000：51000`，其中最后一个值与 Jenkins master 上的这个更改值相匹配，第一个值是 Jenkins master 的宿主机上的端口号，基于 JNLP 的 Jenkins 代理通过它（52000）与 Jenkins master 进行通信。
- `-v jenkins-data:/var/jenkins_home`：可选，但是建议使用。将容器的目录 `/var/jenkins_home` 映射到 [Docker volume](https://docs.docker.com/engine/admin/volumes/volumes/) 卷并命名为 `jenkins-data`。如果这个卷不存在，那么这个 `docker run` 命令会自动创建卷。如果你希望每次重启 Jenkins 时能持久化 Jenkins 的状态，则必须使用这个参数。如果没有指定，则每次 Jenkins 重启时都会初始化为新的实例。
注意：`jenkins-data` 卷可以通过 [`docker volume create`](https://docs.docker.com/engine/reference/commandline/volume_create/) 命令独立创建：
`docker volume create jenkins-data`
除了将容器的目录 `/var/jenkins_home` 映射到 Docker 卷外，也可以映射到宿主机的本地文件系统，例如，通过 `-v $HOME/jenkins:/var/jenkins_home` 可以将容器的目录 `/var/jenkins_home` 映射到宿主机 $HOME 目录下的 jenkins 子目录中，通常是 `/Users/<your-username>/jenkins` 或 `/home/<your-username>/jenkins`。
- `-v /var/run/docker.sock:/var/run/docker.sock`：可选。`/var/run/docker.sock` 表示 Docker 守护进程监听的 Unix 套接字。这个映射使得 jenkinsci/blueocean 容器可以和 Docker 守护进程通信，如果 jenkinsci/blueocean 容器需要实例化其他 Docker 容器时这种通信是必须的。如果使用 docker 参数（例如 `agent { docker { ... } }`）运行语法中包含 [代理](https://jenkins.io/doc/book/pipeline/syntax/#agent) 部分的声明式管道，则此选项是必需的。更多资料参考 [Pipeline Syntax](https://jenkins.io/doc/book/pipeline/syntax/) 页面。
- `jenkinsci/blueocean`：jenkinsci/blueocean 镜像本身。如果运行 `docker run` 命令时还没有下载这个镜像，docker 会自动下载。此外，如果在你上次运行这个命令之后镜像有更新的话，运行这个命令时会自动下载新发布的镜像。
注意：Docker 镜像也可以通过 [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) 命令单独下载：`docker pull jenkinsci/blueocean`。

注意：如果复制并粘贴以上命令片段不起作用，请尝试复制并粘贴这个无注释的版本：
```
docker run \
  -u root \
  --rm \
  -d \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkinsci/blueocean
```
#####3. 进行 [安装后的设置](https://jenkins.io/doc/book/installing/#setup-wizard)
####2.1.2.2 在 Windows 上
#####1. 打开命令行窗口
#####2. 下载 [jenkinsci/blueocean 镜像](https://hub.docker.com/r/jenkinsci/blueocean/)并用 [`docker run`](https://docs.docker.com/engine/reference/commandline/run/)命令在 Docker 中运行为容器：
```
docker run ^
  -u root ^
  --rm ^
  -d ^
  -p 8080:8080 ^
  -p 50000:50000 ^
  -v jenkins-data:/var/jenkins_home ^
  -v /var/run/docker.sock:/var/run/docker.sock ^
  jenkinsci/blueocean
```
每个选项的解释参考上一部分。
#####3. 进行 [安装后的设置](https://jenkins.io/doc/book/installing/#setup-wizard)
###2.1.3 访问 Jenkins/Blue Ocean 的容器
如果你有 Docker 经验并且想通过终端或命令行窗口使用 [`docker exec`](https://docs.docker.com/engine/reference/commandline/exec/) 命令访问 Jenkins/Blue 容器，可以在 [`docker run`](https://docs.docker.com/engine/reference/commandline/run/) 命令中用 `--name jenkins-blueocean` 之类的选项将容器命名为  jenkins-blueocean。

这意味着可以通过下面的命令访问这个容器：
```
docker exec -it jenkins-blueocean bash
```
###2.1.4 通过 Docker logs 访问 Jenkins console 日志
有时候你会需要访问 Jenkins console 日志，例如 [Post-installation setup wizard](https://jenkins.io/doc/book/installing/#setup-wizard) 中的 [Unlocking Jenkins](https://jenkins.io/doc/book/installing/#unlocking-jenkins)。

如果执行上面的 `docker run ...`  命令时没有通过 `-d` 选项指定为后台模式，那么 Jenkins console 日志可以在运行 Docker 命令的终端轻松获取到。

否则，可以通过 [docker logs](https://docs.docker.com/engine/reference/commandline/logs/) 访问 jenkinsci/blueocean 容器中的 Jenkins console 日志：
```
docker logs <docker-container-name>
```
其中 `<docker-container-name>` 可以通过 [docker ps]() 命令获取。如果在执行上面的 `docker run ...`  命令时指定了 `--name jenkins-blueocean` 选项，可以这样使用 `docker logs` 命令：
```
docker logs jenkins-blueocean
```
###2.1.5 访问 Jenkins 根目录
有时候你会需要访问 Jenkins 根目录，例如检查 `workspace` 子目录中的 Jenkins 构建详情。

如果将 Jenkins 根目录（`/var/jenkins_home`）映射到机器的本地文件系统（通过上面的 `docker run ...` 命令），则可以直接通过本地机器的终端或命令行窗口访问这个目录的内容。

其他情况下，如果在 `docker run ...` 命令中指定了 `-v jenkins-data:/var/jenkins_home`，可以通过 [docker exec](https://docs.docker.com/engine/reference/commandline/exec/) 命令在 `jenkinsci/blueocean` 容器的终端或命令行窗口访问 Jenkins 根目录的内容：
```
docker exec -it <docker-container-name> bash
```
跟上面一部分的例子一样，`<docker-container-name>` 可以通过 [docker ps](https://docs.docker.com/engine/reference/commandline/ps/) 命令得到。如果在 `docker run ...` 命令中使用了 `--name jenkins-blueocean` 选项（参考 [Accessing the Jenkins/Blue Ocean Docker container](https://jenkins.io/doc/book/installing/#accessing-the-jenkins-blue-ocean-docker-container)），则可以直接使用 `docker exec` 命令：
```
docker exec -it jenkins-blueocean bash
```
##2.2 WAR 文件
Jenkins 的 Web application ARchive (WAR) 版本的文件可以安装在任何支持 Java 的操作系统平台上。

####下载并运行 Jenkins 的 WAR 版本的文件：

- 下载 [最新稳定版本的 Jenkins WAR 文件](http://mirrors.jenkins.io/war-stable/latest/jenkins.war) 到合适的目录中。
- 在下载目录中打开终端或命令行窗口。
- 运行命令 `java -jar jenkins.war`。
- 通过浏览器访问 `http://localhost:8080`，等到出现 **Unlock Jenkins** 页面。
- 下面的步骤参考最后一部分 - 安装后的设置。

####注意：

- Unlike downloading and running Jenkins with Blue Ocean in Docker (above), this process does not automatically install the Blue Ocean features, which would need to installed separately via the Manage Jenkins > Manage Plugins page in Jenkins. Read more about the specifics for installing Blue Ocean on the Getting started with Blue Ocean page.

- You can change the port by specifying the --httpPort option when you run the java -jar jenkins.war command. For example, to make Jenkins accessible through port 9090, then run Jenkins using the command:
java -jar jenkins.war --httpPort=9090
##2.3 macOS
需要使用安装包：

- [下载最新安装包](http://mirrors.jenkins.io/osx/latest)
- 打开安装包并按照说明操作

也可以通过 `brew` 安装 Jenkins：

- 安装最新版本：
```
brew install jenkins
```
- 安装长期支持版本：
```
brew install jenkins-lts
```
##2.4 Linux
###Debian/Ubuntu
在 Ubuntu 等基于 Debian 的 发行版中，可以通过 apt 安装 Jenkins。

最近的版本在 [apt 仓库](https://pkg.jenkins.io/debian/) 中可用。较旧但稳定的 LTS 版本位于 [这个 apt 仓库](https://pkg.jenkins.io/debian-stable/) 中。
```
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
```
这个软件包的安装将会：

- 设置 Jenkins 在开机后启动为守护进程。详情参考 /etc/init.d/jenkins。
- 创建 jenkins 用户来运行这个服务。
- 将控制台日志直接输出到文件 `/var/log/jenkins/jenkins.log`。如果你正在解决 Jenkins 的问题，请检查此文件。
- 将启动的配置参数填充道 /etc/default/jenkins 文件中，例如 `JENKINS_HOME`。
- 设置 Jenkins 监听 8080 端口。通过浏览器访问这个端口来启动配置。

>如果你的 /etc/init.d/jenkins 文件启动 Jenkins 失败，编辑 /etc/default/jenkins 将 `----HTTP_PORT=8080----` 这一行替换为 `----HTTP_PORT=8081----`，当然你也可以使用 8081 之外的其他端口。
##2.5 Windows
使用安装程序来安装：

- [下载最新安装包](http://mirrors.jenkins.io/windows/latest)
- 运行这个安装包并按照指示操作
##2.6 其他操作系统
请参考原文
#3. 安装后的设置
通过上面的步骤下载、安装并运行 Jenkins 后，需要进行安装后的设置。

这个设置向导会引导你完成几个快速“one-off”步骤来解锁 Jenkins，通过使用插件进行自定义，并创建第一个可以访问 Jenkins 的管理员用户。
##3.1 解锁 Jenkins
在第一次访问新创建的 Jenkins 实例时，系统会要求你通过自动创建的密码来解锁它。

###3.1.1 通过浏览器访问 http://localhost:8080 （或其他你在安装时指定的端口）并等待  Unlock Jenkins 页面出现
![setup-jenkins-01-unlock-jenkins-page](http://img.blog.csdn.net/20180302092846147?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
###3.1.2 从 Jenkins 控制台的日志输出中，复制自动生成的字母数字密码（在两组星号之间）
![setup-jenkins-02-copying-initial-admin-password](http://img.blog.csdn.net/20180302092922886?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
###3.1.3 在 Unlock Jenkins 页面，复制密码到 `Administrator password` 字段中，点击 Continue
注意：

- 如果是在 Docker 的后台模式（detached mode）中运行 Jenkins，可以通过 Docker logs 访问 Jenkins 的控制台日志。
- Jenkins 的控制台日志指明这个密码还可以从哪里获取。在你访问新安装的 Jenkins 主界面之前，必须在设置向导中填入这个密码。如果你跳过安装向导中后续的用户创建步骤，则这个密码同时也是默认管理员账号（用户名是 admin）的密码。
##3.2 通过插件自定义 Jenkins
解锁 Jenkins 之后，会出现自定义 Jenkins 页面。作为初始设置的一部分，可以在这里安装任意数量的有用的插件。

点击下面两个选项中的一个：

- **Install suggested plugins** - 安装推荐的一组插件，可以满足大多数使用场景。
- **Select plugins to install** - 选择要安装的插件集。第一次访问插件选择页面时，会默认选中推荐的插件。

>如果不确定需要哪个插件，选择 Install suggested plugins 即可。可以在后面通过 [Manage Jenkins](https://jenkins.io/doc/book/managing) > [Manage Plugins](https://jenkins.io/doc/book/managing/plugins/) 页面来安装或删除额外的 Jenkins 插件。

设置向导显示 Jenkins 的配置过程和正在安装的所选 Jenkins 插件集。这个过程可能需要几分钟的时间。
##3.3 创建第一个管理员用户
最后，通过插件自定义 Jenkins 完成后，Jenkins 会要求你创建第一个管理员用户。
###3.3.1 填写信息
当 **Create First Admin User** 页面出现时，在每个字段中填写管理员用户的信息并点击 **Save** 结束。
###3.3.2 使用 Jenkins
当 **Jenkins is ready** 页面出现时，点击 **Start** 开始使用 Jenkins。
注意：

- 这个页面可能表明 Jenkins 已经差不多准备好了！相反，如果不是这样，请单击 **Restart**。
- 如果这个页面在一分钟后没有自动刷新，用浏览器手动刷新。
###3.3.3 登录
如果需要，使用刚刚创建的用户的凭据登录到 Jenkins，然后就可以使用 Jenkins！

>从此刻起，只有在提供了有效的用户名和密码凭据后才可以访问到 Jenkins 的界面。