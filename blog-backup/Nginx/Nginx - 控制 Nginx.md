[原文地址](http://nginx.org/en/docs/control.html)

可以通过信号 signals 来控制 Nginx。主进程的进程 ID 默认写到文件 /usr/local/nginx/logs/nginx.pid 中，其中文件名可以在源码安装时更改，或在 nginx.conf 配置文件中使用 pid 指令来更改。主进程支持以下信号：

- TERM, INT：快速停止服务。fast shutdown
- QUIT：平滑的停止服务。graceful shutdown
- HUP：更改配置，跟上更改的时区（仅适用于FreeBSD和Linux），使用新配置启动新的工作进程（worker process），平滑的关闭旧工作进程。
- USR1：重新打开日志文件。
- USR2：升级可执行文件。
- WINCH：平滑的关闭工作进程。

每一个工作进程也可以通过信号 signals 来单独控制，虽然没啥必要：

- TERM, INT：快速停止服务。
- QUIT：平滑的停止服务。
- USR1：重新打开日志文件。
- WINCH：调试异常终止（需要启用debug_points）。

#1. 更改配置
为了让 nginx 重新读取配置文件，应该将一个 HUP 信号发送到主进程。主进程首先检查配置文件的语法有效性，然后尝试应用新配置，即打开日志文件和新的侦听套接字。如果失败，它会回滚更改并继续使用旧配置。如果成功，它会启动新的工作进程，并向旧的工作进程发送信号，请求他们正常关闭。旧的工作进程关闭监听套接字不再新建连接，但是会继续为已有的连接服务。旧的工作进程会在所有客户端都服务完成之后退出。

举例，nginx 在 FreeBSD 4.x 命令上运行：
```
ps axw -o pid,ppid,user,%cpu,vsz,wchan,command | egrep '(nginx|PID)'
```
上面命令结果如下：
```
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
33127 33126 nobody   0.0  1380 kqread nginx: worker process (nginx)
33128 33126 nobody   0.0  1364 kqread nginx: worker process (nginx)
33129 33126 nobody   0.0  1364 kqread nginx: worker process (nginx)
```
如果发送 HUP 信号给主进程，输出如下：
```
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
33129 33126 nobody   0.0  1380 kqread nginx: worker process is shutting down (nginx)
33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33135 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
```
PID 为 33129 的旧的工作进程会继续工作一段时间再退出：
```
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33135 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
```
#2. 改变日志文件
为了改变日志文件，首先要重命名。然后发送 USR1 信号给主进程。主进程会重新打开所有的日志文件，并将其分配给正在运行工作进程的非特权用户。重新打开日志文件成功后，主进程会关闭所有打开的日志文件并发消息给工作进程，让工作进程重新打开文件。旧文件可以用于后期处理，比如压缩。
#3. 热更新可执行文件（服务不间断）
为了升级服务器可执行文件，应该先用新的可执行文件替换旧文件（放在磁盘上的对应位置即可）。然后发送 USR2 信号到主进程。主进程首先使用进程标识将其文件重命名为带有 .oldbin 后缀的新文件，例如 /usr/local/nginx/logs/nginx.pid.oldbin，然后启动这个新的可执行文件，然后启动新的工作进程：
```
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33135 33126 nobody   0.0  1380 kqread nginx: worker process (nginx)
33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
```
之后，所有工作进程（旧的和新的）继续接受请求。如果旧的主进程接受到 WINCH 信号，它将向其工作进程发送消息，请求它们正常关闭：
```
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
33135 33126 nobody   0.0  1380 kqread nginx: worker process is shutting down (nginx)
36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
```
一段时间后，只有新的工作进程会处理请求：
```
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
```
应该注意的是，旧的主进程没有关闭其侦听套接字，并且可以在需要时再次启动它的工作进程。如果由于某种原因，新的可执行文件无法正常工作，可以执行以下操作之一：

- 将 HUP 信号发送到旧的主进程。旧的主进程将启动新的工作进程而不重新读取配置。之后，将 QUIT 信号发送到新的主进程，正常关闭所有新进程。
- 发送 TERM 信号到新的主进程。然后它会向其工作进程发送一条消息，要求他们立即退出，并且他们几乎立即退出。（如果新进程因某种原因未退出，则应将 KILL 信号发送给它们以强制它们退出。）当新主进程退出时，旧主进程将自动启动新工作进程。

如果新的主进程退出，则旧的主进程将去掉进程 ID 对应文件的 .oldbin 后缀。
如果升级成功，则应该发送 QUIT 信号给旧的主进程使其退出：
```
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
36264     1 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
```