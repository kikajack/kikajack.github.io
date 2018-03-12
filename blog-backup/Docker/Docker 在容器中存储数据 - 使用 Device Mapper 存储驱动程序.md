[原文地址](https://docs.docker.com/storage/storagedriver/device-mapper-driver/)

Device Mapper 是基于内核的框架，支持 Linux 上的许多高级卷管理技术。 Docker 的 `devicemapper` 存储驱动程序利用此框架的精简配置和快照功能进行镜像和容器管理。本文将 Device Mapper 存储驱动程序称为 `devicemapper`，将内核框架称为 `Device Mapper`。

在支持这个特性的系统中，对 `devicemapper` 的支持包含在 Linux 内核中。然而，要想在 Docker 中使用还需要特殊配置。例如，在 RHEL 或 CentOS 的原始安装的 Docker 默认使用 `overlay`，但无法支持这个配置。

`devicemapper` 驱动程序使用专用于 Docker 的块设备，并在块级而不是文件级运行。可以通过将物理存储添加到 Docker 主机来扩展这些设备，并且它们比在操作系统级别使用文件系统要更好。
#1. 先决条件
- `devicemapper` 存储驱动程序是在 RHEL、CentOS 和 Oracle Linux 上运行的 Docker EE 和 Commercially Supported Docker Engine (CS-Engine) 唯一支持的存储驱动程序。
- `devicemapper` 也可以用于在 CentOS、Fedora、Ubuntu 和 Debian 上运行的 Docker CE。
- 改变存储驱动程序会使在本地创建的容器无法访问。通过 `docker save` 来保存容器，并将已有镜像上传到 Docker Hub 或私有仓库，以便后面需要时再次创建。
#2. 确认 Docker 是否支持 `devicemapper` 存储驱动程序
下面所有步骤都需要满足上面的先决条件。
##2.1 配置 `loop-lvm` 模式用于测试环境
这个配置只适合测试环境。回环设备速度慢且资源密集，并且需要你以特定大小在磁盘上创建文件。他们还可以引入竞争条件。建议用于测试环境是因为安装容易。

对于生产环境，参考下一小节。
####1. 停止 Docker
```
$ sudo systemctl stop docker
```
####2. 编辑 `/etc/docker/daemon.json`。如果不存在就先创建。添加下面的内容。
```
{
  "storage-driver": "devicemapper"
}
```
查看每个存储驱动程序的所有选项：

- [Stable](https://docs.docker.com/engine/reference/commandline/dockerd/#storage-driver-options)
- [Edge](https://docs.docker.com/edge/engine/reference/commandline/dockerd/#storage-driver-options)

如果 `daemon.json` 文件中的 JSON 格式有误，则 Docker 无法启动。
####3. 启动 Docker
```
$ sudo systemctl start docker
```
####4. 验证守护进程使用了 `devicemapper` 存储驱动程序。使用 `docker info` 命令并查看 Storage Driver 部分
```
$ docker info

  Containers: 0
    Running: 0
    Paused: 0
    Stopped: 0
  Images: 0
  Server Version: 17.03.1-ce
  Storage Driver: devicemapper
  Pool Name: docker-202:1-8413957-pool
  Pool Blocksize: 65.54 kB
  Base Device Size: 10.74 GB
  Backing Filesystem: xfs
  Data file: /dev/loop0
  Metadata file: /dev/loop1
  Data Space Used: 11.8 MB
  Data Space Total: 107.4 GB
  Data Space Available: 7.44 GB
  Metadata Space Used: 581.6 kB
  Metadata Space Total: 2.147 GB
  Metadata Space Available: 2.147 GB
  Thin Pool Minimum Free Space: 10.74 GB
  Udev Sync Supported: true
  Deferred Removal Enabled: false
  Deferred Deletion Enabled: false
  Deferred Deleted Device Count: 0
  Data loop file: /var/lib/docker/devicemapper/data
  Metadata loop file: /var/lib/docker/devicemapper/metadata
  Library Version: 1.02.135-RHEL7 (2016-11-16)
<output truncated>
```
这个主机运行在不建议用于生产环境的 `loop-lvm` 模式。这表现为 `Data loop file` 和 `Metadata loop file` 位于 `/var/lib/docker/devicemapper` 下的文件中。这些是回环加载的稀疏文件（loopback-mounted sparse files）。对于生产系统，请参阅下一小节的配置 `direct-lvm` 模式。
##2.2 配置 `direct-lvm` 模式用于生产环境
使用 `devicemapper`  存储驱动程序的生产环境中的主机必须使用 `direct-lvm` 模式。该模式使用块设备来创建精简池。这比使用回环设备更快，更有效地使用系统资源，并且块设备可以按需增长。然而，比 `loop-lvm` 模式需要更多的设置。

满足前面的先决条件后，按照下面的步骤来配置 Docker 使用 `direct-lvm` 模式下的 `devicemapper`  存储驱动程序。

>警告：改变存储驱动程序会使在本地创建的容器无法访问。通过 `docker save` 来保存容器，并将已有镜像上传到 Docker Hub 或私有仓库，以便后面需要时再次创建。
###允许 DOCKER 配置 DIRECT-LVM 模式
在 Docker 17.06 或更高版本，Docker 可以替你管理块设备，简化 `direct-lvm` 模式的配置。这仅适用于 Docker 的首次设置。只能使用一个块设备。如果需要使用多个块设备，请手动配置 `direct-lvm` 模式。添加了以下新的配置选项：

|选项|  描述| 是否必须？|  默认值|  示例|
|-|-|-|-|-|
|`dm.directlvm_device`| 用于 `direct-lvm`配置的块设备的路径| Yes|    |`dm.directlvm_device="/dev/xvdf"`|
|`dm.thinp_percent`|  从传入的块设备中用于存储的空间百分比。|  No| 95  |`dm.thinp_percent=95`|
|`dm.thinp_metapercent`|  从传入的块设备中用于元数据存储的空间百分比。| No| 1 |`dm.thinp_metapercent=1`|
|`dm.thinp_autoextend_threshold`| lvm 应该自动扩展精简池的阈值，总存储空间百分比的形式。|  No| 80  |`dm.thinp_autoextend_threshold=80`|
|`dm.thinp_autoextend_percent`| 每次触发自动扩展时增加精简池的大小，总存储空间百分比的形式。  |No |20 |`dm.thinp_autoextend_percent=20`|
|`dm.directlvm_device_force`| 即使文件系统已经存在，是否格式化块设备。如果设置为 false 并且存在文件系统，则会记录错误并保持文件系统不变。|No| false |`dm.directlvm_device_force=true`|

编辑 `daemon.json` 文件并设置合适的选项，然后重启 Docker是设置生效。下面的 `daemon.json` 例子设置了上表中的所有选项：
```
{
  "storage-driver": "devicemapper",
  "storage-opts": [
    "dm.directlvm_device=/dev/xdf",
    "dm.thinp_percent=95",
    "dm.thinp_metapercent=1",
    "dm.thinp_autoextend_threshold=80",
    "dm.thinp_autoextend_percent=20",
    "dm.directlvm_device_force=false"
  ]
}
```
查看每个存储驱动程序的所有选项：

- [Stable](https://docs.docker.com/engine/reference/commandline/dockerd/#storage-driver-options)
- [Edge](https://docs.docker.com/edge/engine/reference/commandline/dockerd/#storage-driver-options)

Docker 自动调用命令来配置块设备。

>警告：在 Docker 准备好块设备后无法改变这些值，否则会触发错误。

仍然需要执行定期维护任务。参考下面的管理 `devicemapper`。
###手动配置 DIRECT-LVM 模式
下面的步骤创建了一个配置为精简池的逻辑卷，以用作存储池的备份。它假定你在 `/dev/xvdf` 有一个备用块设备，并具有足够的可用空间来完成任务。设备标识符和卷大小在你的环境中可能不同，你应该在整个过程中替换自己的值。该过程还假定 Docker 守护程序处于停止状态。
####1. 确定要使用的块设备。该设备位于 `/dev/` 下（如 `/dev/xvdf`），并且需要足够的可用空间来存储主机运行的工作负载的镜像和容器层。固态硬盘是最理想的。
####2. 停止 Docker
```
$ sudo systemctl stop docker
```
####3. 安装下面的包
- RHEL / CentOS：`device-mapper-persistent-data`、`lvm2` 和所有依赖
- Ubuntu / Debian：`thin-provisioning-tools`、`lvm2` 和所有依赖
####4. 通过 `pvcreate` 命令在步骤 1 确定的块设备上创建物理卷。将 `/dev/xvdf` 替换为你的设备名
>警告：接下来的几个步骤是破坏性的，所以请确保你指定了正确的设备！！
```
$ sudo pvcreate /dev/xvdf

Physical volume "/dev/xvdf" successfully created.
```
####5. 通过 `vgcreate` 命令在同一个设备上创建 `docker` volume group
```
$ sudo vgcreate docker /dev/xvdf

Volume group "docker" successfully created
```
####6. 通过 `lvcreate` 命令创建两个名为 `thinpool` 和 `thinpoolmeta` 的逻辑卷。最后一个参数指定可用空间的大小，以便在空间不足时自动扩展数据或元数据。这些是推荐值。
```
$ sudo lvcreate --wipesignatures y -n thinpool docker -l 95%VG

Logical volume "thinpool" created.

$ sudo lvcreate --wipesignatures y -n thinpoolmeta docker -l 1%VG

Logical volume "thinpoolmeta" created.
```
####7. 通过 `lvconvert` 命令将卷转换为精简池和精简池中的元数据存储位置。
```
$ sudo lvconvert -y \
--zero n \
-c 512K \
--thinpool docker/thinpool \
--poolmetadata docker/thinpoolmeta

WARNING: Converting logical volume docker/thinpool and docker/thinpoolmeta to
thin pool's data and metadata volumes with metadata wiping.
THIS WILL DESTROY CONTENT OF LOGICAL VOLUME (filesystem etc.)
Converted docker/thinpool to thin pool.
```
####8. 通过 lvm 配置文件配置精简池的自动扩展。
```
$ sudo vi /etc/lvm/profile/docker-thinpool.profile
```
####9. 指定 `thin_pool_autoextend_threshold` 和 `thin_pool_autoextend_percent` 的值
`thin_pool_autoextend_threshold` 是触发 lvm 尝试自动扩展可用空间的空间使用率百分比（100 = 禁用，不推荐）。 

`thin_pool_autoextend_percent` 是触发自动扩展时会扩展的大小（0 = 禁用）。

下面的例子在磁盘使用率达到 80% 时自动增加 20% 的容量：
```
activation {
  thin_pool_autoextend_threshold=80
  thin_pool_autoextend_percent=20
}
```
保存文件。
####10. 使用 `lvchange` 命令应用 LVM 配置文件
```
$ sudo lvchange --metadataprofile docker-thinpool docker/thinpool

Logical volume docker/thinpool changed.
```
####11. 启用对主机上逻辑卷的监视
没有这一步，即使存在 LVM 配置文件，也不会自动扩展。
```
$ sudo lvs -o+seg_monitor

LV       VG     Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert Monitor
thinpool docker twi-a-t--- 95.00g             0.00   0.01                             monitored
```
####12. 如果之前在这个主机上运行过 Docker，或 `/var/lib/docker/` 文件已经存在，移除文件以便让 Docker 使用新的 LVM pool 来存储镜像和容器
```
$ mkdir /var/lib/docker.bk
$ mv /var/lib/docker/* /var/lib/docker.bk
```
如果以下任何步骤失败并且需要恢复，则可以删除 `/var/lib/docker` 并将其替换为 `/var/lib/docker.bk`。
####13. 编辑 `/etc/docker/daemon.json` 并配置 `devicemapper`存储驱动程序所需的选择。增加下面的内容：
```
{
    "storage-driver": "devicemapper",
    "storage-opts": [
    "dm.thinpooldev=/dev/mapper/docker-thinpool",
    "dm.use_deferred_removal=true",
    "dm.use_deferred_deletion=true"
    ]
}
```
####14. 启动 Docker
- systemd:
```
$ sudo systemctl start docker
```
- service:
```
$ sudo service docker start
```
####15. 通过 `docker info` 命令验证 Docker 使用了新的配置
```
$ docker info

Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 17.03.1-ce
Storage Driver: devicemapper
 Pool Name: docker-thinpool
 Pool Blocksize: 524.3 kB
 Base Device Size: 10.74 GB
 Backing Filesystem: xfs
 Data file:
 Metadata file:
 Data Space Used: 19.92 MB
 Data Space Total: 102 GB
 Data Space Available: 102 GB
 Metadata Space Used: 147.5 kB
 Metadata Space Total: 1.07 GB
 Metadata Space Available: 1.069 GB
 Thin Pool Minimum Free Space: 10.2 GB
 Udev Sync Supported: true
 Deferred Removal Enabled: true
 Deferred Deletion Enabled: true
 Deferred Deleted Device Count: 0
 Library Version: 1.02.135-RHEL7 (2016-11-16)
<output truncated>
```
如果 Docker 配置正确，则 `Data file` 和 `Metadata file` 是空的，并且 `Pool Name` 是 `docker-thinpool`。

####16. 验证配置无误后，可以删除包含之前配置的 `/var/lib/docker.bk` 目录。
```
$ rm -rf /var/lib/docker.bk
```
#3. 管理 devicemapper
##3.1 监控精简池（thin pool）
不要单独依靠 LVM 自动扩展。因为虽然卷组会自动扩展，但卷仍可能被填满。可以使用 `lvs`  `或lvs -a` 监视卷上的可用空间。最好在操作系统级别使用监控工具，例如 Nagios。

通过 `journalctl` 查看 LVM 的日志：
```
$ journalctl -fu dm-event.service
```
如果遇到精简池的重复问题，可以将存储选项 `dm.min_free_space` 设置为 `/etc/docker.daemon.json` 中的值（表示百分比）。例如，将其设置为 10 可确保当可用空间达到或接近 10％ 时操作失败并发出警告。请参阅 [Engine守护程序参考中的存储驱动程序选项](https://docs.docker.com/engine/reference/commandline/dockerd/#storage-driver-options)。
##3.2 增加正在运行的设备的容量
可以增加正在运行的精简池设备上的池的容量。这对于数据的逻辑卷已满并且卷组处于满负荷状态时很有用。具体过程取决于使用的是 `loop-lvm` 精简池还是 `direct-lvm` 精简池。
###1. 调整 LOOP-LVM 精简池
最简单的调整 `loop-lvm` 精简池的方法是使用 `device_tool` 工具，当然你也可以使用操作系统的工具。
####使用 device_tool 工具
Docker Github 仓库的 `contrib/` 目录中提供了一个名为 `device_tool.go` 的社区维护脚本。可以使用此工具调整 `loop-lvm` 精简池的大小，避免冗长的过程。这个工具不能保证能正常工作，但你只能在非生产系统上使用 `loop-lvm`。

如果不想使用 `device_tool` 工具，可以手动调整精简池。

1. 从 Github 仓库下载项目后，切换到 `contrib/docker-device-tool` 目录，按照 README.md 文件中的指示编译这个工具。
2. 使用工具。下面例子中将精简池调整为 200 GB。
```
$ ./device_tool resize 200GB
```
####使用操作系统工具
如果不想使用 `device_tool` 工具，可以按照下面的步骤手动调整 `loop-lvm` 精简池。

在 `loop-lvm` 模式下，使用一个回环设备来存储数据，另一个回环设备来存储元数据。`loop-lvm` 模式仅支持用于测试，因为它具有显着的性能和稳定性缺点。

如果你正在使用 `loop-lvm` 模式，`docker info` 命令会显示 `Data loop file` 和 `Metadata loop file` 文件的路径：
```
$ docker info |grep 'loop file'

 Data loop file: /var/lib/docker/devicemapper/data
 Metadata loop file: /var/lib/docker/devicemapper/metadata
```
按照下面的步骤来增加精简池的大小。在这个例子中，精简池由 100GB 调整为 200GB。
####1. 列出设备大小
```
$ sudo ls -lh /var/lib/docker/devicemapper/

total 1175492
-rw------- 1 root root 100G Mar 30 05:22 data
-rw------- 1 root root 2.0G Mar 31 11:17 metadata
```
####2. 通过 `truncate` 命令将 `data` 文件调整为 200GB
`truncate` 命令用于增大或减小指定文件的体积。注意，减小体积是一种破坏性操作。
```
$ sudo truncate -s 200G /var/lib/docker/devicemapper/data
```
####3. 验证文件体积是否改变
```
$ sudo ls -lh /var/lib/docker/devicemapper/

total 1.2G
-rw------- 1 root root 200G Apr 14 08:47 data
-rw------- 1 root root 2.0G Apr 19 13:27 metadata
```
####4. 磁盘上的回环文件已经变了，但内存中的还没有。以 GB 为单位列出内存中的回环设备。重新加载后再次列出，会发现重新加载后变化生效。
```
$ echo $[ $(sudo blockdev --getsize64 /dev/loop0) / 1024 / 1024 / 1024 ]

100

$ sudo losetup -c /dev/loop0

$ echo $[ $(sudo blockdev --getsize64 /dev/loop0) / 1024 / 1024 / 1024 ]

200
```
####5. 重新加载 devicemapper 精简池
a. 首先获取精简池名称。名字是由`：`分隔的第一个字段。这个命令提取：
```
    $ sudo dmsetup status | grep ' thin-pool ' | awk -F ': ' {'print $1'}

    docker-8:1-123141-pool
```
b. 转储精简池的设备映射表。
```
    $ sudo dmsetup table docker-8:1-123141-pool

    0 209715200 thin-pool 7:1 7:0 128 32768 1 skip_block_zeroing
```
c. 使用输出的第二个字段计算精简池的总扇区。该数字以 512-k 扇区表示。一个 100G 文件有 209715200 个 512-k 扇区。如果你把这个数字加倍到200G，会得到 419430400 个 512-k 扇区。
d. 使用以下三个 `dmsetup` 命令重新加载具有新扇区号的精简池。
```
    $ sudo dmsetup suspend docker-8:1-123141-pool

    $ sudo dmsetup reload docker-8:1-123141-pool --table '0 419430400 thin-pool 7:1 7:0 128 32768 1 skip_block_zeroing'

    $ sudo dmsetup resume docker-8:1-123141-pool
```
###2. 调整 DIRECT-LVM 精简池
要调整 `direct-lvm` 精简池，需要先将新的块设备连接到 Docker 主机，并留言内核给它指定的名称。在这个例子中，新的块设备是 `/dev/xvdg`。

按照下面的步骤来调整 `direct-lvm` 精简池，将块设备和其他参数改为适合你的情况。
####1. 收集你的卷组（volume group）信息
通过 `pvdisplay` 命令找出当前正被你的精简池使用的物理块设备，已经卷组的名称。
```
$ sudo pvdisplay |grep 'VG Name'

PV Name               /dev/xvdf
VG Name               docker
```
将后面的步骤中的块设备和卷组名称改为你的值。
####2. 通过 `vgextend` 命令使用卷组名和块设备名来扩展卷组。
```
$ sudo vgextend docker /dev/xvdg

Physical volume "/dev/xvdg" successfully created.
Volume group "docker" successfully extended
```
####3. 扩展 `docker/thinpool` 逻辑卷。该命令立即使用 100％ 的卷，而不会自动扩展。如果要扩展元数据精简池，使用 `docker/thinpool_tmeta`。
```
$ sudo lvextend -l+100%FREE -n docker/thinpool

Size of logical volume docker/thinpool_tdata changed from 95.00 GiB (24319 extents) to 198.00 GiB (50688 extents).
Logical volume docker/thinpool_tdata successfully resized.
```
####4. 在 `docker info` 的输出中查看 `Data Space Available` 字段验证精简池新尺寸。对于 `docker/thinpool_tmeta` 逻辑卷，查看 `Metadata Space Available` 字段。
```
Storage Driver: devicemapper
 Pool Name: docker-thinpool
 Pool Blocksize: 524.3 kB
 Base Device Size: 10.74 GB
 Backing Filesystem: xfs
 Data file:
 Metadata file:
 Data Space Used: 212.3 MB
 Data Space Total: 212.6 GB
 Data Space Available: 212.4 GB
 Metadata Space Used: 286.7 kB
 Metadata Space Total: 1.07 GB
 Metadata Space Available: 1.069 GB
<output truncated>
```
##3.3 重启主机，使 `devicemapper` 生效
如果重启主机时发现 docker 服务无法启动，并报错“Non existing device”。需要用下面的命令重新激活逻辑卷：
```
sudo lvchange -ay docker/thinpool
```
#4. `devicemapper` 存储驱动程序如何工作
>警告：不要直接操作 `/var/lib/docker/` 中的任何文件或目录。这些文件和目录由 Docker 管理。

通过 `lsblk` 命令从操作系统角度查看设备和对应的池。
```
$ sudo lsblk

NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
xvda                    202:0    0    8G  0 disk
└─xvda1                 202:1    0    8G  0 part /
xvdf                    202:80   0  100G  0 disk
├─docker-thinpool_tmeta 253:0    0 1020M  0 lvm
│ └─docker-thinpool     253:2    0   95G  0 lvm
└─docker-thinpool_tdata 253:1    0   95G  0 lvm
  └─docker-thinpool     253:2    0   95G  0 lvm
```
通过 `mount` 命令来查看 Docker 使用的挂载点：
```
$ mount |grep devicemapper
/dev/xvda1 on /var/lib/docker/devicemapper type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
```
在使用 `devicemapper` 时，Docker 将镜像和层中的内容存储到精简池，并通过将其挂载到 `/var/lib/docker/devicemapper/` 的子目录暴露给容器。
##4.1 磁盘上的镜像和容器层
`/var/lib/docker/devicemapper/metadata/` 目录包含 Devicemapper 自身配置和已经存在的容器层的元数据。`devicemapper` 存储驱动程序使用快照（snapshot），而这个元数据包含这些快照的信息。这些文件是 JSON 格式。

`/var/lib/devicemapper/mnt/` 目录包含每个镜像和已有容器的挂载点。镜像层挂载点是空的，但容器的挂载点显示容器的文件系统，因为它显示在容器内。（a container’s mount point shows the container’s filesystem as it appears from within the container.）
##4.2 镜像层和共享
`devicemapper` 存储驱动程序使用专用块设备而不是格式化的文件系统，并在块级别上对文件进行操作，以便在写时复制（CoW）操作期间实现最高性能。
####快照
`devicemapper` 的另一个特性是使用快照（也会叫做精简设备或虚拟设备），快照将每层中引入的差异存储为非常小的轻量级精简池。快照提供许多好处：

- 容器中共享的层之会在磁盘上存储一次，除非是可写层。例如，如果有 10 个基于 `alpine` 的镜像，`alpine` 镜像及其父镜像之会在磁盘上存储一次。
- 快照是写时复制（CoW）策略的实现。这意味着给定的文件或目录只有在容器被修改或删除时才会被复制到容器的可写层。
- 因为 `devicemapper` 在块级别上对文件进行操作，一个可写层的多个块可以同时修改。
- 快照可以通过标准的操作系统级备份工具进行备份。只要复制 `/var/lib/docker/devicemapper/` 即可。
####DEVICEMAPPER 工作流程
通过 `devicemapper` 存储驱动程序启动 Docker 时，所有和镜像及容器层相关的对象都保存在 `/var/lib/docker/devicemapper/`，它由一个或多个块级设备（环回设备（仅用于测试）或物理磁盘）支持。

- _base device_ 是最低级别的对象。这就是精简池本身。可以使用 `docker info` 来检查。它包含一个文件系统。 这个 _base device_ 是每个镜像和容器层的起点。_base device_ 是 Device Mapper 的实现细节，而不是 Docker 层。
- _base device_ 的元数据和所有的镜像及容器层都以 JSON 格式存储在 `/var/lib/docker/devicemapper/metadata/` 中。这些层是写时复制的快照，意味着它们都是空的，直到和它们的依赖层产生差异。
- 每个容器的可写层都挂载到 `/var/lib/docker/devicemapper/mnt/` 中的一个挂载点。对每个只读镜像层和每个停止状态的容器，对应有一个空目录。

每个镜像层都是下面层的快照。每个镜像的最底层是池中的 _base device_  的快照。在运行容器时，容器是它所依赖的镜像的快照。下面例子中的 Docker 主机上有两个容器在运行，分别是 `ubuntu` 和 `busybox`。

![two_dm_container](http://img.blog.csdn.net/20180311095715278?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
#5. 容器的读写如何作用于 `devicemapper`
##5.1 读文件
通过 `devicemapper`，读操作发生在块级别。下图显示了在示例容器中对单独块（0x44f）的读操作的高层级的处理。

![dm_container](http://img.blog.csdn.net/20180311095756210?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

应用程序向容器中的块 0x44f 发出读取请求。由于容器是镜像的精简快照，它没有该块，但它具有一个指向最接近的包含这个块的父镜像的指针，并从那里读取块。该块现在存在于容器的内存中。
##5.2 写文件
**写新文件**：通过 `devicemapper` 驱动程序，通过 _allocate-on-demand_ 操作实现将新数据写到容器中。新文件的每个块都分配到容器的可写层，并且块写入那里。（Each block of the new file is allocated in the container’s writable layer and the block is written there.）

**改文件**：文件的相关块从最近的包含这些块的层中读取。当容器写入文件时，只有修改后的块被写入容器的可写层。

**删除文件或目录**：当删除容器可写层中的文件或目录时，或者镜像层删除其父层中存在的文件时，`devicemapper` 存储驱动程序会截获对该文件或目录的进一步读取尝试，并回应文件或目录不存在。不会真的删除，但在读的时候告诉你不存在。

**写然后删除文件**：如果容器写入文件并稍后删除文件，则所有这些操作都发生在容器的可写层中。在这种情况下，如果使用 `direct-lvm`，块将被释放。如果使用`loop-lvm`，块可能不会被释放。这是不在生产环境中使用 `loop-lvm` 的另一个原因。
#6. Device Mapper 和 Docker 性能
- 按需分配（allocate-on demand）性能影响：
`devicemapper` 存储驱动程序使用按需分配操作将精简池中的新块分配到容器的可写层。每个块都是 64KB，所以这是用于写入的最小大小。

- 写时复制（Copy-on-write）性能影响：
容器第一次修改特定块时，该块将写入容器的可写层。因为这些写操作发生在块的级别而不是文件，所以性能影响最小化。但是，编写大量数据块仍然会对性能产生负面影响，并且设备映射程序存储驱动程序实际上可能在此方案中的性能比其他存储驱动程序差。对于写入繁重的工作负载，应使用完全绕过存储驱动程序的数据卷。
##6.1 性能最佳实践
请记住这些事情，以便在使用 `devicemapper` 存储驱动程序时最大限度地提高性能。

- 使用 `direct-lvm`：生产环境中不要使用 `loop-lvm`。

- 使用高速存储：固态存储设备（SSD）读写性能更好。

- 内存使用情况：`devicemapper` 比一些其他存储驱动程序使用更多的内存。每个启动的容器都会将其文件的一个或多个副本加载到内存中，具体取决于同时修改同一文件的块数。由于内存压力，`devicemapper` 存储驱动程序可能不是高密度用例中某些工作负载的正确选择。

- 对于频繁写入的工作负载使用卷：卷为写入繁重的工作负载提供了最佳和最可预测的性能。这是因为它们绕过了存储驱动程序，并且不会产生精简配置和写入时复制引入的任何潜在开销。卷还有其他好处，例如允许在容器之间共享数据，并且即使在没有正在运行的容器正在使用它们时也会持续存在。