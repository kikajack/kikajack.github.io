[原文地址](https://docs.docker.com/network/iptables/)

在 Linux 上，Docker 操纵 iptables 规则来提供网络隔离。这是一个实现细节，你不应修改 Docker 插入到你的 iptables 策略中的规则。
#1. 在 Docker 的规则之前添加 iptables 策略
所有 Docker 的 iptables 规则都被添加到 `DOCKER` 链中。不要手动操作此表。如果你需要添加在 Docker 规则之前加载的规则，请将它们添加到 `DOCKER-USER` 链中。 这些规则在 Docker 自动创建任何规则之前加载。
##1.1 限制到 Docker 守护进程的连接
默认情况下，所有外部源 IP 都被允许连接到 Docker 守护进程。要仅允许特定的 IP 或网络访问容器，请在 DOCKER 过滤器链的顶部插入否定规则。例如，以下规则只允许外部 IP 为 `192.168.1.1` 的主机访问：
```
$ iptables -I DOCKER-USER -i ext_if ! -s 192.168.1.1 -j DROP
```
你可以改为允许来自源子网的连接。以下规则仅允许从子网 `192.168.1.0/24` 进行访问：
```
$ iptables -I DOCKER-USER -i ext_if ! -s 192.168.1.0/24 -j DROP
```
最后，可以通过 `--src-range` 指定 IP 地址范围（注意，在使用 `--src-range` 或 `--dst-range` 时需要添加 `-m`）：
```
$ iptables -I DOCKER-USER -m iprange -i ext_if ! --src-range 192.168.1.1-192.168.1.3 -j DROP
```
可以将 `-s` 或 `--src-range` 与 `-d` 或 `--dst-range` 组合使用来控制源和目的 IP 地址。例如，如果 Docker 守护进程监听 `192.168.1.99` 和 `10.1.2.3`，可以制定特定于 `10.1.2.3` 的规则并保持 `192.168.1.99` 开启。

`iptables` 是复杂的，更复杂的规则超出了这个话题的范围。请参阅 [Netfilter.org HOWTO](https://www.netfilter.org/documentation/HOWTO/NAT-HOWTO.html) 了解更多信息。
#2. 防止 Docker 操纵 iptables
为防止 Docker 操纵 iptables 策略，在 `/etc/docker/daemon.json` 中将 iptables 关键字设置为 false。对大多数用户来说这是不合适的，因为此时 iptables 策略需要用户手工管理。