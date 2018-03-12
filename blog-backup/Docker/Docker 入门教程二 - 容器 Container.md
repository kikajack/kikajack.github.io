[原文地址](https://docs.docker.com/get-started/part2/)

#1. 先决条件 Prerequisites
- 安装的 Docker 版本是 1.13 及以上。
- 读完 [第一部分](http://blog.csdn.net/kikajack/article/details/79350391)
- 用下面的命令快速测试你的环境是否完备：
```
docker run hello-world
```
#2. 概述
现在开始用 Docker 的方式构建应用。我们从这个应用的层次结构底部开始，也就是这里讲的容器。在容器层上面有第三部分讲的 service 层，定义了生产中的容器的行为方式。最顶层的是第五部分讲的 stack 层，定义了所有 service 的交互。

- Stack
- Services
- Container
#3. 新的部署环境
过去，如果要写个 Python 应用，首先要在机器上安装 Python 运行时。这就带来了一个问题：要使应用按照预期运行，就需要机器上的环境完美适合应用程序，同时生产环境需要与开发环境完全一致。
通过 Docker，可以将一个可移植的 Python 运行时作为一个 image 镜像获取，无需安装。 然后，构建时可以将基础 Python 镜像与应用程序代码一起包括在内，确保应用程序，依赖项和运行时都一起发布。
通过 Dockerfile 定义可移植的镜像。
#4. 用 Dockerfile 定义容器
Dockerfile 定义了容器中的环境包含哪些东西。对网络接口、磁盘等资源的访问被虚拟化到了这个环境内部，从而与系统的其他部分隔离，因此必须映射端口到外部，并且指明需要把哪些文件复制到容器内部。这些完成后，通过这个 Dockerfile 对应用的构建在任何地方运行时都会有相同的表现。
##4.1 Dockerfile
创建一个空目录。切换到这个新目录中，创建名为 `Dockerfile` 的文件并把下面的内容复制到这个文件中并保存。注意注释解释了每一条命令：
```
# 使用官方的 Python 运行时作为父容器
FROM python:2.7-slim

# 把工作目录设置为 /app
WORKDIR /app

# 把当前目录中的所有东西复制到容器中的 /app 目录中
ADD . /app

# 安装所有通过 requirements.txt 文件指定的需要的包
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# 开启容器的 80 端口，使得外部可以访问
EXPOSE 80

# 定义环境变量
ENV NAME World

# 容器启动时，运行 app.py 
CMD ["python", "app.py"]
```
##4.2 使用代理服务器
代理服务器会切断跟 web 应用的连接。如果使用了代理服务器，需要添加下面几个配置到 Dockerfile 文件中，通过 ENV 命令指定代理服务器的 host 和端口号：
```
# 设置代理服务器，将 host:port 替换为你的服务器的值
ENV http_proxy host:port
ENV https_proxy host:port
```
把这几行添加到 pip 命令之前，这样就可以成功安装了。

这个 Dockerfile 文件使用了两个还没定义的文件，下面创建这两个文件。
#5. 应用
在刚才创建的 Dockerfile 所在目录下继续创建两个文件，分别是 `requirements.txt` 和 `app.py`。当上面的 Dockerfile 被构建为一个镜像时，`requirements.txt` 和 `app.py` 都会在镜像里（通过 `ADD` 指令实现），`app.py` 的输出可以通过 HTTP 访问（通过 `EXPOSE` 指令实现）。
##5.1 requirements.txt
```
Flask
Redis
```
##5.2 app.py
```
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# 连接到 Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```
现在我们可以看到，`pip install -r requirements.txt` 为 Python 安装了 Flask 和 Redis 库，`app` 打印了环境变量 NAME，以及对`socket.gethostname()` 的调用的输出。最后，因为 Redis并没有运行（因为我们只安装了 Python 库，并没有安装 Redis 本身），我们应该失败并得到报错信息。
注意：在容器内获取主机名时，会获取到容器的 ID，类似运行中可执行文件的进程 ID。

这就完成了！并不需要在系统上提前安装 Python 或 `requirements.txt` 中的任何程序，也不需要构建或运行这个镜像来安装到系统上。看上去并没有设置 Python 和 Flask 的环境，但是已经设置好了。
#6. 构建应用
现在可以构建这个应用了。查看 Dockerfile 所在目录中的文件：
```
$ ls
Dockerfile    app.py      requirements.txt
```
运行 `build` 命令，创建 Docker 镜像。通过 `-t` 参数打标签，使镜像有一个容易记住的名字。
```
docker build -t friendlyhello .
```
新构建的镜像在哪里？在你机器的本地 Docker 镜像注册处（registry）：
```
$ docker image ls

REPOSITORY            TAG                 IMAGE ID
friendlyhello         latest              326387cea398
```
#7. 运行应用
##7.1 端口映射
运行应用程序，使用 `-p` 参数将机器的 4000 端口映射到容器的 80 端口：
```
docker run -p 4000:80 friendlyhello
```
本来应该在 `http://0.0.0.0:80` 看到 Python 服务的应用程序显示的消息。但是这个消息是从容器内获取的，容器并不知道它的 80 端口已经被映射到宿主机的 4000 端口，所有应该用这个 URL 来访问：`http://localhost:4000`，网页如下：

![浏览器中的镜像](http://img.blog.csdn.net/20180223160830188?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注意：如果在 Windows 7 中使用 Docker Toolbox，需要使用 Docker 所在机器的 IP 地址来访问，此时用 `localhost` 无效。例如：`http://192.168.99.100:4000/`。可以使用命令 `docker-machine ip` 查看地址。

也可以在命令行中使用 CURL 命令查看到相同的内容：
```
$ curl http://localhost:4000

<h3>Hello World!</h3><b>Hostname:</b> 8fc990912a14<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
```
这个端口映射 `4000:80` 用来演示通过 Dockerfile 的 EXPOSE 命令暴露的端口和使用命令 `docker run -p` 发布的不同。下面的步骤中，我们把容器的 80 端口映射到宿主机的 80 端口，使用 `http://localhost` 访问。
##7.2 退出容器
在终端中通过 `CTRL+C` 退出。

>Windows 中明确退出容器：
Windows 中，`CTRL+C` 无法停止容器。需要通过命令 `docker container stop <Container NAME or ID>` 来停止容器。否则，再次运行容器时会从守护进程返回错误。

##7.3 后台运行
，detached 模式：
```
docker run -d -p 4000:80 friendlyhello
```
##7.4 容器 ID
通过 `-d` 参数后台运行应用后，可以获取应用的长容器 ID，然后通过 ID 重新在终端操作容器。 还可以使用 `docker container ls` 查看缩写的容器 ID（也可以在运行命令时使用）：
```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED
1fa4ab2cf395        friendlyhello       "python app.py"     28 seconds ago
```
注意，CONTAINER ID 和在 `http://localhost:4000` 看到的一样。

现在通过 `docker container stop` 使用容器 ID 来结束进程：
```
docker container stop 1fa4ab2cf395
```
#8. 分享镜像
为了演示我们刚才创建的镜像的可移植性，上传我们构建的镜像并在其他地方运行。毕竟，如果想要将容器部署到生产环境，就需要知道如何上传镜像到 registry。

**registry 是仓库 repository 的集合，仓库是镜像的集合**。Docker 的仓库跟 GitHub 的仓库类似，除了代码已经构建好之外。registry 上的每个账号可以创建多个仓库。Docker CLI 命令行默认使用 Docker 的公共 registry。

注意：这里我们之所以使用 Docker 的公共 registry，是因为它免费且是配置好的。此外还有很多公共的 registry 可选，而且你可以使用 [Docker Trusted Registry](https://docs.docker.com/datacenter/dtr/2.2/guides/) 创建自己的私有 registry。
##8.1 用 Docker ID 登录
如果你还没有 Docker 账号，可以在 cloud.docker.com 注册一个。
在本地机器上登录 Docker 的公共 registry：
```
$ docker login
```
##8.2 给镜像打标签（Tag）
将一个本地镜像关联到注册处 registry 中的一个仓库的符号是 `username/repository:tag`。标签是可选的，但是建议使用，因为这是 registry 用来给 Docker 镜像指定版本的机制。给仓库和标签起有意义的名字，例如 `get-started:part2`。这会把镜像放入 `get-started` 仓库，并且添加标签 `part2`。

运行 `docker tag image` 命令，用自己的 username，repository 和 标签名，这样镜像可以上传到指定的位置。命令的语法是：
```
docker tag image username/repository:tag
```
举个例子：
```
docker tag friendlyhello john/get-started:part2
```
运行 [`docker image ls`](https://docs.docker.com/engine/reference/commandline/image_ls/) 来查看新打标签的镜像：
```
$ docker image ls
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
friendlyhello            latest              d9e555c53008        3 minutes ago       195MB
john/get-started         part2               d9e555c53008        3 minutes ago       195MB
python                   2.7-slim            1c7128a655f6        5 days ago          183MB
...
```
##8.3 发布镜像
`docker push` 命令将已经打标签的镜像上传到仓库中：
```
docker push username/repository:tag
```
上传完成后，这次上传的镜像就可以公开访问了。如果你登录了 [Docker Hub](https://hub.docker.com/)，就可以看见这个镜像和对应的 pull 命令。
##8.4 从远程仓库获取并运行镜像
从现在起，你可以使用 `docker run` 命令在任何机器上运行你的应用程序：
```
docker run -p 4000:80 username/repository:tag
```
如果镜像不在机器本地上，则 Docker 会从仓库获取镜像。
```
$ docker run -p 4000:80 john/get-started:part2
Unable to find image 'john/get-started:part2' locally
part2: Pulling from john/get-started
10a267c67f42: Already exists
f68a39a6a5e4: Already exists
9beaffc0cf19: Already exists
3c1fe835fb6b: Already exists
4c9f1fa8fcb8: Already exists
ee7d8f576a14: Already exists
fbccdcced46e: Already exists
Digest: sha256:0601c866aab2adcc6498200efd0f754037e909e5fd42069adeff72d1e2439068
Status: Downloaded newer image for john/get-started:part2
 * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
```
不管 `docker run` 在哪里运行，Docker 会获取你的镜像并运行（这里的镜像安装了 Python 和从 `requirements.txt` 文件指定的依赖，并会运行应用代码）。所有的东西都在一个包里，获取到就可以运行，不需要安装其他东西。
#9. 第二部分结论
下一章，学习如何在 service 中运行这个容器，从而扩展我们的应用程序。
#10. 概括和备忘单
这一章用的的基本命令列表：
```
docker build -t friendlyhello .          # 使用当前目录下的 Dockerfile 创建镜像
docker run -p 4000:80 friendlyhello      # 运行镜像 "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyhello   # 后台运行镜像，detached 模式
docker container ls            # 列出所有的运行中的容器
docker container ls -a         # 列出所有的容器，包括不运行的
docker container stop <hash>   # 友好的关闭指定容器
docker container kill <hash>   # 强制关闭指定容器
docker container rm <hash>     # 删除指定容器
docker container rm $(docker container ls -a -q)  # 删除所有容器
docker image ls -a                                # 列出所有镜像
docker image rm <image id>                 # 删除指定镜像
docker image rm $(docker image ls -a -q)   # 删除所有镜像
docker login                               # 用 Docker 凭证在当前 CLI 会话中登录
docker tag <image> username/repository:tag     # 给要上传到 registry 的镜像打标记
docker push username/repository:tag            # 将已经打标记的镜像上传到 registry
docker run username/repository:tag             # 从 registry 运行镜像
```