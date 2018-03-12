[原文地址](https://docs.docker.com/config/containers/multi-service_container/)

容器的主要运行进程是 Dockerfile 末尾的 ENTRYPOINT 和 CMD。通常建议通过每个容器运行一个服务来分隔关注区域。该服务可以分成多个进程（例如，Apache Web 服务器启动多个工作进程）。虽然可以有多个进程，但为了高效利用 Docker，避免一个容器负责整个应用程序的多个方面。可以使用用户定义的网络和共享卷来连接多个容器。

容器的主进程负责管理它启动的所有进程。在某些情况下，主进程设计的不好，并且在容器退出时无法正常处理“reaping”（stopping）子进程。如果你的进程属于这个类别，你可以在运行容器时使用 `--init` 选项。`--init` 标志将一个很小的 `init` 进程作为主进程插入到容器中，并在容器退出时清理所有进程。以这种方式处理这些进程优于使用完整的初始化进程，如 `sysvinit`，`upstart` 或 `systemd` 来处理容器中的进程生命周期。

如果需要在一个容器中运行多个服务，可以有多种实现方式：

- 将所有命令放入包装脚本中，并附带测试和调试信息。运行包装脚本作为你的CMD。这是一个非常简单的例子。首先，包装脚本：
```
#!/bin/bash

# Start the first process
./my_first_process -D
status=$?
if [ $status -ne 0 ]; then
  echo "Failed to start my_first_process: $status"
  exit $status
fi

# Start the second process
./my_second_process -D
status=$?
if [ $status -ne 0 ]; then
  echo "Failed to start my_second_process: $status"
  exit $status
fi

# Naive check runs checks once a minute to see if either of the processes exited.
# This illustrates part of the heavy lifting you need to do if you want to run
# more than one service in a container. The container exits with an error
# if it detects that either of the processes has exited.
# Otherwise it loops forever, waking up every 60 seconds

while sleep 60; do
  ps aux |grep my_first_process |grep -q -v grep
  PROCESS_1_STATUS=$?
  ps aux |grep my_second_process |grep -q -v grep
  PROCESS_2_STATUS=$?
  # If the greps above find anything, they exit with 0 status
  # If they are not both 0, then something is wrong
  if [ $PROCESS_1_STATUS -ne 0 -o $PROCESS_2_STATUS -ne 0 ]; then
    echo "One of the processes has already exited."
    exit -1
  fi
done
```
然后是 Dockerfile：
```
FROM ubuntu:latest
COPY my_first_process my_first_process
COPY my_second_process my_second_process
COPY my_wrapper_script.sh my_wrapper_script.sh
CMD ./my_wrapper_script.sh
```
- 使用类似 `supervisord` 的进程管理器。这是一种中等重量的方法，需要将 `supervisord` 及其配置打包到镜像中（或将映像基于supervisord的映像），跟它所管理的不同应用程序放在一起。然后启动 `supervisord`，它为你管理你的进程。下面是一个使用这种方法的 Dockerfile 示例，它假定预先写好的 `supervisord.conf`，`my_first_process` 和 `my_second_process` 文件都与 Dockerfile 存在于同一个目录中。
```
FROM ubuntu:latest
RUN apt-get update && apt-get install -y supervisor
RUN mkdir -p /var/log/supervisor
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY my_first_process my_first_process
COPY my_second_process my_second_process
CMD ["/usr/bin/supervisord"]
```
