[原文地址](https://www.nginx.com/resources/admin-guide/nginx-https-upstreams/)

#1. 先决条件 Prerequisites
- Nginx 或 Nginx Plus。
- 一台代理服务器或一组 upstream 服务器。
- SSL 证书和私钥。
#2. 获取 SSL 的服务器证书
可以从可信的证书颁发机构（certificate authority，CA）购买证书，或通过 OpenSSL 库自己创建证书。服务器证书和私钥应该被放到每一台 upstream 服务器上。
#3. 获取 SSL 的客户端证书
Nginx 通过使用 SSL 客户端证书向 upstream 服务器标识自己。客户端证书必须由可信的 CA 签名，并且和相对应的私钥一起放置在 Nginx 上。
还需要配置 upstream 服务器来为所有的 SSL 入口连接获取客户端证书，并且信任颁发 Nginx 客户端证书的CA。然后，当 Nginx 连接到 upstream 服务器时，会一并提供客户端证书，upstream 服务器会接受这个证书。

#4. 配置 Nginx
首先将连接到 upstream group 的 URL 改为支持 SSL 连接。在 Nginx 配置文件中，通过 [`proxy_pass`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) 指令为代理服务器或一个 upstream group 指定 https 协议：
```
location /upstream {
    proxy_pass https://backend.example.com;
}
```
通过 [`proxy_ssl_certificate`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ssl_certificate)  和 [`proxy_ssl_certificate_key`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ssl_certificate_key)  指令为每一台需要认证 Nginx 的 upstream server 添加客户端证书和私钥：
```
location /upstream {
    proxy_pass                https://backend.example.com;
    proxy_ssl_certificate     /etc/nginx/client.pem;
    proxy_ssl_certificate_key /etc/nginx/client.key
}
```
如果为一个 upstream 或自己的 CA 使用自己签名的证书，需要包含 [`proxy_ssl_trusted_certificate`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ssl_trusted_certificate)  指令，其指定的证书文件必须是 PEM 格式。还可以通过 [`proxy_ssl_verify`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ssl_verify)  指令和 [`proxy_ssl_verfiy_depth`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ssl_verify_depth)  指令检查安全证书的有效性：
```
location /upstream {
    ...
    proxy_ssl_trusted_certificate /etc/nginx/trusted_ca_cert.crt;
    proxy_ssl_verify       on;
    proxy_ssl_verify_depth 2;
    ...
}
```
每个新建立的 SSL 连接都需要一次客户端和服务器之间完整的 SSL 握手流程，这十分消耗 CPU。通过 [`proxy_ssl_session_reuse`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ssl_session_reuse)  指令可以让 Nginx 代理提前协商好连接参数并使用所谓的缩略握手（abbreviated handshake）：
```
location /upstream {
    ...
    proxy_ssl_session_reuse on;
    ...
}
```
还可以指定使用哪个 SSL 协议，哪个 cipher：
```
location /upstream {
        ...
        proxy_ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        proxy_ssl_ciphers   HIGH:!aNULL:!MD5;
}
```
#5. 配置 Upstream 服务器
每台 upstream 服务器都应该被配置为接受 HTTPS 连接，通过[`ssl_certificate`](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate)  指令和 [`ssl_certificate_key`](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate_key)  指令指定服务器证书和私钥：
```
server {
    listen              443 ssl;
    server_name         backend1.example.com;

    ssl_certificate     /etc/ssl/certs/server.crt;
    ssl_certificate_key /etc/ssl/certs/server.key;
    ...
    location /yourapp {
        proxy_pass http://url_to_app.com;
        ...
    }
}
```
通过[`ssl_client_certificate`](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_client_certificate)  指令指定服务器证书和私钥：
```
server {
    ...
    ssl_client_certificate /etc/ssl/certs/ca.crt;
    ssl_verify_client      off;
    ...
}
```
#6. 完整示例
```
http {
    ...
    upstream backend.example.com {
        server backend1.example.com:443;
        server backend2.example.com:443;
   }

    server {
        listen      80;
        server_name www.example.com;
        ...

        location /upstream {
            proxy_pass          https://backend.example.com;
            proxy_ssl_certificate         /etc/nginx/client.pem;
            proxy_ssl_certificate_key     /etc/nginx/client.key
            proxy_ssl_protocols           TLSv1 TLSv1.1 TLSv1.2;
            proxy_ssl_ciphers             HIGH:!aNULL:!MD5;
            proxy_ssl_trusted_certificate /etc/nginx/trusted_ca_cert.crt;

            proxy_ssl_verify        on;
            proxy_ssl_verify_depth  2;
            proxy_ssl_session_reuse on;
        }
    }

    server {
        listen      443 ssl;
        server_name backend1.example.com;

        ssl_certificate        /etc/ssl/certs/server.crt;
        ssl_certificate_key    /etc/ssl/certs/server.key;
        ssl_client_certificate /etc/ssl/certs/ca.crt;
        ssl_verify_client      off;

        location /yourapp {
            proxy_pass http://url_to_app.com;
        ...
        }

    server {
        listen      443 ssl;
        server_name backend2.example.com;

        ssl_certificate        /etc/ssl/certs/server.crt;
        ssl_certificate_key    /etc/ssl/certs/server.key;
        ssl_client_certificate /etc/ssl/certs/ca.crt;
        ssl_verify_client      off;

        location /yourapp {
            proxy_pass http://url_to_app.com;
        ...
        }
    }
}
```
这个例子中，`proxy_pass` 指令的 https 参数要求从 Nginx 转发到 upstream 服务器的流量需要加密。
当安全连接首次从 Nginx 转发到 upstream 服务器时，需要完整的握手流程。`proxy_ssl_certificate` 指令指明 upstream 服务器所需的 PEM 格式的证书位置。`proxy_ssl_certificate_key` 指令指明 upstream 服务器所需的私钥位置。`proxy_ssl_protocols` 指令和 `proxy_ssl_ciphers` 指令指明协议和 cipher。
当 Nginx 再次转发连接到 upstream 服务器时，因为 `proxy_ssl_session_reuse` 指令，session 参数会被再次使用，从而可以更快的建立安全连接。
`proxy_ssl_trusted_certificate` 指令指定受信任的 CA 证书所在的文件名，这个证书可用于验证 upstream 中的证书。`proxy_ssl_verify_depth` 指令指定检查证书链中的两个证书，`proxy_ssl_verify` 指令验证证书的有效性。