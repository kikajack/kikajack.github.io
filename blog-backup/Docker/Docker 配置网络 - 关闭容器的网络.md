[原文地址](https://docs.docker.com/network/none/)

如果想要完全关闭容器中的网络堆栈，可以在启动容器的时候使用 `--network none` 标志。在容器中只会创建回环（loopback）设备。下面例子演示了这一点：

###1. 创建容器：
```
$ docker run --rm -dit \
  --network none \
  --name no-net-alpine \
  alpine:latest \
  ash
```
###2. 检查容器的网络堆栈
在容器内执行常见的网络命令。注意没有创建 `eth0`。
```
$ docker exec no-net-alpine ip link show

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN qlen 1
    link/ipip 0.0.0.0 brd 0.0.0.0
3: ip6tnl0@NONE: <NOARP> mtu 1452 qdisc noop state DOWN qlen 1
    link/tunnel6 00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00 brd 00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
```
```
$ docker exec no-net-alpine ip route
```
因为没有路由表，第二个命令返回空。
###3. 停止容器
因为在创建时使用了 `--rm` 参数，容器会在停止时自动删除。
```
$ docker container rm no-net-alpine
```