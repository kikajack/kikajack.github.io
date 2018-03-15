[原文地址](https://docs.docker.com/config/containers/logging/)

`docker logs` 命令显示运行中的容器记录的日志信息。`docker service logs` 命令显示服务中的所有容器记录的日志信息。日志信息和格式几乎完全取决于容器的终端命令。

默认情况下，`docker logs` 和 `docker service logs` 命令会显示类似于终端中交互式运行命令的输出。UNIX 和 Linux 命令在运行时通常会打开三个 I/O 流，分别称为 `STDIN`，`STDOUT` 和 `STDERR`。`STDIN` 是命令的输入流，可能包括来自键盘的输入或来自另一个命令的输入。`STDOUT` 通常是命令的正常输出，而 `STDERR` 通常用于输出错误消息。默认情况下，`docker logs`  显示命令的 `STDOUT` 和 `STDERR`。要阅读有关 I/O 和 Linux 的更多信息，请参阅 [I/O 重定向的Linux文档项目文章](http://www.tldp.org/LDP/abs/html/io-redirection.html)。

某些情况下，需要采取以下步骤才能使 `docker logs` 显示有用信息：

- 如果你使用将日志发送到文件、外部主机、数据库或另外一个后端日志的日志驱动程序，`docker logs` 不会显示有用信息。
- 如果你的镜像运行的是 web 服务器或数据库等非交互式进程，这个应用程序可能会将输出发送到日志文件而不是 `STDOUT` 和 `STDERR`。

在第一种情况下，你的日志会通过其他方式处理，你可以选择不使用 `docker logs`。第二种情况下，官方的 `nginx` 镜像显示了一种解决方法，官方的 Apache `httpd` 镜像显示了另一种解决方法。

官方的 `nginx` 镜像创建了一个从 `/dev/stdout` 到 `/var/log/nginx/access.log` 的符号链接，和一个从 `/dev/stderr` 到 `/var/log/nginx/error.log` 的符号链接，覆盖了日志文件并使所有日志发送到指定的相关设备。参考 [Dockerfile](https://github.com/nginxinc/docker-nginx/blob/8921999083def7ba43a06fabd5f80e4406651353/mainline/jessie/Dockerfile#L21-L23)。

官方的 Apache `httpd` 镜像改变了 `httpd` 应用程序的配置文件，将其正常输出改为 `/proc/self/fd/1`（也就是 `STDOUT`），错误输出改为 `/proc/self/fd/2`（也就是 `STDID`）。参考 [Dockerfile](https://github.com/docker-library/httpd/blob/b13054c7de5c74bbaa6d595dbe38969e6d4f860c/2.2/Dockerfile#L72-L75)。
