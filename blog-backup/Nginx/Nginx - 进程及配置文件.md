#1. 进程
Nginx 服务启动后，会有一个主进程（master process），一个或多个工作进程（worker processes）。如果开启了 [缓存](https://www.nginx.com/resources/admin-guide/content-caching/)，缓存加载和缓存管理进程也会在 Nginx 服务启动时运行。
主进程用于读取并评估配置文件，维护工作进程的运行。
工作进程用于处理请求。工作进程的数量在 nginx.conf 配置文件中指定，可以设置为固定值或根据可用的 CPU 核心数量来动态调整（参考 [worker_processes](http://nginx.org/en/docs/ngx_core_module.html#worker_processes)）。
#2. 配置文件
[参考 Nginx 官网](https://www.nginx.com/resources/admin-guide/configuration-files/)

配置文件改完后，需要重启 Nginx 进程或发送 reload 信号给 Nginx 进程。
##2.1 配置文件名称和位置
Nginx 配置文件名称通常是 nginx.conf，位置是 /etc/nginx 或 /usr/local/nginx/conf。

##2.2 简单指令和块指令
配置文件由指令组成。指令分为简单指令和块指令（block directives）。简单指令由名称和参数组成，以空格分隔，并以分号（;）结束。块指令不以分号结尾，而是用一系列由大括号（{}）包围的附加指令来结束。如果块指令的大括号内可以有其他指令，这个块指令就被称为一个上下文（例如： [events](http://nginx.org/en/docs/ngx_core_module.html#events)， [http](http://nginx.org/en/docs/http/ngx_http_core_module.html#http)， [server](http://nginx.org/en/docs/http/ngx_http_core_module.html#server) 和 [location](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)）。

- 简单指令：
```
user             nobody;
error_log        logs/error.log notice;
worker_processes 1;
```
- 块指令：
```
location / {
    root /data/www;
}
```
##2.3 上下文
配置文件中，如果指令在任何上下文之外，则被认为是在主上下文 [main context](http://nginx.org/en/docs/ngx_core_module.html) 中。events 和 http 指令在主上下文 main context 中，而 server 指令在 http 中，location 指令在 server 中。
##2.4 注释行
配置文件中，所有注释行用井号开头（`#`）。
##2.5 配置文件的可维护性
为了让配置文件容易维护，可以根据功能将配置文件分割为不同的文件，放在 /etc/nginx/conf.d 目录中，然后在主配置文件 nginx.conf  中通过 include 指令引入这些文件。
```
include conf.d/http;
include conf.d/stream;
include conf.d/exchange-enhanced;
```
##2.6 顶级指令（区分流量类型）
配置文件中有几个用于区分流量类型的顶级指令（称为上下文），不能写在其他命令下：

- events – 处理一般连接
- http – 处理 HTTP 流量
- mail – 处理 Mail 流量
- stream – 处理 TCP 流量

除了这几个区分流量类型的指令，其他的指令位于主上下文 [main context](http://nginx.org/en/docs/ngx_core_module.html) 中。
在这几个区分流量类型的指令中，可以包含一个或多个服务器上下文（server context）来定义虚拟服务器。在服务器上下文中可以根据不同的流量类型使用不同的指令。
对于 HTTP 上下文中的 HTTP 流量，每个服务器上下文中的指令可以处理指定域名或 IP 地址的请求。服务器上下文中可以定义一个或多个 location 上下文，根据不同的 URI 来做不同的处理。
对于 mail 和 stream 上下文中的 mail 和 TCP 流量，每个服务器上下文中的指令可以处理指定 TCP 端口或 UNIX socket 套接字的请求。
使用上下文的示例：
```
user nobody; # 主上下文中的指令

events {
    # 关于 connection processing 的配置
}

http {
    # 配置针对 HTTP 流量的虚拟主机
    server {
        # 虚拟主机1的 HTTP 配置
        location /one {
            # 配置 URIs 中包含 '/one' 的处理方式
        }
        location /two {
            # 配置 URIs 中包含 '/two' 的处理方式
        }
    }

    server {
        # 虚拟主机2的 HTTP 配置
    }
}

stream {
    # 配置针对 TCP 流量的虚拟主机
    server {
        # 配置 TCP 虚拟主机1
    }
}
```
对于大多数指令，子上下文会继承父上下文中的指令值。上下文继承可以参考指令 [proxy_set_header](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header)。