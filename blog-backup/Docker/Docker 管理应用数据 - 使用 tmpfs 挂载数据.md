[原文地址](https://docs.docker.com/storage/tmpfs/)

默认情况下，卷和绑定挂载将挂载到容器的文件系统中，并将其内容存储在主机上。

有时候你不想把容器的数据存储在主机上，但出于性能或安全原因，或者如果数据是不需要持久化的状态信息，你也不希望将数据写入容器的可写层。例如容器的应用程序根据需要创建和使用的一次性临时密码。

为了让容器能够访问数据而不需要永久地写入数据，可以使用 `tmpfs` 挂载，该挂载仅存储在主机的内存中（如果内存不足，则为 swap）。当容器停止时，`tmpfs` 挂载会被移除。如果提交容器，则不会保存 `tmpfs` 挂载。

![types-of-mounts-tmpfs](http://img.blog.csdn.net/20180307183529584?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
#1. 选择 `--tmpfs` 或 `--mount` 标志
最初，`--tmpfs` 标志用于独立容器，而 `--mount` 标志用于 swarm 服务。但是，从 Docker 17.06 开始，也可以在独立容器上使用 `--mount`。一般来说，`--mount` 更明确和详细。最大的不同在于 `--tmpfs` 语法将所有选项组合在一个字段中，而 `--mount` 语法将它们分开。下面是每个标志的语法比较。

>知识点：初学者应该使用 `--mount` 语法。有经验的用户会更熟悉 `--tmpfs` 语法，但是仍然建议使用 `--mount` 语法，因为调查显示它更加易用。

- `--tmpfs`: Mounts a tmpfs mount without allowing you to specify any configurable options, and can only be used with standalone containers.

- `--mount` 标志：由多个名值对组成，逗号分隔，每个键值由 `<key> = <value>` 元组组成。`--mount` 语法比 `--tmpfs` 更冗长，但键的顺序并不重要，并且标志的值更易于理解。
 - 要挂载的类型 `type`，可以是 bind、volume 或 tmpfs。本主题主要使用 tmpfs。
 - 要挂载的目的地 `destination`，将文件或目录挂载在容器中的路径作为其值。 可能被指定为 destination、dst 或 target。
 - `tmpfs-type` 和 `tmpfs-mode` 选项。

下面的例子在可能的地方展示了 `--mount` 和 `--tmpfs` 语法。
##1.1 `--tmpfs` 和 `--mount` 之间的表现差异
- `--tmpfs` 标志不支持任何配置选项。
- `--tmpfs` 标志不能用于 swarm 服务。必须使用 `--mount`。
#2. tmpfs 容器的限制
- `tmpfs` 挂载不能在容器之间共享。
- `tmpfs` 挂载只能用于 Linux 容器，不支持 Windows 容器。
#3. 在容器中使用 `tmpfs` 挂载
要在容器中使用 `tmpfs` 挂载，使用 `tmpfs` 标志，或使用在容器中使用 `--mount` 标志同时指定 `type=tmpfs` 和 `destination` 选项。下面的例子在 Nginx 容器中的 `/app` 创建了一个 `tmpfs` 挂载。第一个例子使用 `--mount` 标志，第二个使用 `--tmpfs` 标志。

- `--mount`
```
$ docker run -d \
  -it \
  --name tmptest \
  --mount type=tmpfs,destination=/app \
  nginx:latest
```
- `--tmpfs`
```
$ docker run -d \
  -it \
  --name tmptest \
  --tmpfs /app \
  nginx:latest
```
通过 `docker container inspect tmptest` 来验证挂载类型是 `tmpfs`，查看 Mounts 部分：
```
"Tmpfs": {
    "/app": ""
},
```
删除容器：
```
$ docker container stop tmptest

$ Docker container rm tmptest
```
##3.1 指定 `tmpfs` 选项
`tmpfs` 挂载允许两个配置选项，都不是必须的。如果你需要指定这些选项，必须使用 `--mount` 标志。

|选项|  描述|
|-|-|
|`tmpfs-size`|  tmpfs 挂载的大小，单位字节。默认无限制
|`tmpfs-mode`|  tmpfs 的八进制文件模式。例如，700或0770。默认为 1777 或所有人都可写。
下面的示例将 `tmpfs-mode` 设为 1770，这样容器中就不是所有人都可写的。
```
docker run -d \
  -it \
  --name tmptest \
  --mount type=tmpfs,destination=/app,tmpfs-mode=1770 \
  nginx:latest
```