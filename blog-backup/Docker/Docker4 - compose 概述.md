[官方文档](https://docs.docker.com/compose/overview/)
[中文资料参考这里](https://beginor.github.io/2017/06/08/use-compose-instead-of-run.html)
#一. docker-compose 概述
###1. docker run 命令的缺点
docker run 启动每一个镜像的时候， 需要指定一些参数（容器名称name、 映射的卷volume、 绑定的端口publish、 网络以及重启策略restart）。
```
docker run \
  --detach \
  --name registry \
  --hostname registry \
  --volume $(pwd)/app/registry:/var/lib/registry \
  --publish 5000:5000 \
  --restart unless-stopped \
  registry:latest
```
当镜像数量增多时，需要专门编排一下，docker-compose 派上用场。
###2. docker-compose
Compose 是定义和运行多个容器的工具。使用 Compose，可以通过 YAML 文件来配置 Docker。然后，通过单个命令创建并启动配置文件中的所有 Docker 服务。
使用 Compose 的三个步骤：

1. 定义应用程序的 Dockerfile，以便它可以在任何地方复制。
2. 定义组成应用程序的 `docker-compose.yml`，使各个 Docker 可以在隔离的环境中一起运行。
3. 最后，运行 docker-compose up和撰写将启动并运行您的整个应用程序。
#二. 使用
##1. 命令
通常将 docker-compose.yml 文件放到一个目录， 表示一个应用， docker 会为这个应用创建一个独立的网络， 便于和其它应用进行隔离。
`docker-compose up -d`：按照配置启动容器的实例。
`docker-compose down`：停止实例。
##2. 示例
在Windows上搭建开发环境，安装 Nginx 和 PHP 应用。
项目目录架构：
```
php-dev
├── docker-compose.yml
├── project
│   └── index.php
└── nginx
    └── conf
        └── nginx.conf
```
###1）安装 Docker
win10 已经集成了 Docker Compose，安装过程这里不再墨迹。
###2）文件系统映射
进入settings 设置页面，选择 Shared Drives，选择要映射的磁盘，确定即可。注意部分版本的 Docker 必须要用户设置 windows 系统的登录密码。
###3）安装 PHP
选择带 fpm 的 PHP 精简版安装镜像 `php:7.0.8-fpm-alpine`，执行命令安装：
`docker pull php:7.0.8-fpm-alpine`
###4）安装 Nginx
同样用精简版 `nginx:mainline-alpine`，执行命令安装：
`docker pull nginx:mainline-alpine`
###5）编写docker-compose.yml代码如下：
```
version: '3'
services:
  php:
    image: php:7.0.8-fpm-alpine
    command: php-fpm
    ports:
      - "9123:9000"
    volumes:
      - ./project:/var/www/html:ro
  nginx:
    image: nginx:mainline-alpine
    command: nginx -g 'daemon off;'
    volumes:
      - ./project:/usr/share/nginx/html:ro
      - ./nginx/conf/nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - "8880:80"
    links:
      - php
```
###6）写 nginx.conf 配置文件
部分配置如下：
```
    server {
        listen       80;
        server_name  localhost;
    error_log /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;

        root   /var/www/html;
        location / {
            if (-f $request_filename) {
                expires max;
                break;
            }
            if (!-e $request_filename) {
                rewrite ^/(.*)$ /index.php/$1 last;
            }
            index  index.html index.htm index.php;
            autoindex  off;
        }

        location ~ \.php(.*)$  {
      root           /usr/share/nginx/html;
            fastcgi_pass   127.0.0.1:9123;
            fastcgi_index  index.php;
            fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            fastcgi_param  PATH_INFO  $fastcgi_path_info;
            fastcgi_param  PATH_TRANSLATED  $document_root$fastcgi_path_info;
            include        fastcgi_params;
        }
    }
```
###7）构建容器
`docker-compose up -d`：在项目根目录下根据 compose 配置文件编排容器，`-d` 表示以后台进程方式启动。
`docker-compose stop` ：停止 compose 启动的容器。

启动容器后，用`docker exec -it 容器id /bin/bash` 可以进入指定的容器。

现在，可以访问 localhost:8880 了。