[原文地址](https://docs.docker.com/engine/security/userns-remap/)

Linux 命名空间为运行中的进程提供了隔离，限制他们对系统资源的访问，而进程没有意识到这些限制。有关 Linux 命名空间的更多信息，请参阅 [Linux 命名空间](https://www.linux.com/news/understanding-and-securing-linux-namespaces)。

防止容器内的特权升级攻击的最佳方法是将容器的应用程序配置为作为非特权用户运行。对于其进程必须作为容器中的 root 用户运行的容器，可以将此用户重新映射到 Docker 主机上权限较低的用户。映射的用户被分配了一系列 UID，这些 UID 在命名空间内作为从 0 到 65536 的普通 UID 运行，但在主机上没有特权。
# 1. 关于重新映射和从属用户和组的 ID
重新映射本身由两个文件处理：`/etc/subuid` 和 `/etc/subgid`。每个文件的工作原理相同，但其中一个关注用户 ID 范围，另一个关注组 ID 范围。考虑 `/etc/subuid` 中的以下条目：
```
testuser:231072:65536
```
这意味着 testuser 将从 231072 开始，在后面的 65536 个整数中按顺序为用户分配一个 ID。例如，命名空间中的 UID 231072 映射到容器中的 UID 0（root），UID 231073 映射为 UID 1，依此类推。如果某个进程尝试提升特权到命名空间外部，则该进程将作为主机上无特权的高数字 UID 运行，该 UID 甚至不映射到真实用户。这意味着该进程完全没有主机系统的权限。

>多个范围
>
>通过为  `/etc/subuid` 或 `/etc/subgid` 文件中的同一用户或组添加多个不重叠的映射，可以为给定的用户或组分配多个从属范围。在这种情况下，根据内核对 `/proc/self/uid_map` 和 `/proc/self/gid_map` 中只有五个条目的限制，Docker 仅使用前五个映射。

当你将 Docker 配置为使用 `userns-remap` 功能时，可以选择指定现有用户和/或组，也可以指定为 `default`。如果指定为 `default`，则会为此创建并使用用户和组 `dockremap`。

>警告：某些发行版（如 RHEL 和 CentOS 7.3）不会自动将新组添加到 `/etc/subuid` 和 `/etc/subgid` 文件中。此时，需要手动编辑这些文件并分配不重叠的范围。下一小节**先决条件**中介绍了此步骤。

范围不重叠非常重要，这样一个进程无法访问不同的命名空间。在大多数 Linux 发行版中，添加或删除用户时，系统工具会替你管理这些范围。

此重新映射对容器来说是透明的，但在容器需要访问 Docker 主机上的资源（例如绑定挂载到系统用户无法写入的文件系统区域）的情况下引入了一些配置复杂性。从安全角度来看，最好避免这些情况。
# 2. 先决条件
① 即使关联是实现细节，下级 UID 和 GID 范围也必须与现有用户关联。用户拥有 `/var/lib/docker/` 下的命名空间存储目录。如果不想使用现有的用户，Docker 可以为你创建一个并使用它。如果想使用现有的用户名或用户 ID，则它必须已经存在。通常，这意味着相关条目需要位于 `/etc/passwd` 和 `/etc/group` 中，但如果你使用的是不同的身份验证后端，则此要求可能会有所不同。

通过 `id` 命令验证这一点：
```
$ id testuser

uid=1001(testuser) gid=1001(testuser) groups=1001(testuser)
```
② 在主机上处理命名空间重映射的方式是使用两个文件 `/etc/subuid` 和 `/etc/subgid`。这些文件通常在添加或删除用户或组时自动管理，但在少数发行版（如 RHEL 和 CentOS 7.3）中，可能需要手动管理这些文件。

每个文件包含三个字段：用户的 username 或 ID，后跟开头的 UID 或 GID（在命名空间内被视为 UID 或 GID 0）以及用户可用的最大数量的 UID 或 GID。例如，给出以下条目：
```
testuser:231072:65536
```
这意味着由 testuser 启动的用户命名空间中的进程对应主机的 UID 范围是 231072 （对应命名空间中的 UID 0）到 296608。这些范围不应该重叠，这种命名空间中的进程才不会访问到其他命名空间中的进程。

添加完你的用户后，检查 `/etc/subuid` 和 `/etc/subgid` 文件看看是否分别为这个用户创建了一个条目。如果没有，那就需要手动添加，注意不要重叠。

如果你想通过 Docker 自动创建 `dockremap` 用户，在重新配置并重启 Docker 后检查这些文件中的 `dockremap` 条目。

③ 如果非特权用户需要写入 Docker 主机上的某个位置，调整这个位置的相关权限。这也适用于通过 Docker 自动创建 `dockremap` 用户，只是在重新配置并重启 Docker 之前无法修改权限。

④ 启用 `userns-remap` 可有效地屏蔽现有镜像和容器层以及 `/var/lib/docker/` 中的其他 Docker 对象。这是因为 Docker 需要调整这些资源的所有权，并将它们实际存储在 `/var/lib/docker/` 中的子目录中。最好在新安装的 Docker 中启用此功能，而不是现有的。

同样，如果禁用 `userns-remap`，则无法访问启用时创建的任何资源。

⑤ 检查用户命名空间的限制，以确保可行。
# 3. 在守护进程上开启 userns-remap
可以使用 `--userns-remap` 标志启动 `dockerd`，或者按照这个步骤使用 `daemon.json` 配置文件来配置守护进程。建议使用 `daemon.json` 配置文件。如果使用标志，参考下面的命令：
```
$ dockerd --userns-remap="testuser:testuser"
```
① 编辑 `/etc/docker/daemon.json`。假设文件之前是空的，下面的条目会使用名为 testuser 的用户和组来开启 `userns-remap`。可以通过 ID 或 name 来寻址用户和组。如果组名或 ID 与用户名或 ID 不同，只需指定组名或 ID。如果同时提供用户和组的 name 或 ID，需要使用冒号（:)字隔。假设 testuser 的 UID 和 GID 是 1001，以下格式都适用于该值：

- `testuser`
- `testuser:testuser`
- `1001`
- `1001:1001`
- `testuser:1001`
- `1001:testuser`
```
{
  "userns-remap": "testuser"
}
```
>注意：要使用 `dockermap` 用户并且让 Docker 替你创建，请将该值设置为默认值而不是 testuser。

保存文件并重启 Docker。

② 如果你在使用 `dockermap` 用户，通过 `id` 命令验证 Docker 已经创建了这个用户。
```
$ id dockremap

uid=112(dockremap) gid=116(dockremap) groups=116(dockremap)
```
验证条目已经添加到了 `/etc/subuid` 和 `/etc/subgid` 文件中。
```
$ grep dockremap /etc/subuid

dockremap:231072:65536

$ grep dockremap /etc/subgid

dockremap:231072:65536
```
如果这些条目不存在，需要以 root 用户身份编辑文件，并且分配起始的 UID 和 GID（在最高的已经分配的值的基础上加上偏移，65536）。注意不要使范围重叠。

③ 通过 `docker image ls` 命令验证之前的镜像已经不可用。输出应该为空。

④ 从 `hello-world` 镜像启动容器。
```
$ docker run hello-world
```
⑤ 验证名称空间目录是否存在于 `/var/lib/docker/` 目录中，目录名称由命名空间中用户的 UID 和 GID 命名，由该 UID 和 GID 拥有，而不是组或全局可读的。某些子目录仍由 root 拥有，并具有不同的权限。
```
$ sudo ls -ld /var/lib/docker/231072.231072/

drwx------ 11 231072 231072 11 Jun 21 21:19 /var/lib/docker/231072.231072/

$ sudo ls -l /var/lib/docker/231072.231072/

total 14
drwx------ 5 231072 231072 5 Jun 21 21:19 aufs
drwx------ 3 231072 231072 3 Jun 21 21:21 containers
drwx------ 3 root   root   3 Jun 21 21:19 image
drwxr-x--- 3 root   root   3 Jun 21 21:19 network
drwx------ 4 root   root   4 Jun 21 21:19 plugins
drwx------ 2 root   root   2 Jun 21 21:19 swarm
drwx------ 2 231072 231072 2 Jun 21 21:21 tmp
drwx------ 2 root   root   2 Jun 21 21:19 trust
drwx------ 2 231072 231072 3 Jun 21 21:19 volumes
```
你看到的目录列表可能会有所差异，如果你使用的存储驱动程序不是 `aufs` 时差异更大。

使用重映射用户所拥有的目录而不是直接位于 `/var/lib/docker/` 下的相同目录，并且可以删除未使用的版本（例如此处的示例中的 `/var/lib/docker/tmp/`）。在启用 `userns-remap` 时，Docker 不使用它们。
# 4. 关闭容器的命名空间重映射
如果在守护进程上启用用户命名空间，则所有容器都将默认启用用户命名空间。在某些情况下，例如特权容器，可能需要禁用特定容器的用户命名空间。有关这些限制的其中一部分，请参阅下一小节**用户名称空间已知限制**。

要为指定容器关闭用户命名空间，给 `docker container create`、`docker container run` 或 `docker container exec` 命令中添加 `--userns=host` 标志，
# 5. 用户命名空间的已知限制
以下标准 Docker 功能与启用用户命名空间的 Docker 守护进程不兼容：

- 共享主机的 PID 或 NET 命名空间（`--pid=host` 或 `--network=host`）。
- 外部（卷或存储）驱动程序不知道或不能使用守护进程的用户映射。
- 在 `docker run` 命令中使用 `--privileged` 模式，但是没有指定 `--userns=host`。

用户名称空间是一项高级功能，需要与其他功能协调。例如，如果卷从主机挂载，则必须预先安排文件所有权，以便对卷内容进行读取或写入访问。

尽管在使用了用户命名空间的容器进程中的 root 用户具有容器中超级用户的许多期望特权，但 Linux 内核根据内部知识限制这个用户命名空间中的进程。一个显着的限制是无法使用 `mknod` 命令。由 root 用户运行时，容器内的设备创建权限被拒绝。
原文：
While the root user inside a user-namespaced container process has many of the expected privileges of the superuser within the container, the Linux kernel imposes restrictions based on internal knowledge that this is a user-namespaced process. One notable restriction is the inability to use the mknod command. Permission is denied for device creation within the container when run by the root user.
