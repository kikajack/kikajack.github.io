[原文地址](https://docs.docker.com/storage/troubleshooting_volume_errors/)

在使用卷或绑定挂载时可能出现的问题。
#1. Error: Unable to remove filesystem
某些基于容器的工具，比如 [Google cAdvisor](https://github.com/google/cadvisor)，会将 Docker 的系统目录（通常是 `/var/lib/docker`）绑定到容器。例如，cadvisor 的文档中指示你这样启动容器：
```
$ sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest
```
当你绑定挂载 `/var/lib/docker/` 时，这会高效地将所有其他正在运行的容器的所有资源作为文件系统装入挂载 `/var/lib/docker/` 的容器中。当尝试删除任何这些容器时，删除尝试可能会失败，并显示如下错误：
```
Error: Unable to remove filesystem for
74bef250361c7817bee19349c93139621b272bc8f654ae112dd4eb9652af9515:
remove /var/lib/docker/containers/74bef250361c7817bee19349c93139621b272bc8f654ae112dd4eb9652af9515/shm:
Device or resource busy
```
如果绑定挂载 `/var/lib/docker/` 目录的容器在对这个目录中的文件系统处理时使用了 `statfs` 或 `fstatfs` 并且没有关闭它们时，就会发生这个问题。

通常，建议禁止使用这种方式绑定挂载 ` /var/lib/docker`。但是，cAdvisor 需要这种绑定挂载来实现核心功能。

如果不确定哪个进程导致错误中提到的路径繁忙并阻止其被删除，则可以使用 `lsof` 命令查找其进程。例如，对于上面的错误：
```
$ sudo lsof /var/lib/docker/containers/74bef250361c7817bee19349c93139621b272bc8f654ae112dd4eb9652af9515/shm
```
要解决此问题，请停止绑定挂载 `/var/lib/docker` 的容器，然后再次尝试删除其他容器。