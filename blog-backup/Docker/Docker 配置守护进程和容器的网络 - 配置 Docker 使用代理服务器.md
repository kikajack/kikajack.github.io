[原文地址](https://docs.docker.com/network/proxy/)

如果你的容器需要使用 HTTP、HTTPS 或 FTP 代理服务器，可以有多种配置方式：

- 对于 Docker 17.07 或更高版本，可以配置 Docker 客户端，将代理信息自动发送到容器。
- 对于 Docker 17.06 或更低版本，必须在 Docker 内部设置合适的环境变量。可以在构建镜像时（会降低镜像灵活性）或在创建或运行容器时进行这一步。
#1. 配置 Docker 客户端
####1.1 
在 Docker 客户端上，创建或编辑启动容器的用户对应的主目录中的 `~/.docker/config.json` 文件。添加如下所示的 JSON，必要时用 `httpsProxy` 或 `ftpProxy` 替换代理服务器的类型，并替换代理服务器的地址和端口。可以同时配置多个代理服务器。

通过将 `noProxy` 键设置为一个或多个逗号分隔的 IP 地址或主机，可以选择性地排除通过代理服务器的主机或范围。如本例所示，支持使用 `*` 字符作为通配符。
```
{
 "proxies":
 {
   "default":
   {
     "httpProxy": "http://127.0.0.1:3001",
     "noProxy": "*.test.example.com,.example2.com"
   }
 }
}
```
保存这个文件。
####1.2 在创建或启动新容器时，环境变量会在容器中自动设置。
#2. 使用环境变量
##2.1 手动设置环境变量
构建镜像时，或在创建或运行容器时使用 `--env` 标志，可以将一个或多个下面的环境变量设置为合适的值。这个方法会降低镜像灵活性，所以如果你的 Docker 版本是 17.07 或更高，你应该配置 Docker 客户端。

|变量|  Dockerfile 示例|  `docker run` 示例|
|-|-|-|
|`HTTP_PROXY`|  `ENV HTTP_PROXY "http://127.0.0.1:3001"`| `--env HTTP_PROXY "http://127.0.0.1:3001"`
|`HTTPS_PROXY`| `ENV HTTPS_PROXY "https://127.0.0.1:3001"`| `--env HTTPS_PROXY "https://127.0.0.1:3001"`
|`FTP_PROXY`| `ENV FTP_PROXY "ftp://127.0.0.1:3001"`| `--env FTP_PROXY "ftp://127.0.0.1:3001"`
|`NO_PROXY`|  `ENV NO_PROXY "*.test.example.com,.example2.com"`|  `--env NO_PROXY "*.test.example.com,.example2.com"`