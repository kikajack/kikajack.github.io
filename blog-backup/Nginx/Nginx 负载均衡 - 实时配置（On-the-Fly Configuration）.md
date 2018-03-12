[原文地址](https://www.nginx.com/resources/admin-guide/load-balancing-api/)

#1. 概述
使用 Nginx Plus 时，服务器组中的 upstream 服务器的配置可以通过 Nginx Plus 的 REST API 接口实时修改。可以发送 API 命令来查看所有服务器或服务器组中指定的服务器，修改指定服务器的参数，增删服务器。
注意：在 Nginx Plus 之前的版本中，实时配置是通过 `upstream_conf` 处理程序来实现的，已废弃。
#2. 先决条件 Prerequisites
使用动态配置需要满足的环境：

1. NGINX Plus R13。
2. 创建了包含服务器的 upstream 服务器组。更多信息可以查看 [HTTP 负载均衡](https://www.nginx.com/resources/admin-guide/load-balancer/) 和 [TCP/UDP 负载均衡](https://www.nginx.com/resources/admin-guide/tcp-load-balancing/)。
3. upstream 服务器组驻留在共享内存区域中。更多信息可以查看 [多进程之间共享数据](https://www.nginx.com/resources/admin-guide/load-balancer/#zone)。
#3. 在配置文件中开启 API
为了开启 API，需要在 Nginx 配置文件中为 API 请求创建一个独立的 location，并且在这个 location 中指定 [api](http://nginx.org/en/docs/http/ngx_http_api_module.html#api) 指令：
```
server {
    ...
    location /api {
        api write=on;
    }
}
```
强烈建议，对这个 location [限制访问](https://nginx.com/resources/admin-guide/restricting-access/#restrict)，例如，只允许来自 127.0.0.1 的访问：
```
server {
    location /api {
        api write=on;
        allow 127.0.0.1;
        deny  all;
    }
}
```
upstream 服务器组中使用 API location 的例子：
```
http {
    # ...
    # Configuration of the server group
    upstream appservers {
        zone appservers 64k;

        server appserv1.example.com      weight=5;
        server appserv2.example.com:8080 fail_timeout=5s;


        server reserve1.example.com:8080 backup;
        server reserve2.example.com:8080 backup;
    }


    server {
        # Location that proxies requests to the group
        location / {
            proxy_pass http://appservers;
            health_check;
        }


        # Location for configuration requests
        location /api {
            api write=on;
            allow 127.0.0.1;
            deny  all;
        }
    }
}
```
这个例子中，第二个 location 只允许来自 127.0.0.1 这个 IP 地址的访问。来自其他 IP 地址的请求一律拒绝。
#4. 使用 API
Nginx Plus 的 REST API 支持下面几种 HTTP 方法：

- GET – 获取 upstream 服务器组或其中一台服务器的信息。
- POST – 向 upstream 服务器组添加一台服务器。
- PATCH – 为一台指定的服务器更新参数。
- DELETE- 从服务器组删除一台服务器。

[Nginx 参考手册](http://nginx.org/en/docs/http/ngx_http_api_module.html) 中详细描述了端点和方法（endpoints and methods）。此外，该 API 还提供了一个 Swagger 规范，可用于探索 API 并了解每个资源的功能。Swagger 文档与 NGINX Plus 捆绑在一起，可以通过 `http://nginxhost/swagger-ui/` 进行访问。
要将配置命令传递给 Nginx，可以通过任何方式发送 API 命令，例如 curl。 所有的请求体和响应都是 JSON 格式。 API URL 应包含 API 版本，当前版本为1。

例如，要向服务器组添加一个新服务器，可以发送下面的 curl 命令：
```
curl -X POST -d '{
   "server": "10.0.0.1:8089",
   "weight": 4,
   "max_conns": 0,
   "max_fails": 0,
   "fail_timeout": "10s",
   "slow_start": "10s",
   "backup": true,
   "down": true
 }' -s http://127.0.0.1/api/1/http/upstreams/appservers/servers
```
从服务器组删除一台服务器：
```
curl -X DELETE -s http://127.0.0.1/api/1/http/upstreams/appservers/servers/0
```
编辑指定服务器的参数：
```
curl -X PATCH -d '{ "down": true }' -s http://127.0.0.1/api/1/http/upstreams/appservers/servers/0
```
#5. 动态示例
可以在只读模式下在线尝试 NGINX Plus API 接口：http://demo.nginx.com/swagger-ui/。
#6. 配置动态配置的持久性
上例中的配置允许将实时更改存储在共享内存中。 重新加载 NGINX Plus 的配置文件时，这些更改将被丢弃。
要使这些更改在配置重新加载期间保持不变，需要将 upstream 服务器的列表从 upstream block 移动到一个专用文件，以保持 upstream 服务器的状态。文件的路径通过 [`state`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#state) 指令设置。 Linux发行版的推荐路径是 /var/lib/nginx/state/，FreeBSD发行版的路径是 /var/db/nginx/state/：
```
http {
    # ...
    upstream appservers {
        zone appservers 64k;
        state /var/lib/nginx/state/appservers.conf;

        # All these servers should be moved to the file using the upstream_conf API:
        # server appserv1.example.com      weight=5;
        # server appserv2.example.com:8080 fail_timeout=5s;
        # server reserve1.example.com:8080 backup;
        # server reserve2.example.com:8080 backup;
    }
}
```
注意，**这个文件只能通过来自 API 接口的配置命令来修改，而不应该直接修改文件。**