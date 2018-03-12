[原文地址](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

Docker 可以通过从 Dockerfile 中读取指令来自动构建镜像，Dockerfile 是一个文本文件，其中包含了按顺序排列的构建指定镜像所需的全部命令。Dockerfiles 采用特殊格式，使用一系列特别的指令。可以在 [Dockerfile 参考页面](https://docs.docker.com/engine/reference/builder/) 学习这些基础知识。如果对于编写 Dockerfile 你还是新手，那么接着往下看吧。

本文档介绍了由 Docker 公司和 Docker 社区推荐的用于构建高效镜像的最佳实践和方法。要查看更多实践和建议，请点击 [Dockerfile for buildpack-deps](https://github.com/docker-library/buildpack-deps/blob/master/jessie/Dockerfile)。

注意：要查看 Dockerfile 命令的详情，点击 [Dockerfile 参考页面](https://docs.docker.com/engine/reference/builder/) 。
#1. 一般准则和建议
##1.1 容器应该精简 ephemeral
由 `Dockerfile` 定义的映像生成的容器应尽可能精简。意思是说，在容器被停止和销毁，并且建立和配置完成一个新的容器时，有绝对最少的设置和配置。 你可能需要查看 十二要素应用宣言 的 [Processes](https://12factor.net/processes) 部分（译文在 [这里](https://12factor.net/zh_cn/processes)），以了解以这种无状态方式运行容器的动机。

原文：
The container produced by the image your Dockerfile defines should be as ephemeral as possible. By “ephemeral,” we mean that it can be stopped and destroyed and a new one built and put in place with an absolute minimum of set-up and configuration. You may want to take a look at the Processes section of the 12 Factor app methodology to get a feel for the motivations of running containers in such a stateless fashion.
##1.2 使用 `.dockerignore` 文件
执行 `docker build` 命令时你所在的当前工作目录被称为构建上下文，Dockerfile 文件必须在这个构建上下文中。默认情况下，Dockerfile 被假设在当前目录中，但是可以通过 `-f` 标志指定一个不同位置。不管 Dockerfile 文件位于何处，当前目录中的所有文件和目录都会作为构建上下文发送到 Docker 守护进程。无意中包含了构建镜像不需要的文件会产生更大的构建上下文和更大的镜像大小。这些反过来又会增加构建时间、获取和上传镜像的时间以及容器的运行时间。要查看构建上下文有多大，在构建 Dockerfile 时查找类似下面的消息。
```
Sending build context to Docker daemon  187.8MB
```
可以使用 `.dockerignore` 文件排除与构建无关的文件而不重构源代码库。该文件支持类似 `.gitignore` 文件的排除模式。有关创建此文件的信息，参考 [这里](https://docs.docker.com/engine/reference/builder/#dockerignore-file)。
##1.3 使用多段构建
如果 Docker 版本是 17.05 或更高，那就可以使用 [多段构建](https://docs.docker.com/develop/develop-images/multistage-build/) 来大幅降低最终镜像的大小，而无需在构建期间跳过 through hoops 来减少中间层的数量或删除中间文件。

镜像仅由最终一个阶段构建，大部分时间既有利于构建缓存，又能使镜像图层最小化。（Images being built by the final stage only, you can most of the time benefit both the build cache and minimize images layers.）

你的构建阶段可能包含多个层，下面例子从最不常见的变更到最常见的变更排序：

- 安装构建应用程序所需的工具

- 安装或更新库和依赖

- 产生应用

一个 Go 应用程序的 Dockerfile 示例：
```
FROM golang:1.9.2-alpine3.6 AS build
# Install tools required to build the project
# We need to run `docker build --no-cache .` to update those dependencies
RUN apk add --no-cache git
RUN go get github.com/golang/dep/cmd/dep

# Gopkg.toml and Gopkg.lock lists project dependencies
# These layers are only re-built when Gopkg files are updated
COPY Gopkg.lock Gopkg.toml /go/src/project/
WORKDIR /go/src/project/
# Install library dependencies
RUN dep ensure -vendor-only

# Copy all project and build it
# This layer is rebuilt when ever a file has changed in the project directory
COPY . /go/src/project/
RUN go build -o /bin/project

# This results in a single layer image
FROM scratch
COPY --from=build /bin/project /bin/project
ENTRYPOINT ["/bin/project"]
CMD ["--help"]
```
##1.4 避免安装无用包
要降低复杂性、依赖、文件大小和构建时间，就要避免安装额外的或不需要的包。例如在数据库镜像中不需要文本编辑器。
##1.5 每个容器只解决一个问题
将应用程序解耦为多个容器使得横向扩展和重用容器变得更容易。例如，一个 Web 应用程序堆栈可能由三个独立的容器组成，每个容器都有其独特的镜像，以解耦的方式管理 Web 应用程序、数据库和内存中的缓存。

你可能听过这句话“每个容器一个进程”。虽然这个口头禅的意图很好，但并不一定每个容器只有一个操作系统进程。除了现在可以使用 [init 进程创建容器](https://docs.docker.com/engine/reference/run/#specifying-an-init-process) 之外，一些程序可能会自行产生其他进程。例如，Celery 可以派生多个工作进程，或者 Apache 可能会为每个请求创建一个进程。 虽然“每个容器一个进程”是一个很好的经验法则，但它并不是硬性规定。 尽你最大的努力使容器保持干净和模块化。

如果容器互相依赖，可以使用 [Docker 容器网络](https://docs.docker.com/network/) 来确保容器之间的通信。
##1.6 最小化层数
在 Docker 17.05 甚至 1.10 之前，最小化镜像的层数是很重要的。下面的改善措施缓解了这个需求：

- Docker 1.10 及更高版本中，只有 RUN、COPY 和 ADD 命令会创建层。其他命令创建临时的中间层镜像，不会在构建时增加体积。

- Docker 17.05 及更高版本，增加了分段构建功能，使得可以只复制所需的项目文件到最终的镜像中。这让你可以在中间层构建过程中添加工具和调试信息，而不会增大最终镜像的体积。
##1.7 排序多行参数
只要有可能，通过按字母数字顺序排列多行参数来简化后面的更改。这有助于避免软件包重复并使列表更容易更新。这也使得 PR 更容易阅读和审核。在反斜杠（\）之前添加空格也有帮助。

这是来自 [`buildpack-deps` 镜像](https://github.com/docker-library/buildpack-deps) 的例子：
```
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion
```
##1.8 构建缓存
在构建镜像的过程中，Docker 会按照指定的顺序执行 Dockerfile 文件中的指令。检查完所有指令后，Docker 会从缓存中寻找可用的镜像，而不是创建一个新镜像。如果不想使用缓存，可以在执行 `docker build` 命令是添加 `--no-cache=true` 选项。

然而，如果允许 Docker 使用缓存，就需要理解它何时能，何时不能，找到匹配的镜像。Docker 遵守的基本规则如下：

- 从缓存中已经存在的父镜像开始，将下一条指令与从该基本镜像派生的所有子镜像进行比较，以查看是否使用完全相同的指令构建了其中的一个子镜像。如果没有则缓存失效。

- 大多数情况下，简单的将 Dockerfile 中的指令和子镜像中的一个进行比较就足够了。然而，部分指令需要更多的检查和解释。

- 对于 `ADD` 和 `COPY` 指令，镜像中的文件内容都需要检查并为每个文件计算校验和 checksum。这些校验和中不考虑文件的最后编辑时间和最后访问时间。在缓存查找过程中，将校验和与现有镜像中的校验和进行比较。如果文件中的内容有任何更改，如内容和元数据，则缓存将失效。

- 除了 `ADD` 和 `COPY` 指令，缓存检查时不会通过检查容器中的文件来决定缓存是否匹配。例如在处理 `RUN apt-get -y update` 命令时，不会通过检查容器中更新过的文件来决定缓存是否命中。此时只会对比命令字符串是否相同来寻找匹配的缓存。

一旦关闭缓存，所有后续的 Dockerfile 命令都会生成新镜像，不使用缓存。
#2. The Dockerfile instructions
这些建议可以帮助你写出高效的、容易维护的 Dockerfile。
## FROM
FROM 指令的 [Dockerfile 参考资料](https://docs.docker.com/engine/reference/builder/#from)

只要有可能，使用官方仓库作为镜像的基础。推荐使用 [Alpine 镜像](https://hub.docker.com/_/alpine/)，因为它的控制非常严格，并且保持最小（目前低于5 MB），同时仍然是完整的发行版。
## LABEL
理解 [labels 对象](https://docs.docker.com/config/labels-custom-metadata/)

可以给镜像添加标签，来帮助项目组织镜像、记录许可信息、帮助自动化或出于其他原因。对于每个标签，添加一行以 LABEL 开头并带有一个或多个键值对的行。下面示例显示了多种支持的格式。解释性意见包含在内。

注意：如果字符串中包含空格，则必须用双引号引起来或转义这个空格。如果字符串中包含双引号，必须转义。
```
# 设置一个或多个独立的标签
LABEL com.example.version="0.0.1-beta"
LABEL vendor="ACME Incorporated"
LABEL com.example.release-date="2015-02-12"
LABEL com.example.version.is-production=""
```
镜像可以有多个标签。在 Docker 1.10 版本之前，建议将所有的标签合并到一个 LABEL 指令中，以防止创建额外的层。现在不需要这么做了，但是仍然支持合并标签。
```
# 在同一行中设置多个标签
LABEL com.example.version="0.0.1-beta" com.example.release-date="2015-02-12"
```
上面的例子也可用下面的写法：
```
# 一次设置多个标签，并使用续行字符打断很长的行
LABEL vendor=ACME\ Incorporated \
      com.example.is-beta= \
      com.example.is-production="" \
      com.example.version="0.0.1-beta" \
      com.example.release-date="2015-02-12"
```
有关可使用的标签中键和值的信息，参阅 [Understanding object labels](https://docs.docker.com/config/labels-custom-metadata/)。有关查询 querying 标签的信息，参阅 [Managing labels on objects](https://docs.docker.com/config/labels-custom-metadata/#managing-labels-on-objects) 中与过滤相关的项目。另请参阅 Dockerfile 参考中的 [LABEL](https://docs.docker.com/engine/reference/builder/#label)。
## RUN
RUN 指令的 [Dockerfile 参考资料](https://docs.docker.com/engine/reference/builder/#run)

将很长或很复杂的 RUN 语句用反斜线（\）切分为多行可以让 Dockerfile 文件易读、易理解并且易维护。
###1. APT-GET 指令
RUN 最常见的用例可能是 `apt-get` 应用程序。因为 `RUN apt-get` 命令会安装软件包，有几个需要注意的问题。

应该避免使用 `RUN apt-get upgrade` 或 `dist-upgrade`，因为许多来自父镜像的“essential”基本软件包无法在非特权容器内升级。如果父镜像中的软件包已过时，应联系其维护人员。如果你知道需要更新某个特定软件包，比如“foo”，请使用 `apt-get install -y foo` 自动更新。

在同一个 RUN 语句中一同运行 `apt-get update` 和 `apt-get install`。例如：
```
    RUN apt-get update && apt-get install -y \
        package-bar \
        package-baz \
        package-foo
```
**RUN 语句中单独使用 `apt-get update` 会导致缓存问题**，并使后面的 `apt-get install` 指令执行失败。例如，看下面的 Dockerfile：
```
    FROM ubuntu:14.04
    RUN apt-get update
    RUN apt-get install -y curl
```
上面的镜像构建完成后，所有的层都会在 Docker 缓存中。假设后面会通过添加额外的包来变更 `apt-get install` 这条指令：
```
    FROM ubuntu:14.04
    RUN apt-get update
    RUN apt-get install -y curl nginx
```
此时 Docker 会认为这个例子中的前两步和上个例子的一样，从而使用上个例子生成的缓存，导致 `apt-get update` 命令并未执行。`apt-get update` 没有运行，所以后面可能会安装的 `curl` 和 `nginx` 可能不是最新版本。

使用 `RUN apt-get update && apt-get install -y` 可以确保 Dockerfile 安装最新版本的包，无需进一步编码或手动干预。这种技术被称为“缓存破坏”（cache busting）。 也可以通过指定软件包的版本来清除缓存。这被称为版本固定（version pinning），例如：
```
    RUN apt-get update && apt-get install -y \
        package-bar \
        package-baz \
        package-foo=1.3.*
```
版本固定会强制构建时检索特定的版本，而不管缓存中的内容。该技术还可以减少由于所需软件包的意外更改而导致的故障。

下面是一个组织良好的 RUN 指令，用来演示所有的 `apt-get` 建议。
```
RUN apt-get update && apt-get install -y \
    aufs-tools \
    automake \
    build-essential \
    curl \
    dpkg-sig \
    libcap-dev \
    libsqlite3-dev \
    mercurial \
    reprepro \
    ruby1.9.1 \
    ruby1.9.1-dev \
    s3cmd=1.1.* \
 && rm -rf /var/lib/apt/lists/*
```
s3cmd 指定要安装 `1.1.*` 版本。如果镜像在之前使用的是旧的版本，指定新版本会导致 `apt-get update` 命令的缓存破坏，从而确保安装的是这个指定的新版本。每个包单独出现在一行中，可以防止出现包重复的错误。

此外，当通过删除 /var/lib/apt/lists 目录来清除 apt 的缓存时，可以减小镜像尺寸（因为 apt 缓存不会存入层）。这里的 RUN 语句用 `apt-get update` 命令开头，所以在执行 `apt-get install` 命令之前包缓存总是会得到更新。

注意：官方的 Debian 和 Ubuntu 镜像会 [自动运行](https://github.com/moby/moby/blob/03e2923e42446dbb830c654d0eec323a0b4ef02a/contrib/mkimage/debootstrap#L82-L105) `apt-get clean`，因此不需要显式调用。
###2. 使用管道
部分 RUN 命令借助管道 pipe 将一个命令的输出发送到另一个命令。下面例子演示了管道符号 `|` 的使用：
```
RUN wget -O - https://some.site | wc -l > /number
```
Docker 使用 `/bin/sh -c` 解释器执行这些命令，该解释器只评估管道中最后一个操作的退出代码以确定是否成功。在上面的示例中，只要 `wc -l` 命令执行成功，即使 `wget` 命令执行失败，此构建步骤也会成功并生成新镜像。

预先设置 `set -o pipefail &&` 命令，可以使管道中的任何一步发生错误时，都会导致命令执行失败，从而不再构建镜像。例如：
```
RUN set -o pipefail && wget -O - https://some.site | wc -l > /number
```
注意：并使是所有的 shell 都支持 `-o pipefail` 选项（比如 Debian 基础镜像中的默认 shell `dash`）。此时，可以使用 RUN 的 exec 形式来显式选择一个支持 `pipefail` 选项的 shell。例如：
```
RUN ["/bin/bash", "-c", "set -o pipefail && wget -O - https://some.site | wc -l > /number"]
```
## CMD
CMD 指令的 [Dockerfile 参考资料](https://docs.docker.com/engine/reference/builder/#cmd)

CMD 指令应该用来运行镜像中的软件，可以有任意多个参数。格式为：`CMD [“executable”, “param1”, “param2”…]`。因此，如果镜像用来运行服务，例如 Apache 和 Rails，可以通过 `CMD ["apache2","-DFOREGROUND"]` 来运行。事实上，所有的基于服务的镜像都推荐使用这种命令格式。

大多数情况下，CMD 需要交互式的 shell，例如 bash、Python 或 Perl。例如，`CMD ["perl", "-de0"]`、`CMD ["python"]` 或 `CMD ["php", "-a"]`。CMD 采用这种形式时，意味着当你执行类似 `docker run -it python` 这样的命令时可以直接进入到一个可用的 shell。除非您和您的预期用户已经非常熟悉 `ENTRYPOINT` 的工作方式，否则 CMD 应该很少以 `CMD ["param", "param"]` 和 ENTRYPOINT 的方式使用。

## EXPOSE
EXPOSE 指令的 [Dockerfile 参考资料](https://docs.docker.com/engine/reference/builder/#expose)

EXPOSE 指令指示开启容器的哪个端口来监听连接。应该为应用程序使用通用的传统端口。例如，包含 Apache Web 服务器的镜像将使用`EXPOSE 80`，而包含 MongoDB 的映像将使用 `EXPOSE 27017` 等。

为了使外部可以访问，用户可以在执行 `docker run` 命令时使用标志将容器的某个端口映射到用户选择的端口。对于容器链接，Docker 为从服务容器返回到源的路径（即 `MYSQL_PORT_3306_TCP`）提供环境变量。（原文：For container linking, Docker provides environment variables for the path from the recipient container back to the source (ie, MYSQL_PORT_3306_TCP).）
## ENV
ENV 指令的 [Dockerfile 参考资料](https://docs.docker.com/engine/reference/builder/#env)

要让新软件更容易运行，可以使用 ENV 来更新容器中安装的软件的 PATH 环境变量。例如，`ENV PATH /usr/local/nginx/bin:$PATH` 可以确保 `CMD ["nginx"]` 正常工作。

通过 ENV 指令可以提供所需的环境变量，指示服务按照预期运行，例如 Postgres 的 PGDATA 环境变量。

最后，ENV 还可用于设置常用的版本号，使版本更容易维护，例如下面的例子：
```
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```
跟程序中的常量（而不是硬编码值）类似，此方法可让你更改单个 ENV 指令，以自动的地处理容器中的软件版本。

跟 RUN 命令一样，每个 ENV 行会创建一个新的中间层。这意味着即使在后面的层中 unset 环境变量，这个值仍然会持久化在这个层中，其值可能会丢弃。可以通过创建类似下面的 Dockerfile 并且构建镜像来测试一下：
```
FROM alpine
ENV ADMIN_USER="mark"
RUN echo $ADMIN_USER > ./mark
RUN unset ADMIN_USER
CMD sh
```
```
$ docker run --rm -it test sh echo $ADMIN_USER

mark
```
在同一个层中使用带 shell 命令的 RUN 命令来 set、use 和 unset 变量可以避免这种情况，并且确保彻底 unset 环境变量。可以通过分号 `;` 或 `&&` 来分隔命令。使用 `&&` 时，任何一个命令执行失败都会导致镜像构建失败。这是个好主意。使用反斜线 `\` 作为行继续符号，可以提高 Linux 中 Dockerfile 的可读性。可以把所有的命令放入一个 shell 脚本中，通过 RUN 命令直接运行这个脚本。
```
FROM alpine
RUN export ADMIN_USER="mark" \
    && echo $ADMIN_USER > ./mark \
    && unset ADMIN_USER
CMD sh
```
```
$ docker run --rm -it test sh echo $ADMIN_USER
```
## ADD or COPY
ADD 指令的 [Dockerfile 参考资料](https://docs.docker.com/engine/reference/builder/#add)

COPY 指令的 [Dockerfile 参考资料](https://docs.docker.com/engine/reference/builder/#copy)

ADD 和 COPY 在功能上相似，通常来说优先使用 COPY。因为 COPY 比 ADD 更加清晰。COPY 只支持将本地文件复制到容器，而 ADD 有好几个不能一下子区分出来的特性（像只支持本地的 tar 文件提取，远程 URL 支持）。因此，ADD 的最佳用途是将本地 tar 文件自动提取到镜像中，如 `ADD rootfs.tar.xz /`。

如果 Dockerfile 中有多个步骤使用了上下文中的不同文件，挨个使用 COPY 命令，而不是一次全部完成。这可确保每个步骤的构建缓存仅在特定的所需文件发生更改时才会失效（强制重新运行该步骤）。

示例：
```dockerfile
COPY requirements.txt /tmp/
RUN pip install --requirement /tmp/requirements.txt
COPY . /tmp/
```
上面的例子中，相比使用 `COPY . /tmp/`，用于 RUN 这一步的缓存更加不容易失效。

因为镜像大小的考虑，非常不建议通过 ADD 从远程 URL 获取包，可以使用 curl 或 wget 来代替，这样可以删除在解压缩后不再需要的文件，并且不必在镜像中添加其他层。例如，**避免使用下面的例子**：
```
ADD http://example.com/big.tar.xz /usr/src/things/
RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
RUN make -C /usr/src/things all
```
相反，使用这个例子：
```
RUN mkdir -p /usr/src/things \
    && curl -SL http://example.com/big.tar.xz \
    | tar -xJC /usr/src/things \
    && make -C /usr/src/things all
```
对于其他不需要 ADD 的 tar 文件自动解压缩功能的时候，尽量使用 COPY。
## ENTRYPOINT
ENTRYPOINT 指令的 [Dockerfile 参考资料](https://docs.docker.com/engine/reference/builder/#entrypoint)

ENTRYPOINT 指令的最佳用途是设置镜像的主命令，允许该镜像像该命令一样运行（然后使用 CMD 作为默认标志）。

下面的镜像，ENTRYPOINT 设置为命令行工具 s3cmd：
```
ENTRYPOINT ["s3cmd"]
CMD ["--help"]
```
现在要查看命令的帮助可以这样运行：
```
$ docker run s3cmd
```
或使用正确的参数来执行一次命令：
```
$ docker run s3cmd ls s3://mybucket
```
这很有用，因为如上面的命令所示，镜像名称可以作为对二进制文件的二次引用。

ENTRYPOINT 指令也可以与辅助脚本结合使用，即使启动工具可能需要多个步骤，也可以使其与上述命令类似（封装到了脚本中）。

例如，[Postgres 官方镜像](https://hub.docker.com/_/postgres/) 使用下面的脚本作为其 ENTRYPOINT：
```
#!/bin/bash
set -e

if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"

    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi

    exec gosu postgres "$@"
fi

exec "$@"
```
注意：这个脚本使用了 [exec](http://wiki.bash-hackers.org/commands/builtin/exec) 这个 Bash 命令，因此最终运行的应用程序称为容器的 PID 1。这会允许应用程序接受任何发送到容器的 Unix 信号。更多信息参考 [ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#entrypoint)。

辅助脚本被复制到容器中，并且在容器启动时通过 ENTRYPOINT 运行：
```
COPY ./docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
```
这个脚本允许用户使用多种方式同 Postgres 交互。

可以简单的启动 Postgres：
```
$ docker run postgres
```
或者用来运行 Postgres 并且向服务器传参数：
```
$ docker run postgres postgres --help
```
最后，还可以用来开启完全不同的工具，比如 Bash：
```
$ docker run --rm -it postgres bash
```
## VOLUME
VOLUME 指令的 [Dockerfile 参考资料](https://docs.docker.com/engine/reference/builder/#volume)

VOLUME 指令应该用来暴露数据库存储区域、配置存储或 docker 容器创建的文件及文件夹。强烈建议将 VOLUME 用于镜像的任何可变部分和用户可用部分。
## USER
USER 指令的 [Dockerfile 参考资料](https://docs.docker.com/engine/reference/builder/#user)

如果服务运行时不需要特权，使用 USER 指令切换为非 root 用户。在 Dockerfile 中通过类似 `RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres` 的命令创建用户和用户组。

注意：镜像中的用户和用户组会得到非确定性的 UID/GID，因为不管镜像如何重建，“下一个”UID/GID 都会被分配。 所以，如果 UID/GID 很关键，就必须明确指定。

注意：由于 Go archive/tar 包处理稀疏文件（sparse files）时存在 [未解决的错误](https://github.com/golang/go/issues/13548)，试图在 Docker 容器内创建具有足够大UID的用户可能导致磁盘耗尽，因为容器层中的 `/var/log/faillog` 文件会填满 NUL（\0）字符。 将 `--no-log-init` 标志传递给 `useradd` 可以解决此问题。 Debian/Ubuntu 的 `adduser` 不支持 `--no-log-init` 标志。

避免安装或使用 `sudo`，因为它具有可能导致问题的不可预知的 TTY 和信号转发行为。 如果需要与 sudo 类似的功能，例如以 root 身份初始化守护程序，但将其作为非 root 用户运行），请考虑使用 [gosu](https://github.com/tianon/gosu)。

最后，为了减少层数和复杂性，避免频繁切换 USER。
## WORKDIR
WORKDIR 指令的 [Dockerfile 参考资料](https://docs.docker.com/engine/reference/builder/#workdir)

应该始终为 WORKDIR 使用绝对路径来保证清晰可靠。另外，应该使用 `WORKDIR` 而不是像 `RUN CD ... && do-something` 这样的繁琐指令，这些指令很难读懂、排除故障和维护。
## ONBUILD
ONBUILD 指令的 [Dockerfile 参考资料](https://docs.docker.com/engine/reference/builder/#onbuild)

ONBUILD 指令在所在的 Dockerfile 构建完成后执行。ONBUILD 在从当前镜像派生的任何子镜像中执行。可以将 ONBUILD 命令看作父 Dockerfile 给子 Dockerfile 的指令。

Docker 构建时会在执行子 Dockerfile 的任何命令之前执行 ONBUILD 命令。

ONBUILD 命令在从指定镜像构建新镜像时很有用。例如，可以为语言堆栈镜像使用 ONBUILD，在 Dockerfile 中使用该语言编写任意用户软件，就像在 [Ruby 的 ONBUILD](https://github.com/docker-library/ruby/blob/master/2.4/jessie/onbuild/Dockerfile) 变体中看到的一样。

从 ONBUILD 构建的镜像应该有一个独立的标签，例如：`ruby:1.9-onbuild` 或 `ruby:2.0-onbuild`。

在 ONBUILD 中使用 ADD 或 COPY 时需要小心。如果新构建的上下文缺少所需资源，或导致 ONBUILD 的镜像构建失败。按照上面的建议添加一个单独的标签，通过允许 Dockerfile 作者做出选择可以帮助缓解这种情况。
#3. 官方仓库示例
这些官方仓库具有示例性 Dockerfiles：

- [Go](https://hub.docker.com/_/golang/)
- [Perl](https://hub.docker.com/_/perl/)
- [Hy](https://hub.docker.com/_/hylang/)
- [Ruby](https://hub.docker.com/_/ruby/)
#4. 附加资源
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [More about Base Images](https://docs.docker.com/develop/develop-images/baseimages/)
- [More about Automated Builds](https://docs.docker.com/docker-hub/builds/)
- [Guidelines for Creating Official Repositories](https://docs.docker.com/docker-hub/official_repos/)