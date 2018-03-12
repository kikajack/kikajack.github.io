本文介绍了如何配置 NGINX 和 NGINX Plus 以接受 PROXY 协议，**将负载平衡器或代理的 IP 地址重写为在 PROXY 协议头中接收到的 IP 地址，配置客户端 IP 地址的简单日志记录，启用 NGINX 和 TCP upstream 服务器之间的 PROXY 协议**。
#1. 概述
PROXY 协议允许 Nginx 和 Nginx Plus 接受来自代理服务器和负载平衡器的客户端连接信息，比如 HAproxy 和 Amazon Elastic Load Balancer (ELB)
通过 PROXY 协议，Nginx 可以从 HTTP，SSL，HTTP / 2，SPDY，WebSocket 和 TCP 中获取到源 IP 地址。获取到客户端的源 IP 地址，可以在为网页指定语言、设置 IP 黑名单或只是简单的日志和统计分析。
通过 PROXY 协议传输的数据是客户端的 IP 地址、代理服务器的 IP 地址和所有的端口号。Nginx 可以通过这个数据使用几种不同方法获取到客户端的源 IP 地址：

- `$proxy_protocol_addr` 和 `$proxy_protocol_addr_port` 变量保存客户端源 IP 地址和端口。`$remote_addr` 和 `$remote_port` 变量保存负载平衡服务器的 IP 和端口。
- 使用 realip 模块将 `$remote_addr` 和 `$remote_port` 变量从负载平衡器的IP和端口重写为原始客户端 IP 地址和端口。 `$realip_remote_addr` 和 `$realip_remote_port` 端口变量将保留负载均衡器的地址和端口，`$proxy_protocol_addr` 和 `$proxy_protocol_port` 变量将始终保留原始客户端 IP 和端口。
#2. 先决条件 Prerequisites
- Nginx Plus R3 或 Nginx 开源版本1.5.12 才能接受 HTTP 的 PROXY 协议。
- Nginx Plus R11 或 Nginx 开源版本 1.11.4 才能接受 TCP 的 PROXY 协议。
- Nginx Plus R7 或 Nginx 开源版本 1.9.13 才能支持 TCP 客户端的 PROXY 协议。
- Nginx 开源版本可能会需要默认情况下不会包含的 [`ngx_http_realip_module`](https://nginx.org/en/docs/http/ngx_http_realip_module.html) 和 [`ngx_stream_realip_module`](https://nginx.org/en/docs/stream/ngx_stream_realip_module.html) 模块，详情可以查看 [安装 Nginx 开源版本](https://www.nginx.com/resources/admin-guide/installing-nginx-open-source/)。对于 Nginx Plus，不需要额外的安装步骤。
#3. 配置 NGINX 以接受 PROXY 协议
要配置 Nginx 接受 PROXY 协议头，请将 `proxy_protocol` 参数添加到 [http](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen) 或 [stream](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#listen) 的 `listen` 指令中：
```
http {
    ...
        server {
        listen 80   proxy_protocol;
        listen 443  ssl proxy_protocol;
        ...
    }
}
```
对 TCP stream 流的配置：
```
stream {
    ...
        server {
        listen 12345   proxy_protocol;
        ...
    }
}
```
现在可以使用表示客户端 IP 地址和端口的 `$proxy_protocol_addr` 和 `$proxy_protocol_port` 变量，另外还可以配置 [HTTP realip](https://nginx.org/en/docs/http/ngx_http_realip_module.html) 或 [stream realip](https://nginx.org/en/docs/stream/ngx_stream_realip_module.html) 模块以将负载均衡器的 IP 替换为 `$remote_addr` 和 `$remote_port` 变量中的客户端 IP。
#4. 将负载均衡器的 IP 地址改为客户端 IP 地址
##4.1 相关变量及模块
可以将负载均衡器的 IP 地址改为从 PROXY 协议接收到的客户端 IP 地址。这可以通过 [`ngx_http_realip_module`](https://nginx.org/en/docs/http/ngx_http_realip_module.html) 和 [`ngx_stream_realip_module`](https://nginx.org/en/docs/stream/stream.html) 模块来实现。通过这些模块，`$remote_addr` 和 `$remote_port` 变量保存客户端源 IP 地址和端口。`$realip_remote_addr` 和 `$realip_remote_port` 变量保存负载平衡服务器的 IP 和端口。
##4.2 把 IP 地址从负载均衡器的变为客户端的：
####4.2.1 确保 Nginx 已经被配置为可以接受 PROXY 协议头。配置过程 [参考这里](https://www.nginx.com/resources/admin-guide/proxy-protocol/#listen)
####4.2.2 确保 Nginx 包含了 [HTTP realip](https://nginx.org/en/docs/http/ngx_http_realip_module.html) 和 [stream realip](https://nginx.org/en/docs/stream/ngx_stream_realip_module.html)  模块：
```
nginx -V 2>&1 | grep -- 'http_realip_module'
nginx -V 2>&1 | grep -- 'stream_realip_module'
```
如果没有这些模块，可以编译包含这些模块的 Nginx，[参考这里](https://www.nginx.com/resources/admin-guide/installing-nginx-open-source/)。
####4.2.3 使用 [http](https://nginx.org/en/docs/http/ngx_http_realip_module.html#set_real_ip_from) 或 [stream](https://nginx.org/en/docs/stream/ngx_stream_realip_module.html#set_real_ip_from) 的 `set_real_ip_from` 指令指定 TCP 代理或负载均衡器的 IP 地址或 CIDR 地址范围：
```
server {
    ...
    set_real_ip_from 192.168.1.0/24;
    ...
}
```
####4.2.4 对于 http {}，把负载均衡器的 IP 地址改为从 PROXY 协议头接收到的客户端 IP 地址。在 [`real_ip_header`](https://nginx.org/en/docs/http/ngx_http_realip_module.html#real_ip_header) 指令中，指定 `proxy_protocol` 参数：
```
server {
    ...
    real_ip_header proxy_protocol;
}
```
#5. 记录原始IP地址
当知道客户端的原始 IP 地址时，可以配置正确的日志记录：
##5.1 对于 http {}
通过 [`proxy_set_header`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header) 指令和 [`$proxy_protocol_addr`](https://nginx.org/en/docs/http/ngx_http_core_module.html#var_proxy_protocol_addr) 变量，可以把 Nginx 的客户端 IP 地址发送到 upstream 服务器：
```
proxy_set_header X-Real-IP       $proxy_protocol_addr;
proxy_set_header X-Forwarded-For $proxy_protocol_addr;
```
##5.2 http 或 stream
对于 http 或 stream，可以使用 `log_format` 指令和 [`$proxy_protocol_addr`](https://nginx.org/en/docs/http/ngx_http_core_module.html#var_proxy_protocol_addr) 变量：
###5.2.1 对于 http {} 块：
```
http {
    ...
    log_format combined '$proxy_protocol_addr - $remote_user [$time_local] '
                        '"$request" $status $body_bytes_sent '
                        '"$http_referer" "$http_user_agent"';
}
```
###5.2.2 对于 stream {} 块：
```
stream {
    ...
    log_format basic '$proxy_protocol_addr - $remote_user [$time_local] '
                        '$protocol $status $bytes_sent $bytes_received '
                         '$session_time';
}
```
#6. 用于 TCP 连接到 Upstream 服务器的 PROXY 协议
对于 TCP 数据流，可以开启在 Nginx 和一台上游服务器（upstream server）之间的 PROXY 协议。在 stream{} 上下文中的 server 块中添加 [`proxy_protocol`](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_protocol) 指令即可：
```
stream {
    server {
        listen 12345;
        proxy_pass example.com:12345;
        proxy_protocol on;
    }
}
```
#7. 示例
```
http {
    log_format combined '$proxy_protocol_addr - $remote_user [$time_local] '
                        '"$request" $status $body_bytes_sent '
                        '"$http_referer" "$http_user_agent"';
    ...

    server {
        server_name localhost;

        listen 80   proxy_protocol;
        listen 443  ssl proxy_protocol;

        ssl_certificate      /etc/nginx/ssl/public.example.com.pem;
        ssl_certificate_key  /etc/nginx/ssl/public.example.com.key;

        location /app/ {
            proxy_pass                       http://backend1;
            proxy_set_header Host            $host;
            proxy_set_header X-Real-IP       $proxy_protocol_addr;
            proxy_set_header X-Forwarded-For $proxy_protocol_addr;
        }
    }
}

stream {
    log_format basic '$proxy_protocol_addr - $remote_user [$time_local] '
                     '$protocol $status $bytes_sent $bytes_received '
                     '$session_time';
...
    server {

        listen              12345 ssl proxy_protocol;

        ssl_certificate     /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/cert.key;

        proxy_pass          backend.example.com:12345;
        proxy_protocol      on;
    }
}
```
这个例子假设 Nginx 前面有负载平衡器（例如，亚马逊 ELB）平衡所有传入的 HTTPS 流量。Nginx 接受端口 443 的 HTTPS 流量，端口 12345 的 TCP 流量，并接受 PROXY 协议（http {} 和 stream {} 块中的 listen 指令中的 proxy_protocol 参数）。

Nginx 终止 HTTPS 流量（`ssl_certificate` 和 `ssl_certificate_key` 指令），将解密后的数据发送到包括客户端的 IP 地址和端口（`proxy_set_header` 指令值）的后台服务器（对于http {} 使用 `proxy_pass http://backend1;`，对于 stream {} 使用 `proxy_pass backend.example.com:12345`）。

`log_format` 指令中指定的 `proxy_protocol_addr` 变量也会将客户端 IP 地址传给 http {} 和 stream {} 块中的日志。

此外，TCP 服务器（stream {} 块）发送自己的 PROXY 协议的流量到后端服务器（proxy_protocol 指令设置为 on）。