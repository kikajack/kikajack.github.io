[参考官网](https://docs.docker.com/engine/reference/builder)
[参考中文资料](http://dockone.io/article/103)
#1. Dockerfile 用途
Docker 可以通过读取 Dockerfile 配置文件自动生成镜像，也可以直接通过 docker 命令或 docker pull 命令生成镜像。

**Dockerfile解决了自动化的问题。**使用Docker build命令即可执行文件中的所有命令，减少了镜像和容器的创建过程，简化了部署。

Dockerfile 是一个文本文档，其中包含用户可以在命令行调用以组装镜像的所有命令。
使用 docker build 用户可以创建一个自动构建，用指定目录下的 Dockerfile 构建镜像。构建由 Docker 守护程序运行，而不是由 CLI 运行。构建过程所做的第一件事是将整个上下文（递归地）发送到守护进程。在大多数情况下，最好以空目录作为上下文，并将 Dockerfile 保存在该目录中。也可以使用该 -f 标志指向文件系统中任何位置的Dockerfile。
```
$ docker build . #用当前目录下的 Dockerfile 构建镜像
$ docker build -f /path/to/a/Dockerfile . # 用/path/to/a/Dockerfile 构建镜像到当前目录中
```
#2. 命令详解
Dockerfile 支持的命令如下：
`INSTRUCTION argument`

指令不区分大小写。但是，命名约定为全部大写。
###1. FROM 命令
所有 Dockerfile 都必须以 FROM 命令开始。 FROM 命令指定镜像基于哪个基础镜像创建，接下来的命令会基于这个基础镜像。FROM 命令可以多次使用，表示会创建多个镜像。
例如：`FROM ubuntu` 表示新的镜像将基于 Ubuntu 这个基础镜像来构建。
###2. MAINTAINER 命令
设置该镜像的作者。语法如下：
`MAINTAINER <author name>`
###3. RUN 命令
在shell或者exec的环境下执行的命令。RUN指令会在新创建的镜像上添加新的层面，接下来提交的结果用在Dockerfile的下一条指令中。语法如下：
`RUN command`
###4. ADD 命令
复制文件指令。它有两个参数<source>和<destination>。destination是容器内的路径。source可以是URL或者是启动配置上下文中的一个文件。语法如下：
`ADD src destination`
###5. CMD 命令
提供了容器默认的执行命令。 Dockerfile只允许使用一次CMD指令。 使用多个CMD会抵消之前所有的指令，只有最后一个指令生效。 CMD有三种形式：
```
CMD ["executable","param1","param2"]
CMD ["param1","param2"]
CMD command param1 param2
```
###6. EXPOSE 命令
指定容器在运行时监听的端口。语法如下：
`EXPOSE port;`
###7. ENTRYPOINT 命令
配置给容器一个可执行的命令，这意味着在每次使用镜像创建容器时一个特定的应用程序可以被设置为默认程序。同时也意味着该镜像每次被调用时仅能运行指定的应用。类似于CMD，Docker只允许一个ENTRYPOINT，多个ENTRYPOINT会抵消之前所有的指令，只执行最后的ENTRYPOINT指令。语法如下：
```
ENTRYPOINT ["executable", "param1","param2"]
ENTRYPOINT command param1 param2
```
###8. WORKDIR 命令
指定RUN、CMD与ENTRYPOINT命令的工作目录。语法如下：
`WORKDIR /path/to/workdir`
###9. ENV 命令
设置环境变量。它们使用键值对，增加运行程序的灵活性。语法如下：
`ENV <key> <value>`
###10. USER 命令
镜像正在运行时设置一个UID。语法如下：
`USER <uid>`
###11. VOLUME 命令
授权访问从容器内到主机上的目录。语法如下：
`VOLUME ["/data"]`
#3. .dockerignore 文件
.dockerignore 文件添加到上下文目录，可以排除文件和目录。