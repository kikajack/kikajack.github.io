[原文地址](http://111.230.25.113:4000/engine/security/seccomp/)

安全计算模式（secure computing mode，`seccomp`）是 Linux 内核功能。可以使用它来限制容器内可用的操作。`seccomp()` 系统调用在调用进程的 `seccomp` 状态下运行。可以使用此功能来限制你的应用程序的访问权限。

只有在使用 `seccomp` 构建 Docker 并且内核配置了 `CONFIG_SECCOMP` 的情况下，此功能才可用。要检查你的内核是否支持 `seccomp`：
```
$ cat /boot/config-`uname -r` | grep CONFIG_SECCOMP=
CONFIG_SECCOMP=y
```
>注意：`seccomp` 配置文件需要 seccomp 2.2.1，这在 Ubuntu 14.04，Debian Wheezy 或 Debian Jessie 中不可用。要在这些发行版上使用 `seccomp`，必须下载 [最新的静态 Linux 二进制文件](http://111.230.25.113:4000/engine/installation/linux/docker-ce/binaries/)（而不是软件包）。
# 1. 为容器传递配置文件
默认的 `seccomp` 配置文件为使用 `seccomp` 运行容器提供了一个合理的设置，并禁用了大约 44 个超过 300+ 的系统调用。它具有适度的保护性，同时提供广泛的应用兼容性。默认的 Docker 配置文件可以在 [这里](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json) 找到。

实际上，该配置文件是白名单，默认情况下阻止访问所有的系统调用，然后将特定的系统调用列入白名单。该配置文件工作时需要定义 `SCMP_ACT_ERRNO` 的 `defaultAction` 并仅针对特定的系统调用覆盖该 `action`。`SCMP_ACT_ERRNO` 的影响是触发 `Permission Denied` 错误。接下来，配置文件中通过将 `action` 被覆盖为 `SCMP_ACT_ALLOW`，定义一个完全允许的系统调用的特定列表。最后，一些特定规则适用于个别的系统调用，如 `personality`，`socket`，`socketcall` 等，以允许具有特定参数的那些系统调用的变体（to allow variants of those system calls with specific arguments）。

`seccomp` 有助于以最小权限运行 Docker 容器。不建议更改默认的 `seccomp` 配置文件。

运行容器时，如果没有通过 `--security-opt` 选项覆盖容器，则会使用默认配置。例如，以下显式指定了一个策略：
```
$ docker run --rm \
             -it \
             --security-opt seccomp=/path/to/seccomp/profile.json \
             hello-world
```
## 1.1 默认配置文件阻止的重要的系统调用
Docker 的默认 `seccomp` 配置文件是一个白名单，它指定了允许的调用。下表列出了由于不在白名单而被有效阻止的重要（但不是全部）系统调用。该表包含每个系统调用被阻止的原因。

Syscall|	Description
-|-
acct|	Accounting syscall which could let containers disable their own resource limits or process accounting. Also gated by CAP_SYS_PACCT.
add_key|	Prevent containers from using the kernel keyring, which is not namespaced.
adjtimex|	Similar to clock_settime and settimeofday, time/date is not namespaced. Also gated by CAP_SYS_TIME.
bpf|	Deny loading potentially persistent bpf programs into kernel, already gated by CAP_SYS_ADMIN.
clock_adjtime|	Time/date is not namespaced. Also gated by CAP_SYS_TIME.
clock_settime|	Time/date is not namespaced. Also gated by CAP_SYS_TIME.
clone|	Deny cloning new namespaces. Also gated by CAP_SYS_ADMIN for CLONE_* flags, except CLONE_USERNS.
create_module|	Deny manipulation and functions on kernel modules. Obsolete. Also gated by CAP_SYS_MODULE.
delete_module|	Deny manipulation and functions on kernel modules. Also gated by CAP_SYS_MODULE.
finit_module|	Deny manipulation and functions on kernel modules. Also gated by CAP_SYS_MODULE.
get_kernel_syms|	Deny retrieval of exported kernel and module symbols. Obsolete.
get_mempolicy|	Syscall that modifies kernel memory and NUMA settings. Already gated by CAP_SYS_NICE.
init_module	|Deny manipulation and functions on kernel modules. Also gated by CAP_SYS_MODULE.
ioperm|	Prevent containers from modifying kernel I/O privilege levels. Already gated by CAP_SYS_RAWIO.
iopl	|Prevent containers from modifying kernel I/O privilege levels. Already gated by CAP_SYS_RAWIO.
kcmp|	Restrict process inspection capabilities, already blocked by dropping CAP_PTRACE.
kexec_file_load|	Sister syscall of kexec_load that does the same thing, slightly different arguments. Also gated by CAP_SYS_BOOT.
kexec_load|	Deny loading a new kernel for later execution. Also gated by CAP_SYS_BOOT.
keyctl	|Prevent containers from using the kernel keyring, which is not namespaced.
lookup_dcookie|	Tracing/profiling syscall, which could leak a lot of information on the host. Also gated by CAP_SYS_ADMIN.
mbind|	Syscall that modifies kernel memory and NUMA settings. Already gated by CAP_SYS_NICE.
mount	|Deny mounting, already gated by CAP_SYS_ADMIN.
move_pages|	Syscall that modifies kernel memory and NUMA settings.
name_to_handle_at|	Sister syscall to open_by_handle_at. Already gated by CAP_SYS_NICE.
nfsservctl	|Deny interaction with the kernel nfs daemon. Obsolete since Linux 3.1.
open_by_handle_at|	Cause of an old container breakout. Also gated by CAP_DAC_READ_SEARCH.
perf_event_open|	Tracing/profiling syscall, which could leak a lot of information on the host.
personality	|Prevent container from enabling BSD emulation. Not inherently dangerous, but poorly tested, potential for a lot of kernel vulns.
pivot_root	|Deny pivot_root, should be privileged operation.
process_vm_readv	|Restrict process inspection capabilities, already blocked by dropping CAP_PTRACE.
process_vm_writev	|Restrict process inspection capabilities, already blocked by dropping CAP_PTRACE.
ptrace	|Tracing/profiling syscall, which could leak a lot of information on the host. Already blocked by dropping CAP_PTRACE.
query_module	|Deny manipulation and functions on kernel modules. Obsolete.
quotactl	|Quota syscall which could let containers disable their own resource limits or process accounting. Also gated by CAP_SYS_ADMIN.
reboot|	Don’t let containers reboot the host. Also gated by CAP_SYS_BOOT.
request_key	|Prevent containers from using the kernel keyring, which is not namespaced.
set_mempolicy|	Syscall that modifies kernel memory and NUMA settings. Already gated by CAP_SYS_NICE.
setns	|Deny associating a thread with a namespace. Also gated by CAP_SYS_ADMIN.
settimeofday|	Time/date is not namespaced. Also gated by CAP_SYS_TIME.
socket, socketcall|	Used to send or receive packets and for other socket operations. All socket and socketcall calls are blocked except communication domains AF_UNIX, AF_INET, AF_INET6, AF_NETLINK, and AF_PACKET.
stime	|Time/date is not namespaced. Also gated by CAP_SYS_TIME.
swapon	|Deny start/stop swapping to file/device. Also gated by CAP_SYS_ADMIN.
swapoff|	Deny start/stop swapping to file/device. Also gated by CAP_SYS_ADMIN.
sysfs|	Obsolete syscall.
_sysctl	|Obsolete, replaced by /proc/sys.
umount	|Should be a privileged operation. Also gated by CAP_SYS_ADMIN.
umount2	|Should be a privileged operation. Also gated by CAP_SYS_ADMIN.
unshare|	Deny cloning new namespaces for processes. Also gated by CAP_SYS_ADMIN, with the exception of unshare --user.
uselib	|Older syscall related to shared libraries, unused for a long time.
userfaultfd|	Userspace page fault handling, largely needed for process migration.
ustat	|Obsolete syscall.
vm86	|In kernel x86 real mode virtual machine. Also gated by CAP_SYS_ADMIN.
vm86old	|In kernel x86 real mode virtual machine. Also gated by CAP_SYS_ADMIN.
# 2. 不使用默认的 seccomp 配置文件
可以传递 `unconfined` 以运行没有默认 `seccomp` 配置文件的容器。
```
$ docker run --rm -it --security-opt seccomp=unconfined debian:jessie \
    unshare --map-root-user --user sh -c whoami
```
