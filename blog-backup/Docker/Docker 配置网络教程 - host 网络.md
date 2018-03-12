[原文地址](https://docs.docker.com/network/network-tutorial-host/)

这部分教程中，独立容器直接连接到 Docker 主机的网络，没有网络隔离。
#1. 目标
这个教程的目的是启动一个直接绑定到 Docker 主机的 80 端口的 Nginx 容器。从网络角度来看，这跟 Nginx 进程直接运行在 Docker 主机上而不是容器中具有相同等级的隔离。然而，其他方面，比如存储，进程命名空间和用户命名空间，Nginx 还是和主机隔离的。
#2. 先决条件
- 此过程要求端口 80 在 Docker 主机上可用。要使 Nginx 在不同的端口上侦听，请参阅 [nginx 镜像文档](https://hub.docker.com/_/nginx/)
- host 网络驱动程序只在 Linux 主机上工作，其他类似 Mac、Windows 的平台无法使用
#3. 过程
##3.1 创建并将容器启动为后台进程
```
docker run --rm -itd --network host --name my_nginx nginx
```
##3.2 在浏览器中输入 http://localhost:80/ 访问 Nginx
##3.3 检查网络堆栈
- 检查所有网络接口并验证新接口创建成功
```
ip addr show
```
- 通过 `netstat` 命令验证哪个进程绑定到了 80 端口。因为进程属于 Docker 守护进程用户，这里需要使用 `sudo` 才能看到名字和 PID。
```
sudo netstat -tulpn | grep :80
```
##3.4 停止容器
···
docker container stop my_nginx
docker container rm my_nginx
···