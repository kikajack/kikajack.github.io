[原文地址](https://docs.docker.com/config/containers/logging/plugins/)

Docker 日志插件允许你在内置的日志驱动程序之外扩展和定制 Docker 的日志功能。日志服务提供方可以实现他们自己的插件以在 Docker Hub、Docker Store 和私有 registry 上使用其服务。本主题讲述这种日志服务下的用户如何将 Docker 配置为使用插件。
# 1. 安装日志驱动程序插件
通过 `docker plugin install <org/image>` 命令安装日志驱动程序插件。需要使用插件开发者提供的信息。

通过 `docker plugin ls` 命令可以查看所有安装的插件，通过 `docker inspect` 命令可以查看指定的插件。
# 2. 将插件配置为默认的日志驱动程序
插件安装好后，可以通过设置 `daemon.json` 文件中的 `logging-driver` 字段为指定的插件名将其配置 Docker 守护进程的默认驱动。如果日志驱动程序支持额外的选项，可以将这些选项设置到这个文件的 `log-opts` 中。
# 3. 配置某一个容器使用插件作为日志驱动程序
插件安装好后，可以在 `docker run` 命令中指定 `--log-driver` 标志为一个容器指定使用某个插件作为日志驱动程序，这在上一章节讲过。如果日志驱动程序支持额外的选项，可以通过一个或多个 `--log-opts` 选项指定它们。
