[原文地址](https://docs.docker.com/storage/storagedriver/select-storage-driver/)

理想情况下，只有很少的数据写入容器的可写层，并且使用 Docker 卷来写入这些数据。但是，有些工作负载要求写入容器的可写层。这是使用存储驱动程序的地方。

Docker 通过插件机制支持几种不同的存储驱动程序。存储驱动程序控制镜像和容器在 Docker 主机上的存储和管理方式。

在读完了 [关于存储驱动程序](http://blog.csdn.net/kikajack/article/details/79502581) 这一部分之后，下一步是选择最适合你的工作负载的存储驱动程序。在作出这一决定时，需要考虑三个高层次因素：

####1. 如果你的内核支持多个存储驱动程序，那么假定满足该存储驱动程序的先决条件，则在没有明确配置存储驱动程序的情况下，Docker 会列出要使用哪个存储驱动程序的优先级列表：
- 如果可能的话，将使用具有最少配置的存储驱动程序，例如 `btrfs` 或 `zfs`。
- 其他情况下，尝试使用具有最佳整体性能和稳定性的存储驱动程序。

 - `overlay2` 是首选，然后是 `overlay`。这些都不需要额外的配置。Docker CE 的默认选择是`overlay2`。
 - `devicemapper` 是第二选择但生产环境需要 `direct-lvm`，因为 `loopback-lvm` 虽然是零配置，但性能很差。

这个选择顺序是在 Docker 的源代码中定义的。可以在 [Docker CE 17.12 的源码](https://github.com/docker/docker-ce/blob/17.12/components/engine/daemon/graphdriver/driver_linux.go#L54-L63) 中查看这个顺序。如果你使用的是不同版本的 Docker，可以使用文件视图顶部的分支选择器选择一个不同的分支。
####2. 因为 Docker 版本或操作系统或 Linux 发行版的不同，可选的存储驱动程序也会有所不同。例如，`aufs` 只支持 Ubuntu 和 Debian，并需要安装额外的包；而 `btrfs` 只支持 SLES，而这个版本上只能安装 Docker EE。在下面的第一部分查看每个 Linux 发行版支持的存储驱动程序有哪些。
####3. 某些存储驱动程序匹配特定格式的 backing filesystem。如果你有使用特定 backing filesystem 的外部要求，这可能会限制你的选择。在下面的第二部分参阅支持的 backing filesystem。
####4. 在缩小了可供选择的存储驱动程序之后，你的选择取决于工作负载的特征和所需的稳定级别。在下面的第二部分参阅其他注意事项以帮助作出最终决定。
#1. 每个 Linux 发行版支持的存储驱动程序
在较高级别上，可以使用的存储驱动程序部分取决于你使用的 Docker 版本。

此外，Docker 不建议任何关闭操作系统安全特性的配置，例如如果在 CentOS 上使用 `overlay` 或 `overlay2` 时需要关闭 selinux。
##1.1 Docker EE 和 CS-Engine
对于 Docker EE 和 CS-Engine，支持存储驱动程序的权威资源是 [产品兼容性矩阵](https://success.docker.com/Policies/Compatibility_Matrix)。要获得 Docker 的商业支持，必须使用受支持的配置。
##1.2 Docker CE
对于 Docker CE，只测试部分配置，并且操作系统的内核可能不支持每个存储驱动程序。通常，以下配置适用于最新版本的 Linux 发行版：

|Linux 发行版| 建议使用的存储驱动程序|
|-|-|
|Docker CE on Ubuntu| aufs, devicemapper, overlay2 (Ubuntu 14.04.4 或更高版本，16.04 或更高版本), overlay, zfs, vfs
|Docker CE on Debian| aufs, devicemapper, overlay2 (Debian Stretch), overlay, vfs
|Docker CE on CentOS| devicemapper, vfs
|Docker CE on Fedora| devicemapper, overlay2 (Fedora 26 或更高版本，试验), overlay (试验), vfs

如果可能，`overlay2` 是推荐的存储驱动程序。首次安装 Docker 时，默认情况下使用 `overlay2`。以前，`aufs` 默认情况下是可用的，但现在不再是这种情况。如果你想在新安装中使用 `aufs`，需要明确地配置它，并且可能需要安装额外的软件包，比如 `linux-image-extra`。

在使用 `aufs` 的现有安装中，仍然可以使用。

如果有疑问，最好的全面配置是使用带有支持 `overlay2` 存储驱动程序的内核的现代 Linux 发行版，并且对于大量的工作负载使用 Docker 卷写入，而不是将数据写入容器的可写层。

`vfs` 存储驱动程序通常不是最佳选择。在使用 `vfs` 存储驱动程序之前，确保了解它的性能、存储特性和限制。
##1.3 Docker for Mac 和 Docker for Windows
这两个版本的 Docker 仅用于开发，而不是生产环境，不支持定义存储驱动程序。
#2. 支持的 backing filesystem
对 Docker 来说，backing filesystem 就是 `/var/lib/docker/` 所在的文件系统。某些存储驱动程序仅适用于特定的 backing filesystem。

|存储驱动程序|  支持的backing filesystems|
|-|-|
|overlay, overlay2| ext4, xfs
|aufs|  ext4, xfs
|devicemapper|  direct-lvm
|btrfs| btrfs
|zfs| zfs
#3. 其他考虑情况
##3.1 是否适合工作负载
除此之外，每个存储驱动程序都有其自身的性能特征，使其更适合不同的工作负载。 概括如下：

- `aufs`、`overlay` 和 `overlay2` 的所有操作都在文件级而不是块级。这更有效地使用内存，但容器的可写层可能在写入繁重的工作负载中增长得相当大。
- 块级存储驱动程序例如 `devicemapper`、`btrfs` 和 `zfs` 在写入繁重的工作负载时表现更好（还是不如 Docker 卷好）。
- 在写入大量的小数据或有很多层的容器或很深的文件系统，`overlay` 比 `overlay2` 性能更好。
- `btrfs` 和 `zfs` 需要更多内存。
- `zfs` 是高密度工作负载（如PaaS）的理想选择。

有关性能、适用性和最佳做法的更多信息，请参阅每个存储驱动程序的文档。
##3.2 共享存储系统和存储驱动程序
如果你的企业使用 SAN，NAS，硬件 RAID 或其他共享存储系统，它们可以提供高可用性，增强的性能，精简配置，重复数据删除和压缩功能。在很多情况下，Docker 可以在这些存储系统之上工作，但 Docker 并没有与它们紧密集成。

每个 Docker 存储驱动程序都基于 Linux 文件系统或卷管理器。请务必遵循现有的最佳实践，以便在共享存储系统之上操作存储驱动程序（文件系统或卷管理器）。例如，如果在共享存储系统上使用 ZFS 存储驱动程序，请务必遵循在特定共享存储系统之上操作 ZFS 文件系统的最佳实践。
##3.3 稳定性
对于一些用户来说，稳定性比性能更重要。尽管 Docker 认为这里提到的所有存储驱动都是稳定的，但有些驱动比较新并且仍在活跃的开发中。一般来说，`aufs`，`overlay` 和 `devicemapper` 是具有最高稳定性的选择。
##3.4 经验和专业知识
选择一个你的组织很容易维护的存储驱动程序。例如，如果你使用 RHEL 或其下游分支，你可能已经有 LVM 和 Device Mapper 的使用经验。如果是这样，`devicemapper` 驱动程序可能是最好的选择。
##3.5 测试你的工作负载
在不同的存储驱动程序上运行你自己的工作负载时，可以测试 Docker 的性能。确保使用等效的硬件和工作负载来匹配生产条件，以便您可以看到哪个存储驱动程序提供了最佳的整体性能。
#4. 检查当前的存储驱动程序
每个单独存储驱动程序的详细文档详细介绍了使用给定存储驱动程序的所有设置步骤。

要查看 Docker 当前使用的存储驱动程序，请使用 `docker info` 并查找 Storage Driver 行：
```
$ docker info

Containers: 0
Images: 0
Storage Driver: overlay
 Backing Filesystem: extfs
<output truncated>
```
要更改存储驱动程序，请参阅新存储驱动程序的具体说明。某些驱动程序需要额外配置，包括配置到Docker主机上的物理或逻辑磁盘。

>重要：更改存储驱动程序时，任何现有的镜像和容器都将无法访问。这是因为他们的层不能被新的存储驱动程序使用。如果恢复更改，则可以再次访问旧镜像和容器，但是使用新驱动程序拉出或创建的任何内容都无法访问。