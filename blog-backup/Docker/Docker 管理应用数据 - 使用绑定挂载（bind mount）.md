[原文地址](https://docs.docker.com/storage/bind-mounts/)

绑定挂载在 Docker 早期就已经存在。与卷相比，绑定挂载功能有限。当使用绑定挂载时，主机上的文件或目录被挂载到容器中。文件或目录由主机上的完整路径或相对路径引用。相比之下，当使用卷时，会在主机上的 Docker 存储目录中创建一个新目录，并且 Docker 会管理该目录的内容。

文件或目录不需要已经存在在 Docker 主机上。如果不存在，会按需创建。绑定挂载性能很好，但它们依赖于具有特定目录结构的主机的文件系统。如果你正在开发新的 Docker 应用程序，请考虑使用命名过的卷。不能使用 Docker CLI 命令直接管理绑定挂载。

![types-of-mounts-bind](http://img.blog.csdn.net/20180307172542384?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
#1. 选择 `-v` 或 `--mount` 标志
最初，`-v` 或 `--volume` 标志用于独立容器，而 `--mount` 标志用于群集服务。但是，从 Docker 17.06 开始，也可以在独立容器上使用 `--mount`。一般来说，`--mount` 更明确和详细。最大的不同在于 `-v` 语法将所有选项组合在一个字段中，而 `--mount` 语法将它们分开。下面是每个标志的语法比较。

>知识点：初学者应该使用 `--mount` 语法。有经验的用户会更熟悉 `-v` 和 `--volume` 语法，但是仍然建议使用 `--mount` 语法，因为调查显示它更加易用。

如果需要指定卷驱动器选项，必须使用 `--mount`。

- `-v` 或 `--volume` 标志：由三个由冒号（:）分隔的字段组成。这些字段必须按照正确的顺序排列，每个字段的含义并不明显。
 - 对于命名卷，第一个字段是卷的名称，并且在给定主机上是唯一的。对于匿名卷，第一个字段被省略。
 - 第二个字段是文件或目录在容器中的挂载路径。
 - 第三个字段是可选的，并且是一个逗号分隔的选项列表，例如` ro`、`consistent`、`delegated`、`cached`、`z` 和 `Z`。这些选项在下面讨论。
- `--mount` 标志：由多个名值对组成，逗号分隔，每个键值由 `<key> = <value>` 元组组成。`--mount` 语法比 `-v` 或 `--volume` 更冗长，但键的顺序并不重要，并且标志的值更易于理解。
 - 要挂载的类型 `type`，可以是 bind、volume 或 tmpfs。本主题主要使用 volume。
 - 要挂载的源 `source`，对于有名字的卷，这里是卷的名字。对于匿名卷忽略这个字段。可以指定为 `src` 或 `source`。
 - 要挂载的目的地 `destination`，将文件或目录挂载在容器中的路径作为其值。 可能被指定为 `destination`、`dst` 或 `target`。
 - 只读选项 `readonly`，这个选项会使得挂载到容器中的绑定挂载只读。
 - 选项 `bind-propagation`，如果有这个选项则会改变绑定传播（bind propagation，参考下面部分）。可以是 `rprivate`, `private`, `rshared`, `shared`, `rslave`, `slave`。
 - 选项 `consistency`，仅用于 Docker for Mac。
 - `--mount` 标志不支持用于修改 selinux 标签的 `z` 和 `Z` 选项。

下面的例子在可能的地方展示了 `--mount` 和 `-v` 语法。
##1.1  `--mount` 和 `-v` 之间的表现差异
因为 `-v` 或 `--volume` 标志在 Docker 已经使用了很久了，无法改变其表现。这意味着在 `--mount` 和 `-v` 之间的表现有差异。

如果使用 `-v` 或 `--volume` 标志来绑定挂载 Docker 主机中不存在的文件或目录，`-v` 会为你创建一个端点。它会始终创建目录。

如果使用 `--mount` 标志来绑定挂载 Docker 主机中不存在的文件或目录，Docker 不会自动为你创建，而是产生报错。
#2. 启动带有绑定挂载的容器
考虑这样的情况，你有源目录，并且当你构建源代码时，文件被保存到另一个目录 `source/target/` 中。你希望文件在容器的 `/app/` 中可用，并且希望容器每次在开发主机构建源代码时都可以访问新的构建。使用以下命令将 `target/` 目录绑定到容器的 `/app/` 中。从源目录中运行该命令。`$(pwd)` 子命令扩展到 Linux 或 macOS 主机上的当前工作目录。

下面的 `--mount` 和 `-v` 示例产生相同的结果。注意不能同时运行，除非在运行第一个例子之后删除 `devtest` 容器。

- `--mount`：
```
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  nginx:latest
```
- `-v`：
```
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app \
  nginx:latest
```
通过 `docker inspect devtest` 来验证绑定挂载创建成果。查看 Mounts 部分：
```
"Mounts": [
    {
        "Type": "bind",
        "Source": "/tmp/source/target",
        "Destination": "/app",
        "Mode": "",
        "RW": true,
        "Propagation": "rprivate"
    }
],
```
这表明挂载方式是绑定挂载，源和目的地都正确，挂载为读写模式，传播方式是 `rprivate`。

停止容器：
```
$ docker container stop devtest

$ docker container rm devtest
```
##2.1 挂载到容器中的非空目录
如果你绑定挂载到容器的非空目录，这个目录中的已有内容会被绑定挂载遮盖（就像先将文件保存到 Linux 主机上的 `/mnt` 中，然后将一个 USB 驱动器挂载到 `/mnt` 中一样。在卸载 USB 驱动器之前，`/mnt` 中的内容将被 USB 驱动器的内容遮盖。遮盖的文件不会被删除或更改，但在装入绑定挂载或卷时不可访问）。这可能是有益的，例如，当你想要测试新版本的应用程序而无需构建新映像时。然而，这也可能令人惊讶，并且这种行为不同于 docker 卷的行为。

这个例子被认为是极端的，但是用主机上的 `/tmp/` 目录替换了容器的 `/usr/` 目录的内容。 在大多数情况下，这会导致容器无法正常工作。

`--mount` 和 `-v` 示例的结果相同：

- `--mount`：
```
$ docker run -d \
  -it \
  --name broken-container \
  --mount type=bind,source=/tmp,target=/usr \
  nginx:latest
  
docker: Error response from daemon: oci runtime error: container_linux.go:262:
starting container process caused "exec: \"nginx\": executable file not found in $PATH".
```
- `-v`：
```
$ docker run -d \
  -it \
  --name broken-container \
  -v /tmp:/usr \
  nginx:latest
  
docker: Error response from daemon: oci runtime error: container_linux.go:262:
starting container process caused "exec: \"nginx\": executable file not found in $PATH".
```
容器虽然创建了，但是无法工作。删除这个容器：
```
$ docker container rm broken-container
```
#3. 使用只读的绑定挂载
对于某些开发的应用程序，容器需要写入绑定挂载，这样变化会被传回 Docker 主机。其他时候，容器只需要读取即可。

这个例子修改了上面的例子，但是通过在 `--mount` 之后将 `ro` 添加到（默认情况下为空）选项列表来将目录挂载为只读绑定挂载。如果存在多个选项，用逗号分隔。

`--mount` 和 `-v` 示例的结果相同：

- `--mount`：
```
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app,readonly \
  nginx:latest
```
- `-v`：
```
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app:ro \
  nginx:latest
```
通过 `docker inspect devtest` 来验证绑定挂载创建成功。检查 Mounts 部分：
```
"Mounts": [
    {
        "Type": "bind",
        "Source": "/tmp/source/target",
        "Destination": "/app",
        "Mode": "ro",
        "RW": false,
        "Propagation": "rprivate"
    }
],
```
停止容器：
```
$ docker container stop devtest

$ docker container rm devtest
```
#4. 配置绑定传播（bind propagation）
绑定挂载和卷中默认的绑定传播类型是 `rprivate`。只有在 Linux 主机上的 绑定挂载才是可配置的。绑定传播是高级主题，多数用户不需要配置。

绑定传播是指在给定的绑定挂载或命名卷中创建的挂载是否可以传播到该挂载的副本。考虑一个挂载点 `/mnt`，它也挂载在`/tmp` 上。传播设置控制 `/tmp/a` 上的挂载是否可以通过 `/mnt/a` 访问。每个传播设置都有一个递归对应点（recursive counterpoint）。在递归的情况下，考虑 `/tmp/a` 也被挂载为 `/foo`。传播设置控制是否存在 `/mnt/a` 和/或 `/tmp/a`。
原文：
Bind propagation refers to whether or not mounts created within a given bind-mount or named volume can be propagated to replicas of that mount. Consider a mount point /mnt, which is also mounted on /tmp. The propagation settings control whether a mount on /tmp/a would also be available on /mnt/a. Each propagation setting has a recursive counterpoint. In the case of recursion, consider that /tmp/a is also mounted as /foo. The propagation settings control whether /mnt/a and/or /tmp/a would exist.

|Propagation 设置|  描述|
|-|-|
|`shared`|  Sub-mounts of the original mount are exposed to replica mounts, and sub-mounts of replica mounts are also propagated to the original mount.
|`slave`| similar to a shared mount, but only in one direction. If the original mount exposes a sub-mount, the replica mount can see it. However, if the replica mount exposes a sub-mount, the original mount cannot see it.
|`private`| The mount is private. Sub-mounts within it are not exposed to replica mounts, and sub-mounts of replica mounts are not exposed to the original mount.
|`rshared`| The same as shared, but the propagation also extends to and from mount points nested within any of the original or replica mount points.
|`rslave`|  The same as slave, but the propagation also extends to and from mount points nested within any of the original or replica mount points.
|`rprivate`|  The default. The same as private, meaning that no mount points anywhere within the original or replica mount points propagate in either direction.

在可以为挂载点设置绑定传播之前，主机的文件系统需要指出绑定传播。

更多关于绑定传播的信息，参考 [Linux kernel documentation for shared subtree](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)。

下面的例子将 `target/` 目录两次挂载到容器中，并且第二次挂载时同时设置了 `ro` 选项和 `rslave` 绑定传播选项。


`--mount` 和 `-v` 示例的结果相同：

- `--mount`：
```
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  --mount type=bind,source="$(pwd)"/target,target=/app2,readonly,bind-propagation=rslave \
  nginx:latest
```
- `-v`：
```
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app \
  -v "$(pwd)"/target:/app2:ro,rslave \
  nginx:latestNow
```
如果你创建了 `/app/foo/`，`/app2/foo/`也会存在。
#5. 配置 selinux 标签
如果你在使用 selinux，可以添加 `z` 或 `Z` 选项将 selinux 标签设置为被挂载到容器中的主机文件或目录。这会影响主机本身的文件或目录，并可能导致 Docker 之外的影响。

- `z` 选项指示绑定挂载的内容在多个容器直接共享。
- `Z` 选项指示绑定挂载的内容私有且不共享。

对这些选项要特别小心。使用 `Z` 选项绑定系统目录（如 `/home` 或 `/usr`）会导致主机无法操作，您可能需要手动重新标记主机文件。

>重点：在服务上使用绑定挂载时，selinux 标签（`:Z` 和 `:z`）和 `:ro` 都会被忽略，详情参考 [moby/moby #32579](https://github.com/moby/moby/issues/32579)

这个例子通过设置 `z` 选项使绑定挂载的内容可以在多个容器之间共享：

无法使用 `--mount` 标志来修改 selinux 标签。
```
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app:z \
  nginx:latest
```
#6. 配置 macOS 的挂载一致性
Mac 版 Docker 使用 osxfs 将目录和文件从 macOS 共享到 Linux VM。这种传播使这些目录和文件可用于在 Docker for Mac 上运行的 Docker 容器。

默认情况下，这些共享是完全一致的，这意味着每次在 macOS 主机上发生写入或通过容器挂载时，都会将更改刷新到磁盘，以便共享中的所有参与者都具有完全一致的视图。在某些情况下，完全一致可能会严重影响性能。Docker 17.05 及更高版本引入了选项来调整每个安装，每个容器的一致性设置。以下选项可用：

- `consistent` or default: The default setting with full consistency, as described above.
- `delegated`: The container runtime’s view of the mount is authoritative. There may be delays before updates made in a container are visible on the host.
- `cached`: The macOS host’s view of the mount is authoritative. There may be delays before updates made on the host are visible within a container.

这些选项只会被 macOS 接受。

`--mount` 和 `-v` 示例的结果相同：

- `--mount`：
```
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,destination=/app,consistency=cached \
  nginx:latest
```
- `-v`：
```
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app:cached \
  nginx:latest
```