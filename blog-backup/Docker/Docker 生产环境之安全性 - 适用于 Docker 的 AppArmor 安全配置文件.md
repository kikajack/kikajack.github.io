[原文地址](http://111.230.25.113:4000/engine/security/apparmor/)

AppArmor（Application Armor，应用程序防护）是一个 Linux 安全模块，可保护操作系统及其应用程序免受安全威胁。要使用它，系统管理员会将 AppArmor 安全配置文件与每个程序相关联。Docker 希望找到加载并执行的 AppArmor 策略。

Docker 自动为容器生成并加载名为 `docker-default` 的默认配置文件。在 Docker 1.13.0 和更高版本中，Docker 二进制文件在 `tmpfs` 中生成该配置文件，然后将其加载到内核中。在早于 1.13.0 的 Docker 版本上，此配置文件将在 `/etc/apparmor.d/docker` 中生成。

>注意：这个配置文件用于容器而不是 Docker 守护进程。

Docker Engine 守护程序的配置文件存在，但目前没有与 `deb` 软件包一起安装。如果你对守护进程配置文件的源代码感兴趣，它位于 Docker Engine 源代码库的 `contrib/apparmor` 中。
# 1. 理解策略
`docker-default` 配置文件是运行容器的默认配置文件。它具有适度的保护性，同时提供广泛的应用兼容性。该配置文件是从以下模板生成的。

运行容器时会使用 `docker-default` 策略，除非通过 `security-opt` 选项覆盖。例如，例如，以下内容明确指定了默认策略：
```
$ docker run --rm -it --security-opt apparmor=docker-default hello-world
```
# 2. 加载和卸载配置文件
将新配置文件加载到 AppArmor 以用于容器：
```
$ apparmor_parser -r -W /path/to/your_profile
```
然后，使用 `--security-opt` 运行自定义配置文件，如下所示：
```
$ docker run --rm -it --security-opt apparmor=your_profile hello-world
```
从 AppArmor 卸载配置文件：
```
# stop apparmor
$ /etc/init.d/apparmor stop
# unload the profile
$ apparmor_parser -R /path/to/profile
# start apparmor
$ /etc/init.d/apparmor start
```
## 2.1 编写配置文件的资源
AppArmor 中文件通配的语法与其他一些通配实现有点不同。强烈建议查看关于 AppArmor 配置文件语法的以下资源。

- [Quick Profile Language](http://wiki.apparmor.net/index.php/QuickProfileLanguage)
- [Globbing Syntax](http://wiki.apparmor.net/index.php/AppArmor_Core_Policy_Reference#AppArmor_globbing_syntax)
# 3. Nginx 示例配置文件
在本例中，为 Nginx 创建了一个自定义 AppArmor 配置文件：
```
#include <tunables/global>


profile docker-nginx flags=(attach_disconnected,mediate_deleted) {
  #include <abstractions/base>

  network inet tcp,
  network inet udp,
  network inet icmp,

  deny network raw,

  deny network packet,

  file,
  umount,

  deny /bin/** wl,
  deny /boot/** wl,
  deny /dev/** wl,
  deny /etc/** wl,
  deny /home/** wl,
  deny /lib/** wl,
  deny /lib64/** wl,
  deny /media/** wl,
  deny /mnt/** wl,
  deny /opt/** wl,
  deny /proc/** wl,
  deny /root/** wl,
  deny /sbin/** wl,
  deny /srv/** wl,
  deny /tmp/** wl,
  deny /sys/** wl,
  deny /usr/** wl,

  audit /** w,

  /var/run/nginx.pid w,

  /usr/sbin/nginx ix,

  deny /bin/dash mrwklx,
  deny /bin/sh mrwklx,
  deny /usr/bin/top mrwklx,


  capability chown,
  capability dac_override,
  capability setuid,
  capability setgid,
  capability net_bind_service,

  deny @{PROC}/* w,   # deny write for all files directly in /proc (not in a subdir)
  # deny write to files not in /proc/<number>/** or /proc/sys/**
  deny @{PROC}/{[^1-9],[^1-9][^0-9],[^1-9s][^0-9y][^0-9s],[^1-9][^0-9][^0-9][^0-9]*}/** w,
  deny @{PROC}/sys/[^k]** w,  # deny /proc/sys except /proc/sys/k* (effectively /proc/sys/kernel)
  deny @{PROC}/sys/kernel/{?,??,[^s][^h][^m]**} w,  # deny everything except shm* in /proc/sys/kernel/
  deny @{PROC}/sysrq-trigger rwklx,
  deny @{PROC}/mem rwklx,
  deny @{PROC}/kmem rwklx,
  deny @{PROC}/kcore rwklx,

  deny mount,

  deny /sys/[^f]*/** wklx,
  deny /sys/f[^s]*/** wklx,
  deny /sys/fs/[^c]*/** wklx,
  deny /sys/fs/c[^g]*/** wklx,
  deny /sys/fs/cg[^r]*/** wklx,
  deny /sys/firmware/** rwklx,
  deny /sys/kernel/security/** rwklx,
}
```
#### 1 保存自定义配置文件
保存自定义配置文件到磁盘上的 `/etc/apparmor.d/containers/docker-nginx` 文件。
本示例中的文件路径不是必需的。在生产中可以使用其他路径。
#### 2 加载配置文件
```
$ sudo apparmor_parser -r -W /etc/apparmor.d/containers/docker-nginx
```
#### 3 使用这个配置文件运行容器
以 detached 模式运行 nginx：
```
$ docker run --security-opt "apparmor=docker-nginx" \
     -p 80:80 -d --name apparmor-nginx nginx
```
#### 4 Exec 进入运行中的容器
```
$ docker container exec -it apparmor-nginx bash
```
#### 5 尝试一些操作来测试配置文件
```
root@6da5a2a930b9:~# ping 8.8.8.8
ping: Lacking privilege for raw socket.

root@6da5a2a930b9:/# top
bash: /usr/bin/top: Permission denied

root@6da5a2a930b9:~# touch ~/thing
touch: cannot touch 'thing': Permission denied

root@6da5a2a930b9:/# sh
bash: /bin/sh: Permission denied

root@6da5a2a930b9:/# dash
bash: /bin/dash: Permission denied
```
恭喜！ 你刚刚部署了一个由定制的 apparmor 配置文件保护的容器！
# 4. 调试 AppArmor
可以使用 `dmesg` 来调试问题和 `aa-status` 检查加载的配置文件。
## 4.1 使用 dmesg
以下是一些有用的技巧，用于调试你可能面临的有关 AppArmor 的任何问题。

AppArmor 向 `dmesg` 发送非常详细的消息。通常，AppArmor 行如下所示：
```
[ 5442.864673] audit: type=1400 audit(1453830992.845:37): apparmor="ALLOWED" operation="open" profile="/usr/bin/docker" name="/home/jessie/docker/man/man1/docker-attach.1" pid=10923 comm="docker" requested_mask="r" denied_mask="r" fsuid=1000 ouid=0
```
在上面例子中可以看到 `profile=/usr/bin/docker`。这意味着用户已经加载了 `docker-engine` （Docker Engine Daemon）配置文件。

>注意：在 Ubuntu > 14.04 的版本中，这一切都很好，但 Trusty 用户在尝试 `docker container exec` 时可能遇到一些问题。

看另一行：
```
[ 3256.689120] type=1400 audit(1405454041.341:73): apparmor="DENIED" operation="ptrace" profile="docker-default" pid=17651 comm="docker" requested_mask="receive" denied_mask="receive"
```
这一次配置文件是 `docker-default`，除非在特权模式下，默认情况下运行在容器上。这一行显示 apparmor 已经在容器中拒绝了 ptrace。这完全如预期。
## 4.2 使用 aa-status
可以通过 `aa-status` 检查加载了哪个配置文件。输出风格如下：
```
$ sudo aa-status
apparmor module is loaded.
14 profiles are loaded.
1 profiles are in enforce mode.
   docker-default
13 profiles are in complain mode.
   /usr/bin/docker
   /usr/bin/docker///bin/cat
   /usr/bin/docker///bin/ps
   /usr/bin/docker///sbin/apparmor_parser
   /usr/bin/docker///sbin/auplink
   /usr/bin/docker///sbin/blkid
   /usr/bin/docker///sbin/iptables
   /usr/bin/docker///sbin/mke2fs
   /usr/bin/docker///sbin/modprobe
   /usr/bin/docker///sbin/tune2fs
   /usr/bin/docker///sbin/xtables-multi
   /usr/bin/docker///sbin/zfs
   /usr/bin/docker///usr/bin/xz
38 processes have profiles defined.
37 processes are in enforce mode.
   docker-default (6044)
   ...
   docker-default (31899)
1 processes are in complain mode.
   /usr/bin/docker (29756)
0 processes are unconfined but have a profile defined.
```
以上输出显示在不同 PID 的容器上运行的 `docker-default` 配置文件处于 `enforce` 模式。这意味着 AppArmor 正在主动阻止并在 `dmesg` 中审计 `docker-default` 配置文件边界之外的任何内容。

上面的输出还显示 `/usr/bin/docker`（Docker Engine 守护进程）配置文件正在以 `complain` 模式运行。这意味着 AppArmor 只会记录到配置文件边界之外的 `dmesg` 活动。（除了在 Ubuntu Trusty 的情况下，执行一些有趣的行为。）
# 5. 向 Docker 的 AppArmor 贡献代码
高级用户和包管理员可以在 Docker Engine 源代码仓库的 [contrib/apparmor](https://github.com/moby/moby/tree/master/contrib/apparmor) 下找到 `/usr/bin/docker`（Docker Engine Daemon）的配置文件。

容器的 `docker-default` 配置文件位于 [profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor) 中。
