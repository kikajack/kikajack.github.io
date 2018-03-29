[原文地址](https://docs.docker.com/compose/environment-variables/)
https://docs.docker.com

Compose 的多个部分在某种情况下处理环境变量。本教程可以帮助你找到所需的信息。
# 1. 替换Compose文件中的环境变量
可以使用 shell 中的环境变量填充 Compose 文件中的值：
```
web:
  image: "webapp:${TAG}"
```
更多信息请参考 Compose 文件手册中的 [Variable substitution](https://docs.docker.com/compose/compose-file/#variable-substitution) 章节。
# 2. 设置容器中的环境变量
可以通过 [environment](http://111.230.25.113:4000/compose/compose-file/#environment) 关键字设置服务容器中的环境变量，就跟使用 `docker run -e VARIABLE=VALUE ...` 一样：
```
web:
  environment:
    - DEBUG=1
```
# 3. 将环境变量传递到容器
在使用 `environment` 关键字时不赋值，就可以将 shell 中的环境变量传递给服务容器，就跟使用 `docker run -e VARIABLE ...` 一样：
```
web:
  environment:
    - DEBUG
```
容器中的 `DEBUG` 变量的值从运行 Compose 的 shell 中的同名变量中获取。
# 4. “env_file”配置选项
可以通过 `env_file` 命令使用外部文件将多个环境变量传递到服务容器，就跟使用 `docker run --env-file=FILE ...` 一样：
```
web:
  env_file:
    - web-variables.env
```
# 5. 使用 ‘docker-compose run’设置环境变量
就像 `docker run -e` 命令一样，可以使用 `docker-compose run -e` 设置一次性容器上的环境变量：
```
docker-compose run -e DEBUG=1 web python console.py
```
也可以通过从 shell 中传递一个变量，而不是直接赋值：
```
docker-compose run -e DEBUG web python console.py
```

容器中的 `DEBUG` 变量的值从运行 Compose 的 shell 中的同名变量中获取。
# 6. “.env”文件
可以在名为 [.env 的环境文件](https://docs.docker.com/compose/env-file/) 中为 Compose 文件中引用的任何环境变量设置默认值，或者用于配置 Compose：
```
$ cat .env
TAG=v1.5

$ cat docker-compose.yml
version: '3'
services:
  web:
    image: "webapp:${TAG}"
```
运行 `docker-compose up` 时，上面定义的 `web` 服务使用 `webapp:v1.5` 镜像。可以通过 [config 命令](https://docs.docker.com/compose/reference/config/) 将应用程序的配置信息打印到终端来验证：
```
$ docker-compose config

version: '3'
services:
  web:
    image: 'webapp:v1.5'
```
shell 中的值优先于 `.env` 文件中指定的值。如果在 shell 中将 TAG 设置为不同的值，则镜像中将使用该值：
```
$ export TAG=v2.0
$ docker-compose config

version: '3'
services:
  web:
    image: 'webapp:v2.0'
```
当在多个文件中设置相同的环境变量时，以下是 Compose 用于选择要使用的值的优先级：

1. Compose 文件
2. Environment 文件
3. Dockerfile
4. 变量未定义

在下面的例子中，我们在 Environment 文件和 Compose 文件上设置了相同的环境变量：
```
$ cat ./Docker/api/api.env
NODE_ENV=test

$ cat docker-compose.yml
version: '3'
services:
  api:
    image: 'node:6-alpine'
    env_file:
     - ./Docker/api/api.env
    environment:
     - NODE_ENV=production
```
运行容器时，在 Compose 文件中定义的环境变量优先。
```
$ docker-compose exec api node

process.env.NODE_ENV
'production'
```
只有在 `environment ` 或 `env_file` 没有 Docker Compose 条目时，`Dockerfile` 中的任何 `ARG` 或 `ENV` 设置才会评估（evaluate）。

> NodeJS 容器的细节
>
>如果你有脚本的 `package.json` 条目像 `NODE_ENV=test node server.js` 一样启动，那么这将覆盖 `docker-compose.yml` 文件中的任何设置。
# 7. 使用环境变量配置 Compose
有几个环境变量可用来配置 Docker Compose 命令行行为。它们以 `COMPOSE_` 或 `DOCKER_` 开头，并记录在 [CLI 环境变量中](https://docs.docker.com/compose/reference/envvars/)。
# 8. 通过 link 创建环境变量
在第一版 Compose 文件中使用 [links](https://docs.docker.com/compose/compose-file/#links) 选项时，会为每个链接创建环境变量。它们记录在 [Link环境变量参考](https://docs.docker.com/compose/link-env-deprecated/) 中。

但是，这些变量已被弃用。link 改为为主机创建别名。
