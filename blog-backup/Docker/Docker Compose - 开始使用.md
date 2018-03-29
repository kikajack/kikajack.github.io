[原文地址](https://docs.docker.com/compose/gettingstarted/)

在此页面上，你将构建一个在 Docker Compose 上运行的简单 Python Web 应用程序。该应用程序使用 Flask 框架并在 Redis 中维护一个计数器。虽然示例使用 Python，但即使你不熟悉这些概念，此处的演示也应该可以理解。
# 1. 先决条件
确保已经安装了 Docker Engine 和 Docker Compose。Python 和 Redis 可以由 Docker 镜像提供，不需要安装。
# 2. 第一步：设置
定义应用程序的依赖。
#### 1. 创建项目目录：
```
$ mkdir composetest
$ cd composetest
```
#### 2. 在项目目录中创建 `app.py` 文件并复制以下内容：
```
import time

import redis
from flask import Flask


app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```
本例中 redis 是应用程序网络上 redis 容器的主机名。我们使用 Redis 的默认端口6379。

>处理瞬态错误
请注意 `get_hit_count` 函数的写法。如果 redis 服务不可用，这个基本的重试循环让我们多次尝试我们的请求。这在启动过程中应用程序刚开始联机时非常有用，而且在应用程序的生命周期中需要重新启动 Redis 服务时，这也会使应用程序更具弹性。在集群中，这也有助于处理节点之间的瞬间连接下降。
#### 3. 在项目目录中创建 `requirements.py` 文件并复制以下内容：
```
flask
redis
```
# 3. 第二步：创建 Dockerfile
在这一步，编写构建 Docker 镜像的 Dockerfile。镜像包含 Python 应用程序的所有依赖，包括 Python 自身。

在你的项目目录中，创建名为 Dockerfile 的文件并复制以下内容：
```
FROM python:3.4-alpine
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```
这会指示 Docker 做下列事情：

- 构建一个基于 Python 3.4 镜像的镜像。
- 将当前目录复制到镜像中的 `/code` 目录。
- 设置工作目录为 `/code`。
- 安装 Python 依赖。
- 设置容器的默认目录位 `python app.py`。

关于如何编写 Dockerfile，请参考 [Docker 用户手册](https://docs.docker.com/engine/tutorials/dockerimages/#building-an-image-from-a-dockerfile) 和 [Dockerfile 手册](https://docs.docker.com/engine/reference/builder/)。
# 4. 第三步：在 Compose 文件中定义服务
在项目目录中创建 `docker-compose.yml` 文件，复制以下内容：
```
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
  redis:
    image: "redis:alpine"
```
这个 Compose 文件定义了两个服务，`web` 和 `redis`。`web` 服务：

- 使用从当前目录中的 Dockerfile 构建的镜像。
- 将容器上的 5000 端口转发到主机上的5000 端口。使用 Flask Web服 务器的默认端口 5000。

`redis` 服务使用从 Docker Hub registry 下载的公共 Redis 镜像。
# 5. 第四步：通过 Compose 构建并运行应用
#### 1. 通过 `docker-compose up` 在项目目录中启动应用
```
$ docker-compose up
Creating network "composetest_default" with the default driver
Creating composetest_web_1 ...
Creating composetest_redis_1 ...
Creating composetest_web_1
Creating composetest_redis_1 ... done
Attaching to composetest_web_1, composetest_redis_1
web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
redis_1  | 1:C 17 Aug 22:11:10.480 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis_1  | 1:C 17 Aug 22:11:10.480 # Redis version=4.0.1, bits=64, commit=00000000, modified=0, pid=1, just started
redis_1  | 1:C 17 Aug 22:11:10.480 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
web_1    |  * Restarting with stat
redis_1  | 1:M 17 Aug 22:11:10.483 * Running mode=standalone, port=6379.
redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
web_1    |  * Debugger is active!
redis_1  | 1:M 17 Aug 22:11:10.483 # Server initialized
redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
web_1    |  * Debugger PIN: 330-787-903
redis_1  | 1:M 17 Aug 22:11:10.483 * Ready to accept connections
```
Compose 会下载 Redis 镜像，从你的代码构建镜像，并启动定义的服务。这个例子中，代码在构建时直接复制到镜像中。
#### 2. 在浏览器中输入 `http://0.0.0.0:5000/` 查看应用状态
如果是在本地运行 Docker，web 应用程序会在 Docker 守护进程宿主机的 5000 端口监听。在浏览器中输入 `http://0.0.0.0:5000/` 查看 `Hello World` 消息。如果没有反应，可以试试 `http://0.0.0.0:5000`。

如果你在 Mac 或 Windows 上使用 Docker Machine，使用 `docker-machine ip MACHINE_VM` 来获取 Docker 主机的 IP 地址。然后在浏览器中打开 `http://MACHINE_VM_IP:5000`。

你应该在浏览器中看到这句话：
```
Hello World! I have been seen 1 times.
```
![quick-hello-world-1](http://img.blog.csdn.net/20180329220336256?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#### 3. 刷新页面
数字会增加。
```
Hello World! I have been seen 2 times.
```
![quick-hello-world-2](http://img.blog.csdn.net/20180329220415718?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#### 4. 切换到另一个终端窗口，通过 `docker image ls` 列出本地镜像
此刻列出的镜像应该包含 `redis` 和 `web`：
```
$ docker image ls
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
composetest_web         latest              e2c21aa48cc1        4 minutes ago       93.8MB
python                  3.4-alpine          84e6077c7ab6        7 days ago          82.5MB
redis                   alpine              9d8fa9aa0e5b        3 weeks ago         27.5MB
```
可以通过 `docker inspect <tag or id>` 检查镜像。
#### 5. 停止应用程序。
在项目目录中执行 `docker-compose down`，或在启动应用程序的终端中按下 CTRL+C。
# 6. 第五步：编辑 Compose 文件，添加绑定挂载
在项目目录中编辑 `docker-compose.yml`，为 `web` 服务添加 [绑定挂载](https://docs.docker.com/engine/admin/volumes/bind-mounts/)：
```
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
    volumes:
     - .:/code
  redis:
    image: "redis:alpine"
```
新的 `volumes` 关键字将主机上的项目目录（当前目录）挂载到容器中的 `/code` 中，允许你即时修改代码，而无需重新构建镜像。
# 7. 第六步：通过 Compose 重新构建并运行应用
从项目目录中，输入 `docker-compose up`，用更新后的 Compose 文件构建应用程序并运行。
```
$ docker-compose up
Creating network "composetest_default" with the default driver
Creating composetest_web_1 ...
Creating composetest_redis_1 ...
Creating composetest_web_1
Creating composetest_redis_1 ... done
Attaching to composetest_web_1, composetest_redis_1
web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
...
```
在浏览器中再次检查 `Hello World` 消息，并刷新看看计数会不会增加。

>共享文件夹，卷和绑定挂载
>
>- 如果你的项目不在 `Users` 目录（`cd ~`）中，你需要共享正在使用的 Dockerfile 和卷的驱动程序或位置。如果收到运行时错误，指示找不到应用程序文件，卷挂载被拒绝或服务无法启动，请尝试启用文件或驱动程序共享。对于使用 Linux 容器的 Docker for Windows 上的任何项目，需要使用共享驱动程序来存放 `C:\Users`（Windows）或 `/Users`（Mac）以外的项目。更多信息，请参阅 Docker for Windows 上的共享驱动器，Docker for Mac 上的文件共享以及有关如何管理容器中的数据的一般示例。
>- 如果在较早的 Windows 操作系统上使用 Oracle VirtualBox，则可能会遇到如本 VB 故障单所述的共享文件夹问题。较新的 Windows 系统满足 Docker for Windows 的要求，并且不需要 VirtualBox。
# 8. 第七步：升级应用程序
由于应用程序代码现在使用卷挂载到容器中，因此可以更改代码并立即查看效果，而无需重新生成镜像。
#### 1. 更改 `app.py` 中的问候语并保存。例如，更改 `Hello World!` 为 `Hello from Docker!`：
```
return 'Hello from Docker! I have been seen {} times.\n'.format(count)
```
#### 2. 在浏览器中刷新应用。问候语应该会刷新，计数器也会增加。
![quick-hello-world-3](http://img.blog.csdn.net/20180329224233549?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
# 9. 第八步：体验其他命令
如果要在后台运行服务，可以在 `docker-compose up` 命令中使用 `-d` 标志（“detached”模式），可以通过 `docker-compose ps` 命令查看当前运行的服务：
```
$ docker-compose up -d
Starting composetest_redis_1...
Starting composetest_web_1...

$ docker-compose ps
Name                 Command            State       Ports
-------------------------------------------------------------------
composetest_redis_1   /usr/local/bin/run         Up
composetest_web_1     /bin/sh -c python app.py   Up      5000->5000/tcp
```
`docker-compose run` 命令允许为服务运行一次性指令。例如，要查看哪些环境变量可用于 `web` 服务：
```
$ docker-compose run web env
```
其他可用命令可以通过 `docker-compose --help` 来查看。也可以为 bash 或 zsh 脚本 [安装命令补全](https://docs.docker.com/compose/completion/)，这样可以自动显示可用命令。

如果你通过 `docker-compose up -d` 启动 Compose，不需要服务时请停止服务：
```
$ docker-compose stop
```
可以卸载所有东西，用 `down` 命令完全移除容器。通过 `--volumes` 还可以删除 Redis 容器使用的数据卷：
```
$ docker-compose down --volumes
```
现在，你已经知道了 Compose 的基本工作原理。
