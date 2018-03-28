[原文地址](https://docs.docker.com/compose/install/)

Compose 可以安装在 macOS、Windows 或 64 位的 Linux 上。
# 1. 先决条件
Docker Compose 依赖 Docker Engine 才能正常工作，因此请确保已根据你的设置安装了本地或远程 Docker Engine。

- 在 Docker for Mac 和 Windows 等桌面系统上，Docker Compose 作为桌面安装的一部分。
- 在 Linux 系统上，首先按照 Get Docker 页面上的说明为操作系统安装 [Docker](https://docs.docker.com/install/#server)，然后再返回此处安装 Compose 的说明。
- 要以非 root 用户身份运行 Compose，请参阅[以非 root 用户身份管理 Docker ](https://docs.docker.com/install/linux/linux-postinstall/)。
# 2. 安装 Compose
这里只翻译 Linux 上的安装，其他系统（macOS、Windows）及其他安装方式（`pip` Python 包管理器）或将 Compose 安装为容器，请参考原文。

### 在 Linux 上安装 Compose
可以使用 curl 从 [GitHub 上的 Compose 仓库](https://github.com/docker/compose/releases) 下载二进制文件。
#### 1. 下载最新版的 Docker Compose：
```
sudo curl -L https://github.com/docker/compose/releases/download/1.19.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```
>注意：把下载命令中的 Compose 替换为最新版本
>上述命令只是示例，版本需要替换为最新的。可以登录 [GitHub 上的 Compose 仓库](https://github.com/docker/compose/releases) 查看。
#### 2. 为二进制文件添加可执行权限：
```
sudo chmod +x /usr/local/bin/docker-compose
```
#### 3. 可选，为 `bash` 或 `zsh` 安装 [命令自动补全](http://111.230.25.113:4000/compose/completion/)

#### 4. 测试
```
$ docker-compose --version
docker-compose version 1.19.0, build 1719ceb
```
# 3. 主分支构建
如果有兴趣尝试预发布版本，可以从 https://dl.bintray.com/docker-compose/master/ 下载二进制文件。预发布版本允许你在发布之前尝试新功能，但可能不太稳定。
# 4. 升级
如果你要从 Compose 1.2 或更低版本升级，请在升级 Compose 后删除或迁移现有容器。这是因为从版本 1.3 开始，Compose 使用 Docker labels 标签来跟踪容器，因此需要重新创建容器以添加标签。

如果 Compose 检测到创建的容器没有标签时会拒绝运行，这样你就不会得到两套。如果你想继续使用现有的容器（例如，它们具有要保留的数据卷），则可以使用 Compose 1.5.x 通过以下命令来迁移它们：
```
docker-compose migrate-to-labels
```
另外，如果你不需要保留，可以删除它们。Compose 会创建新的。
```
docker container rm -f -v myapp_web_1 myapp_db_1 ...
```
# 5. 卸载
要卸载通过 curl 安装的 Docker Compose：
```
sudo rm /usr/local/bin/docker-compose
```
要卸载通过 pip 安装的 Docker Compose：
```
pip uninstall docker-compose
```
>发生“Permission denied”错误了吗？
>如果使用上述任一方法导致“Permission denied”错误，则可能没有适当的权限来移除 docker-compose。要强制删除，请将 sudo 添加到上述任一命令中并再次运行。
