[原文地址](https://docs.docker.com/config/daemon/ipv6/)

需要在 Docker 守护进程中开启对 IPv6 的支持才可以在 Docker 容器或 swarm 服务中使用 IPv6。之后，可以选择将 IPv4 或 IPv6（或两者）与任何容器，服务或网络一起使用。

>注意：只有在 Linux 主机上运行的 Docker 守护程序才支持 IPv6 网络。

1. 编辑 `/etc/docker/daemon.json` 并将关键字 `ipv6` 设为 `true`。
```
{
  "ipv6": true
}
Save the file.
```
2. 重新加载 Docker 配置文件。
```
$ systemctl reload docker
```
现在可以通过 `--ipv6` 标志来创建网络，并通过 `--ip6` 标志为容器分配 IPv6 地址。