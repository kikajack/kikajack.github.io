[原文地址](https://docs.docker.com/storage/storagedriver/)

了解 Docker 如何构建和存储镜像以及容器如何使用这些镜像对高效使用存储驱动程序来说很重要。可以使用这些信息在选择持久化应用程序数据的最佳方式做出明智的选择，并避免出现性能问题。

>注意：存储驱动程序允许你将数据保存在容器的可写层中。这是持久化数据的效率最低的方式。卷或绑定挂载提供更好的读写性能，卷提供比存储驱动程序或绑定挂载更高的安全性和隔离性。卷或绑定挂载都不适合本主题中描述的大多数概念。
#1. 镜像和层
Docker 镜像由一系列层组成。每个层代表镜像的 Dockerfile 中的一条指令。除了最后一个层之外，每个层都是只读了。例如下面这个 Dockerfile：
```
FROM ubuntu:15.04
COPY . /app
RUN make /app
CMD python /app/app.py
```
这个 Dockerfile 包含四条指令，每条指令创建一个层。`FROM` 语句从 `ubuntu:15.04` 镜像创建一个图层开始。`COPY` 命令添加 Docker 客户端当前目录中的一些文件。`RUN` 命令使用 `make` 命令构建您的应用程序。最后，最后一层指定在容器内运行的命令。

每个层与之前的层只有一部分差异。这些层堆叠在一起。当你创建一个新的容器时，你在底层上添加一个新的可写层。这个层通常被称为“容器层”。对正在运行的容器所做的所有更改（如写入新文件，修改现有文件和删除文件）都会写入此可写容器层。下图显示了一个基于 Ubuntu 15.04 镜像的容器。

![container-layers](http://img.blog.csdn.net/20180309204621928?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

存储驱动程序处理有关这些层相互交互方式的详细信息。有不同的存储驱动程序可以使用，在不同情况下它们各有优点和缺点。
#2. 容器和层
容器和镜像之间的主要区别是顶部的可写层。所有对容器添加新的或修改现有数据的内容都存储在该可写层中。当容器被删除时，可写层也被删除。底层镜像保持不变。

由于每个容器都有自己的可写的容器层，并且所有更改都存储在此容器层中，因此多个容器可以共享对相同基础镜像的访问权限，并且拥有自己的数据状态。下图显示了共享相同 Ubuntu 15.04 镜像的多个容器。

![sharing-layers](http://img.blog.csdn.net/20180309210023406?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

>注意：如果需要多个镜像共享访问完全相同的数据，请将此数据存储在 Docker 卷中并将其装入容器中。

Docker 使用存储驱动程序来管理镜像层和可写容器层的内容。每个存储驱动程序都处理实现的方式不同，但所有驱动程序都使用可堆叠的镜像层和写入时复制（CoW）策略。
#3. 磁盘上的容器大小
通过 `docker ps -s` 命令查看运行中的容器的准确大小。有两个不同的列和这个大小相关。

- `size`：磁盘上用于每个容器的可写层的数据大小。
- `virtual size`：用于容器使用的只读镜像数据的数据量加上容器的可写层大小。多个容器可能共享部分或全部只读镜像数据。从同一镜像开始的两个容器 100％ 共享只读数据，而如果两个容器使用的不同镜像具有公共层，则共享这些共同层。因此，你不能只总计 `virtual size`。这会高估可能磁盘总使用量。

磁盘上所有正在运行的容器使用的总磁盘空间是每个容器的 `size` 和 `virtual size` 值的组合。如果多个容器从相同的精确镜像启动，则这些容器在磁盘上的总大小将为（容器的 `size` 之和）加上一个容器的（`virtual size` - `size`）。

这不会计算容器以下面的方式占用的磁盘空间：

- 如果使用 `json-file` 日志驱动程序时用于日志文件的磁盘空间。如果容器生成大量日志数据并且未配置日志轮转，这可能并不重要。（This can be non-trivial if your container generates a large amount of logging data and log rotation is not configured）
- 用于容器的卷和绑定挂载。
- 用于容器配置文件的磁盘空间，通常比较小。
- 如果开启 swap 时，写入磁盘的内存数据。
- Checkpoint，如果使用了实验性的 checkpoint/restore 特征。
#4. 写时复制（CoW）策略
写入时复制是一种共享和复制文件以实现最高效率的策略。如果文件或目录存在于镜像的较低层中，而另一层（包括可写层）需要对其进行读取访问，则它只使用现有文件。第一次需要修改文件时（构建镜像或运行容器时），该文件将被复制到该层并进行修改。这最大限度地减少了每个后续层的 `I/O`和大小。下面将更深入地解释这些优点。
##4.1 共享有利于缩小镜像体积
当使用 `docker pull` 从仓库中下载镜像时，或者用本地还不存在的镜像创建容器时，会独立下载每个层并存储在 Docker 的本地存储区域中（通常是 Linux 主机的 `/var/lib/docker/`）。在这个例子中你可以看到这些层被下载：
```
$ docker pull ubuntu:15.04

15.04: Pulling from library/ubuntu
1ba8ac955b97: Pull complete
f157c4e5ede7: Pull complete
0b7e98f84c4c: Pull complete
a3ed95caeb02: Pull complete
Digest: sha256:5e279a9df07990286cce22e1b0f5b0490629ca6d187698746ae5e28e604a640e
Status: Downloaded newer image for ubuntu:15.04
```
每个层存储在 Docker 主机本地存储区域中它们自己的目录中。要检查文件系统上的层，列出 `/var/lib/docker/<storage-driver>/layers/` 中的内容即可。这个例子使用了 `aufs`，这是默认的存储驱动程序：
```
$ ls /var/lib/docker/aufs/layers
1d6674ff835b10f76e354806e16b950f91a191d3b471236609ab13a930275e24
5dbb0cbe0148cf447b9464a358c1587be586058d9a4c9ce079320265e2bb94e7
bef7199f2ed8e86fa4ada1309cfad3089e0542fec8894690529e4c04a7ca2d73
ebf814eccfe98f2704660ca1d844e4348db3b5ccc637eb905d4818fbfb00a06a
```
目录名和层的 ID 无关（从 Docker 1.10 开始）。

现在假设你有两个不同的 Dockerfile。使用第一个来创建名为 `acme/my-base-image:1.0` 的镜像。
```
FROM ubuntu:16.10
COPY . /app
```
第二个基于 `acme/my-base-image:1.0`，但增加了额外的层：
```
FROM acme/my-base-image:1.0
CMD /app/hello.sh
```
第二个镜像包含来自第一个镜像的所有的层，增加了一个通过 `CMD` 指令创建的新层和一个可读写的容器层。Docker 已经从第一个镜像获取所有的层，因此不需要再次下载。两个镜像会共享公共的层。

如果用这两个 Dockerfile 构建镜像，可以通过 `docker image ls` 和 `docker history` 命令来验证共享层的加密 ID 是否相同。
####1. 创建并进入 `cow-test/` 目录
####2. 在 `cow-test/` 目录中，创建新文件并填入下面内容：
```
#!/bin/sh
echo "Hello world"
```
保存文件并添加可执行权限：
```
chmod +x hello.sh
```
####3. 复制上面的第一个 Dockerfile 文件的内容到新创建的 Dockerfile.base 文件中。
####4. 复制上面的第二个 Dockerfile 文件的内容到新创建的 Dockerfile 文件中。
####5. 在 `cow-test/` 目录中，构建第一个镜像。别忘了在命令最后有一个 `.`。这会设置 PATH 指示 Docker 在哪里查找要添加到镜像中的文件。
```
$ docker build -t acme/my-base-image:1.0 -f Dockerfile.base .

Sending build context to Docker daemon  4.096kB
Step 1/2 : FROM ubuntu:16.10
 ---> 31005225a745
Step 2/2 : COPY . /app
 ---> Using cache
 ---> bd09118bcef6
Successfully built bd09118bcef6
Successfully tagged acme/my-base-image:1.0
```
###6. 构建第二个镜像。
```
$ docker build -t acme/my-final-image:1.0 -f Dockerfile .

Sending build context to Docker daemon  4.096kB
Step 1/2 : FROM acme/my-base-image:1.0
 ---> bd09118bcef6
Step 2/2 : CMD /app/hello.sh
 ---> Running in a07b694759ba
 ---> dbf995fc07ff
Removing intermediate container a07b694759ba
Successfully built dbf995fc07ff
Successfully tagged acme/my-final-image:1.0
```
####7. 检查镜像的大小：
```
$ docker image ls

REPOSITORY                         TAG                     IMAGE ID            CREATED             SIZE
acme/my-final-image                1.0                     dbf995fc07ff        58 seconds ago      103MB
acme/my-base-image                 1.0                     bd09118bcef6        3 minutes ago       103MB
```
####8. 检查构成每个镜像的层：
```
$ docker history bd09118bcef6
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
bd09118bcef6        4 minutes ago       /bin/sh -c #(nop) COPY dir:35a7eb158c1504e...   100B                
31005225a745        3 months ago        /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>           3 months ago        /bin/sh -c mkdir -p /run/systemd && echo '...   7B                  
<missing>           3 months ago        /bin/sh -c sed -i 's/^#\s*\(deb.*universe\...   2.78kB              
<missing>           3 months ago        /bin/sh -c rm -rf /var/lib/apt/lists/*          0B                  
<missing>           3 months ago        /bin/sh -c set -xe   && echo '#!/bin/sh' >...   745B                
<missing>           3 months ago        /bin/sh -c #(nop) ADD file:eef57983bd66e3a...   103MB      
```
```
$ docker history dbf995fc07ff

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
dbf995fc07ff        3 minutes ago       /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/a...   0B                  
bd09118bcef6        5 minutes ago       /bin/sh -c #(nop) COPY dir:35a7eb158c1504e...   100B                
31005225a745        3 months ago        /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>           3 months ago        /bin/sh -c mkdir -p /run/systemd && echo '...   7B                  
<missing>           3 months ago        /bin/sh -c sed -i 's/^#\s*\(deb.*universe\...   2.78kB              
<missing>           3 months ago        /bin/sh -c rm -rf /var/lib/apt/lists/*          0B                  
<missing>           3 months ago        /bin/sh -c set -xe   && echo '#!/bin/sh' >...   745B                
<missing>           3 months ago        /bin/sh -c #(nop) ADD file:eef57983bd66e3a...   103MB  
```
注意，除第二个镜像的顶层外，所有层都是相同的。所有其他层都在这两个镜像之间共享，并且只在 `/var/lib/docker/` 中存储一次。新层实际上并不占用任何空间，因为它不会更改任何文件，而只是运行一个命令。

>注意：`docker history` 的输出中的 `<missing>` 行指示那些层在其他系统上构建并且已经不可用了。可以忽略这些。
##4.2 复制使容器高效
在启动容器时，一个很小的可写容器层会添加到所有层之上。容器中的对文件系统的任何变化都会存储到这个可写容器层。任何容器不支持的文件都不会复制到这个可写层。这意味着可写层会尽可能小。

当容器中的现有文件被修改时，存储驱动程序会执行写时复制操作。实现步骤取决于具体的存储驱动程序。对于默认的 `aufs` 、`overlay` 和 `overlay2` 驱动程序，写时复制操作按照下面的顺序：

- 在所有镜像层搜索要更新的文件。该过程从最新层开始，一次处理一层，直到基础层。找到结果后，它们将被添加到缓存中以加速后面的操作。
- 对找到的文件的第一个副本执行 `copy_up` 操作，将文件复制到容器的可写层。
- 任何修改都只会作用于该文件的副本，并且该容器不能看到存在于较低层中的文件的只读副本。

Btrfs、ZFS 和其他的驱动程序以不同的方式处理写时复制。可以在后面的介绍中阅读更多信息。

写大量数据的容器比其他容器消耗更多的空间。这是因为大多数写入操作会在容器的可写顶层中占用新的空间。

>注意：对于大量写入的应用程序，不应将数据存储在容器中。相反，请使用 Docker 卷，这些卷独立于正在运行的容器，并且设计为对 I/O 有效。另外，卷可以在容器间共享，并且不会增加容器可写层的大小。

`copy_up` 操作可能会导致显着的性能开销。这个开销根据使用的存储驱动程序而不同。大文件，大量层和深层目录树会使影响更加明显。因为每个 `copy_up` 操作仅在第一次修改给定文件时发生，所以性能消耗可以缓解一部分。

为了验证写入时复制的工作方式，以下过程根据我们之前构建的 `acme/my-final-image:1.0` 镜像启动了 5 个容器，并检查它们占用的空间。

注意：下面的过程只能在 Linux 版本的 Docker 上工作。
####1. 在 Docker 主机的终端中运行 `docker run` 命令。最后的字符串是每个容器的 ID。
```
$ docker run -dit --name my_container_1 acme/my-final-image:1.0 bash \
  && docker run -dit --name my_container_2 acme/my-final-image:1.0 bash \
  && docker run -dit --name my_container_3 acme/my-final-image:1.0 bash \
  && docker run -dit --name my_container_4 acme/my-final-image:1.0 bash \
  && docker run -dit --name my_container_5 acme/my-final-image:1.0 bash

  c36785c423ec7e0422b2af7364a7ba4da6146cbba7981a0951fcc3fa0430c409
  dcad7101795e4206e637d9358a818e5c32e13b349e62b00bf05cd5a4343ea513
  1e7264576d78a3134fbaf7829bc24b1d96017cf2bc046b7cd8b08b5775c33d0c
  38fa94212a419a082e6a6b87a8e2ec4a44dd327d7069b85892a707e3fc818544
  1a174fc216cccf18ec7d4fe14e008e30130b11ede0f0f94a87982e310cf2e765
```
####2. 运行 `docker ps` 命令来验证 5 个容器都在运行
```
CONTAINER ID      IMAGE                     COMMAND     CREATED              STATUS              PORTS      NAMES
1a174fc216cc      acme/my-final-image:1.0   "bash"      About a minute ago   Up About a minute              my_container_5
38fa94212a41      acme/my-final-image:1.0   "bash"      About a minute ago   Up About a minute              my_container_4
1e7264576d78      acme/my-final-image:1.0   "bash"      About a minute ago   Up About a minute              my_container_3
dcad7101795e      acme/my-final-image:1.0   "bash"      About a minute ago   Up About a minute              my_container_2
c36785c423ec      acme/my-final-image:1.0   "bash"      About a minute ago   Up About a minute              my_container_1
```
####3. 列出本地存储区域的内容
```
$ sudo ls /var/lib/docker/containers

1a174fc216cccf18ec7d4fe14e008e30130b11ede0f0f94a87982e310cf2e765
1e7264576d78a3134fbaf7829bc24b1d96017cf2bc046b7cd8b08b5775c33d0c
38fa94212a419a082e6a6b87a8e2ec4a44dd327d7069b85892a707e3fc818544
c36785c423ec7e0422b2af7364a7ba4da6146cbba7981a0951fcc3fa0430c409
dcad7101795e4206e637d9358a818e5c32e13b349e62b00bf05cd5a4343ea513
```
####4. 检查大小
```
$ sudo du -sh /var/lib/docker/containers/*

32K  /var/lib/docker/containers/1a174fc216cccf18ec7d4fe14e008e30130b11ede0f0f94a87982e310cf2e765
32K  /var/lib/docker/containers/1e7264576d78a3134fbaf7829bc24b1d96017cf2bc046b7cd8b08b5775c33d0c
32K  /var/lib/docker/containers/38fa94212a419a082e6a6b87a8e2ec4a44dd327d7069b85892a707e3fc818544
32K  /var/lib/docker/containers/c36785c423ec7e0422b2af7364a7ba4da6146cbba7981a0951fcc3fa0430c409
32K  /var/lib/docker/containers/dcad7101795e4206e637d9358a818e5c32e13b349e62b00bf05cd5a4343ea513
```
每个容器都只占用了文件系统上的 32k 存储空间。

写时复制不仅节省空间，还缩短了启动时间。当你启动一个容器（或者来自同一个镜像的多个容器）时，Docker 只需要创建可写的容器层。

如果 Docker 在每次启动一个新容器时都必须制作底层镜像堆栈的整个副本，则容器启动时间和使用的磁盘空间将显着增加。这与虚拟机的工作方式类似，每个虚拟机具有一个或多个虚拟磁盘。
#5. 数据卷和存储驱动程序
容器删除后，任何写到容器但没有保存到数据卷的数据都会和容器一同删除。

数据卷是 Docker 主机文件系统上的一个目录或文件，直接挂载到容器中。数据卷不被存储驱动程序控制。对数据卷的读写操作绕过存储驱动程序，并以本地主机速度运行。可以将任意数量的数据卷装入容器。多个容器也可以共享一个或多个数据卷。

下图显示了一个运行两个容器的独立 Docker 主机。每个容器都存在于 Docker 主机本地存储区（`/var/lib/docker/...`）内的其自己的地址空间内。在 Docker 主机的 `/data` 上还有一个共享数据卷。这被直接安装到两个容器中。

![shared-volume](http://img.blog.csdn.net/20180309210612391?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

数据卷位于 Docker 主机本地存储区域之外，进一步增强了它们与存储驱动程序控制的独立性。当容器被删除时，存储在数据卷中的任何数据都会保留在 Docker 主机上。

关于数据卷的更多信息请参考 [Managing data in containers](https://docs.docker.com/engine/tutorials/dockervolumes/)