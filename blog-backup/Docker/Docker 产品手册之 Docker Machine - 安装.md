[原文地址](https://docs.docker.com/machine/install-machine/)

在 macOS 和 Windows 中，Machine 会在安装 [Docker for Mac](https://docs.docker.com/docker-for-mac/)、[Docker for Windows](https://docs.docker.com/docker-for-windows/) 或 [Docker Toolbox](https://docs.docker.com/toolbox/overview/) 时和 Docker 产品一同安装。

如果你只需要安装 Docker Machine，可以通过下面的步骤直接编译安装 Machine 二进制代码。可以在 GitHub 的 [docker/machine release page](https://github.com/docker/machine/releases/) 中使用最新版本。
#1. 直接安装 Machine
##1.1 [安装 Docker](https://docs.docker.com/engine/installation/)
##1.2 下载 Docker Machine 源码并解压
对于 macOS 系统：
```
$ curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname -s`-`uname -m` >/usr/local/bin/docker-machine && \
  chmod +x /usr/local/bin/docker-machine
```
对于 Linux 系统：
```
$ curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine && \
sudo install /tmp/docker-machine /usr/local/bin/docker-machine
```
对于 Windows 上运行的 [Git BASH](https://git-for-windows.github.io/)：
```
$ if [[ ! -d "$HOME/bin" ]]; then mkdir -p "$HOME/bin"; fi && \
curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-Windows-x86_64.exe > "$HOME/bin/docker-machine.exe" && \
chmod +x "$HOME/bin/docker-machine.exe"
```
>上面的命令必须在 Windows 中使用类似 `Git BASH` 等支持 Linux 命令的终端模拟器来执行。

当然也可以从 [docker/machine release page](https://github.com/docker/machine/releases/) 下载一个发行版来安装。
##1.3 查看 Machine version
```
$ docker-machine version
docker-machine version 0.13.0, build 9371605
```
#2. 安装命令补齐等脚本
Machine 仓库支持几个添加特性的 `bash` 脚本，例如：

- 命令补齐
- 在 shell 提示符下显示活动机器的功能
- 添加了一个可以切换活动机器的 `docker-machine use` 子命令的功能封装

确认版本并将脚本保存到 `/etc/bash_completion.d` 或 `/usr/local/etc/bash_completion.d`:
```
scripts=( docker-machine-prompt.bash docker-machine-wrapper.bash docker-machine.bash ); for i in "${scripts[@]}"; do sudo wget https://raw.githubusercontent.com/docker/machine/v0.13.0/contrib/completion/bash/${i} -P /etc/bash_completion.d; done
```
要开启 `docker-machine` shell 提示功能，在 `~/.bashrc` 中的 `PS1` 设置中添加 `$(__docker_machine_ps1)`。
```
PS1='[\u@\h \W$(__docker_machine_ps1)]\$ '
```
可以在 [每个脚本的顶部](https://github.com/docker/machine/tree/master/contrib/completion/bash) 的注释中找到额外的文档。
You can find additional documentation in the comments at the top of each script.
#3. 如何卸载 Docker Machine
要卸载 Docker Machine：

- 删除可执行文件：`rm $(which docker-machine)`

- 可选项，删除你创建的 machines。
单独删除每个 machine：`docker-machine rm <machine-name>`
删除所有 machine：`docker-machine rm -f $(docker-machine ls -q)` (Windows 系统中可能需要 `-force`)

因为有时可能需要保存并将已经存在的 Docker 迁移到 [Docker for Mac](https://docs.docker.com/docker-for-mac/) 或 [Docker for Windows](https://docs.docker.com/docker-for-windows/) 环境，所以删除 machines 是可选的。

>注意：与 docker-machine 创建的每个虚拟机相关的 config.json，证书和其他数据存储在 Mac 和 Linux 上的 `~/.docker/machine/machines/`，Windows上的 `~\.docker\machine\machines\`。建议不要直接编辑或删除这些文件，因为这只会影响 Docker CLI 的信息，而不会影响实际的虚拟机，无论是本地上的还是远程服务器上的。
#4. 下一步
- [Docker Machine 概述 - 原文](https://docs.docker.com/machine/overview/)
- [Docker Machine 概述 - 中文](http://blog.csdn.net/kikajack/article/details/79383906#t3)
- 在 [本地系统上通过 VirtualBox](https://docs.docker.com/machine/get-started/) 创建和运行 Docker 主机
- 配置 [云服务提供商](https://docs.docker.com/machine/get-started-cloud/) 的多个主机
- [理解 Machine 概念](https://docs.docker.com/machine/concepts/)
- [Docker Machine 驱动参考](https://docs.docker.com/machine/drivers/)
- [Docker Machine 子命令 参考](https://docs.docker.com/machine/reference/)