[原文地址](https://docs.docker.com/storage/volumes/)

卷是持久化 Docker 容器产生和使用的数据的首选方式。绑定挂载依赖主机机器的目录结构，而卷则完全由 Docker 管理。和绑定挂载相比，卷由几个优势：

- 卷比绑定挂载更容易备份和迁移。
- 可以通过 Docker CLI 或 Docker API 管理卷。
- 卷在 Linux 或 Windows 容器中都可以工作。
- 在多个容器之间共享时，卷更安全。
- 卷驱动程序允许你在远程主机或云端存储卷，加密卷内容或增加其他功能。
- 一个新卷的内容可以由容器预填充。

此外，和在容器的可写层持久化数据相比，卷通常是更好的选择，因为使用卷不会增加容器的体积，并且卷的内容脱离指定容器的生命周期而存在。

![types-of-mounts-volume](http://img.blog.csdn.net/20180307094830214?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

如果容器产生了非持久化状态数据，可以考虑使用 tmpfs 挂载避免将数据永久存储到任何位置，并且通过避免写入容器的可写层来提高容器的性能。

卷使用 rprivate 绑定传播，并且绑定传播对卷不可配置。
#1. 选择 `-v` 或 `--mount` 标志
最初，`-v` 或 `--volume` 标志用于独立容器，而 `--mount` 标志用于群集服务。但是，从 Docker 17.06 开始，也可以在独立容器上使用 `--mount`。一般来说，`--mount` 更明确和详细。最大的不同在于 `-v` 语法将所有选项组合在一个字段中，而 `--mount` 语法将它们分开。下面是每个标志的语法比较。

>知识点：初学者应该使用 `--mount` 语法。有经验的用户会更熟悉 `-v` 和 `--volume` 语法，但是仍然建议使用 `--mount` 语法，因为调查显示它更加易用。

如果需要指定卷驱动器选项，必须使用 `--mount`。

- `-v` 或 `--volume` 标志：由三个由冒号（:）分隔的字段组成。这些字段必须按照正确的顺序排列，每个字段的含义并不明显。
 - 对于命名卷，第一个字段是卷的名称，并且在给定主机上是唯一的。对于匿名卷，第一个字段被省略。
 - 第二个字段是文件或目录在容器中的挂载路径。
 - 第三个字段是可选的，并且是一个逗号分隔的选项列表，例如 ro。这些选项在下面讨论。
- `--mount` 标志：由多个名值对组成，逗号分隔，每个键值由 `<key> = <value>` 元组组成。`--mount` 语法比 `-v` 或 `--volume` 更冗长，但键的顺序并不重要，并且标志的值更易于理解。
 - 要挂载的类型 `type`，可以是 bind、volume 或 tmpfs。本主题主要使用 volume。
 - 要挂载的源 `source`，对于有名字的卷，这里是卷的名字。对于匿名卷忽略这个字段。可以指定为 `src` 或 `source`。
 - 要挂载的目的地 `destination`，将文件或目录挂载在容器中的路径作为其值。 可能被指定为 `destination`、`dst` 或 `target`。
 - 只读选项 `readonly`，这个选项会使得挂载到容器中的绑定挂载只读。
 - 选项 `volume-opt`，可以被多次指定，由包含选项名和值的名值对组成。

下面的例子在可能的地方展示了 `--mount` 和 `-v` 语法。
##1.1  `--mount` 和 `-v` 之间的表现差异
与绑定挂载不同，卷的所有选项都可用于 `--mount` 和 `-v` 标志。

当在服务上使用卷时，仅支持 `--mount`。
#2. 创建和管理卷
与绑定挂载不同，可以通过 `docker volume` 命令在任何容器范围之外创建和管理卷。

创建卷：
```
$ docker volume create my-vol
```
列出卷：
```
$ docker volume ls

local               my-vol
```
检查卷：
```
$ docker volume inspect my-vol
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
```
删除卷：
```
$ docker volume rm my-vol
```
#3. 启动带有卷的容器
启动带有卷的容器时，如果卷不存在，Docker 会自动创建这个卷。下面的例子将卷 `myvol2` 挂载到容器中的 `/app/`。

下面的 `--mount` 和 `-v` 示例产生相同的结果。注意不能同时运行，除非在运行第一个例子之后删除 `devtest` 容器和 `myvol2` 卷。

- `--mount` 
```
$ docker run -d \
  --name devtest \
  --mount source=myvol2,target=/app \
  nginx:latest
```
- `-v`
```
$ docker run -d \
  --name devtest \
  -v myvol2:/app \
  nginx:latest
```
通过 `docker inspect devtest` 来验证卷被正确的创建和挂载。查看 Mounts 部分：
```
"Mounts": [
    {
        "Type": "volume",
        "Name": "myvol2",
        "Source": "/var/lib/docker/volumes/myvol2/_data",
        "Destination": "/app",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
```
这显示挂载的是一个卷，并显示了正确的 source 和 destination，并且是读写模式。

停止容器并删除卷：
```
$ docker container stop devtest

$ docker container rm devtest

$ docker volume rm myvol2
```
##3.1 启动带有卷的服务
当启动服务并定义卷时，每个服务容器都使用其自己的本地卷。如果使用本地卷驱动程序，则任何容器都不能共享此数据，但某些卷驱动程序确实支持共享存储。 AWS 的 Docker 和 Azure 的 Docker 都使用 Cloudstor 插件支持持久存储。

下面的例子启动了一个具有 4 个副本的 Nginx 服务，每个副本都使用一个名为 `myvol2` 的本地卷。
```
$ docker service create -d \
  --replicas=4 \
  --name devtest-service \
  --mount source=myvol2,target=/app \
  nginx:latest
```
通过 `docker service ps devtest-service` 来验证服务正在运行：
```
$ docker service ps devtest-service

ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
4d7oz1j85wwn        devtest-service.1   nginx:latest        moby                Running             Running 14 seconds ago   
```
删除服务，这会停止其所有任务：
```
$ docker service rm devtest-service
```
####使用服务的语法差异
`docker service create` 命令不支持  `-v` 或 `--volume` 标志。在挂载卷到服务中的容器时，必须使用 `--mount` 标志。
##3.2 使用容器填充卷
如果启动一个创建新卷的容器（如上所述），并且在容器的要挂载的目录中具有文件或目录（例如 ` /app/ above`），则将该目录的内容会复制到卷中。容器然后挂载并使用该卷，而使用卷的其他容器也可以访问预先填充的内容。

为了说明这一点，本示例启动一个 Nginx 容器，并使用容器的 `/usr/share/nginx/html` 目录（Nginx 存储其默认 HTML 内容的地方）的内容填充新卷 `nginx-vol`。

`--mount` 和 `-v` 示例的结果相同：

- `--mount` 
```
$ docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html \
  nginx:latest
```
- `-v`
```
$ docker run -d \
  --name=nginxtest \
  -v nginx-vol:/usr/share/nginx/html \
  nginx:latest
```
在运行了这些示例中的一个之后，运行下面的命令来清除容器和卷。
```
$ docker container stop nginxtest

$ docker container rm nginxtest

$ docker volume rm nginx-vol
```
#4. 使用只读卷
对于某些开发的应用程序，容器需要写入绑定挂载，以便将更改传到 Docker 主机。在其他时候，容器只需要读取数据。请记住，多个容器可以挂载相同的卷，并且可以同时对它们中的某些容器进行读写挂载，对其他容器进行只读。

这个例子修改了上面的例子，但是通过在 `--mount` 列出的选项中添加 `ro` 将目录挂载为只读卷，如果存在多个选项，用逗号分隔。

`--mount` 和 `-v` 示例的结果相同：

- `--mount` 
```
$ docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html,readonly \
  nginx:latest
```
- `-v`
```
$ docker run -d \
  --name=nginxtest \
  -v nginx-vol:/usr/share/nginx/html:ro \
  nginx:latest
```
通过 `docker inspect nginxtest` 来验证绑定挂载创建正确。查看 Mounts 部分：
```
"Mounts": [
    {
        "Type": "volume",
        "Name": "nginx-vol",
        "Source": "/var/lib/docker/volumes/nginx-vol/_data",
        "Destination": "/usr/share/nginx/html",
        "Driver": "local",
        "Mode": "",
        "RW": false,
        "Propagation": ""
    }
],
```
停止并删除容器，删除卷：
```
$ docker container stop nginxtest

$ docker container rm nginxtest

$ docker volume rm nginx-vol
```
#5. 使用卷驱动程序
通过 `docker volume create` 命令创建卷时，或启动一个包含尚未创建的卷的容器时，可以指定卷驱动程序。下面的例子使用 `vieux/sshfs` 卷驱动程序，第一次是创建独立卷，第二次是启动容器时创建新卷。
##5.1 初始化设置
这个例子假设你有两个节点，第一个节点是 Docker 主机并且可以通过 SSH 连接到第二个。

在 Docker 主机上，安装 `vieux/sshfs` 插件：
```
$ docker plugin install --grant-all-permissions vieux/sshfs
```
##5.2 使用卷驱动程序创建卷
这个例子指定一个 SSH 密码，但是如果两个主机被配置为共享密钥，则可以省略该密码。每个卷驱动程序可能有零个或多个可配置选项，每个可选项都使用 `-o` 标志指定。
```
$ docker volume create --driver vieux/sshfs \
  -o sshcmd=test@node2:/home/test \
  -o password=testpassword \
  sshvolume
```
##5.3 启动会使用卷驱动程序创建卷的容器
这个例子指定一个 SSH 密码，但是如果两个主机被配置为共享密钥，则可以省略该密码。每个卷驱动程序可能有零个或多个可配置选项，每个可选项都使用 `-o` 标志指定。如果卷驱动程序需要你传输选项，则必须使用 `--mount` 标志来挂载卷，而不能使用 `-v`。
```
$ docker run -d \
  --name sshfs-container \
  --volume-driver vieux/sshfs \
  --mount src=sshvolume,target=/app,volume-opt=sshcmd=test@node2:/home/test,volume-opt=password=testpassword \
  nginx:latest
```