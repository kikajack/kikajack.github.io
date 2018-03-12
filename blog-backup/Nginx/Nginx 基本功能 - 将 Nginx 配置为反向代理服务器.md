[原文地址](https://www.nginx.com/resources/admin-guide/reverse-proxy/)

反向代理服务器通常用于负载均衡。

配置为代理服务器后，Nginx 可以把不同协议的请求转发给其他服务器处理，可以在转发请求时修改客户端请求头，可以配置从处理请求的服务器响应的缓存。
#1. 概述
代理通常用于在多台服务器之间分配负载，无缝地显示来自不同网站的内容，或根据协议将处理请求传递给不同的应用程序服务器。
#2. 将请求传递给代理服务器
当 Nginx 代理请求时，它将请求发送到指定的代理服务器，获取响应并将其发送回客户端。可以根据指定的协议将代理的请求发送给 HTTP 服务器（另一台 Nginx 服务器或非 Nginx 服务器）或非 HTTP 服务器（可以运行 PHP 或 Python 应用）。Nginx 支持的协议包括 FastCGI、uwsgi、SCGI 和 memcached。
##2.1 把请求发送到 HTTP 代理服务器
###2.1.1 proxy_pass 指令
`proxy_pass` 指令可以把请求发送到 HTTP 代理服务器，`proxy_pass` 指令在 location 块中：
```
location /some/path/ {
    proxy_pass http://www.example.com/link/;
}
```
下面这个配置会把所有发送到这个 location 的请求发送到用 `proxy_pass` 指定地址的代理服务器，地址可以是域名或 IP 地址，可以包含端口号：
```
location ~ \.php {
    proxy_pass http://127.0.0.1:8000;
}
```
###2.1.2 URI 替换
如果代理服务器地址中包含 URI，比如下面例子中的 /link/，请求中的 URI 部分会被替换为匹配到的 location 参数。比如：请求中如果带 URI：/some/path/page.html，会被发送到这个地址：http://www.example.com/link/page.html。
如果代理服务器地址中不包含 URI，或者无法判断要替换的 URI 部分，则传递完整的请求URI（可能已修改）。
##2.2 把请求发送到非 HTTP 代理服务器
`**_pass` 指令可以把请求发送到非 HTTP 代理服务器

- [fastcgi_pass](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_pass)：将请求发送给 FastCGI 服务器
- [uwsgi_pass](http://nginx.org/en/docs/http/ngx_http_uwsgi_module.html#uwsgi_pass)：将请求发送给 uwsgi 服务器
- [scgi_pass](http://nginx.org/en/docs/http/ngx_http_scgi_module.html#scgi_pass)：将请求发送给 SCGI 服务器
- [memcached_pass](http://nginx.org/en/docs/http/ngx_http_memcached_module.html#memcached_pass)：将请求发送给 memcached 服务器

注意上面几个命令中，指定地址的规则不一样。有的服务器需要附加参数。

proxy_pass 指令也可以指向一组指定的服务器。 在这种情况下，根据指定的方法在组中的服务器之间分配请求。
#3. 传递请求头
Nginx 默认情况下在代理请求中重新定义两个头部字段：Host 和 Connection，并且删除请求头中值为空的字段。Host 字段被设置到 `$proxy_host` 变量，Connection 字段被设置为 close。
[`proxy_set_header`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header) 指令可以改变这些设置，也可以定期其他的头部字段。`proxy_set_header` 指令可以在 [location](http://nginx.org/en/docs/http/ngx_http_core_module.html#location) 块或更高级别的块中使用，也可以在 [server](http://nginx.org/en/docs/http/ngx_http_core_module.html#server) 上下文或 [http](http://nginx.org/en/docs/http/ngx_http_core_module.html#http) 块中使用。
```
location /some/path/ {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_pass http://localhost:8000;
}
```
上面的配置中吧 Host 字段设置到 [`$host`](http://nginx.org/en/docs/http/ngx_http_core_module.html#variables) 变量。

把头部字段设置为空字符串，就可以防止其被传递到代理服务器：
```
location /some/path/ {
    proxy_set_header Accept-Encoding "";
    proxy_pass http://localhost:8000;
}
```
#4. 配置缓冲区
Nginx 默认情况下缓存代理服务器的响应。响应会存储在内部缓冲区，直到接收完毕，才会发送到客户端。缓冲有助于优化慢速客户端的性能，如果响应从 Nginx 同步传递到客户端，可能会浪费代理服务器的时间。但是，启用缓冲时，Nginx 允许代理服务器快速处理响应，但是 Nginx 会把响应存储一段时间，客户端需要等待这段时间后才能开始下载。

[`proxy_buffering`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_buffering) 指令可以使能或禁止缓存。
[`proxy_buffers`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_buffers) 指令可以控制缓存区的大小，分配给每个请求的缓存区个数。
从代理服务器返回响应的第一部分会存储在一个独立的缓冲区中，这个缓冲区的大小由 [`proxy_buffer_size`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_buffer_size) 指令设置。这部分通常包含一个相对较小的响应头，并且可以比其余响应的缓冲区更小。

在以下示例中，缓冲区的默认数量增加，响应第一部分的缓冲区大小比默认值小：
```
location /some/path/ {
    proxy_buffers 16 4k;
    proxy_buffer_size 2k;
    proxy_pass http://localhost:8000;
}
```
如果关闭缓冲区，代理服务器返回的响应会由 Nginx 同步发送到客户端，响应更及时。可以通过 `proxy_buffering` 指令来关闭指定 location 中的缓存功能：
```
location /some/path/ {
    proxy_buffering off;
    proxy_pass http://localhost:8000;
}
```
在这种情况下，Nginx 仅使用由 `proxy_buffer_size` 配置的缓冲区来存储响应的当前部分。
#5. 选择传出IP地址 Choosing an Outgoing IP Address
如果代理服务器有好几个网卡，有时需要选择指定的源 IP 地址来连接到被代理的服务器或上游。当 Nginx 后面的服务器被配置为只能接受来自特定 IP 或 IP 范围的连接时非常有用。

`proxy_bind` 指令用于选择 IP 地址：
```
location /app1/ {
    proxy_bind 127.0.0.1;
    proxy_pass http://example.com/app1/;
}

location /app2/ {
    proxy_bind 127.0.0.2;
    proxy_pass http://example.com/app2/;
}
```
IP 地址也可以用变量来设置。例如，`$server_addr` 变量把请求的 IP 地址传递到 `proxy_bind` 指令：
```
location /app3/ {
    proxy_bind $server_addr;
    proxy_pass http://example.com/app3/;
}
```