[原文地址](https://www.nginx.com/resources/admin-guide/restricting-access/)

本节描述了如何设置连接的最大请求数，或从服务器下载内容的最大速率。

**所有的限制，都需要指定关键字（比如 IP 地址），用这个关键字作为计数的标准。**
#1. 概述
使用 Nginx 或 Nginx Plus 时，可以限制：

- 每个键值的连接数量（例如，每个 IP 地址）。
- 每个键值的请求速率（每秒或每分钟允许处理的请求个数）。
- 一个连接的下载速度。

注意，IP 地址可以通过 NAT 设备共享（一个局域网中可以有多台设备，但可能只有一个公网 IP），应该谨慎使用对 IP 地址的限制。
#2. 限制连接数
要限制连接数，首先，使用 [`limit_conn_zone`](https://nginx.org/en/docs/http/ngx_http_limit_conn_module.html#limit_conn_zone) 指令来定义关键字，并设置共享内存区域的参数（工作进程会使用这个区域来共享键值的计数器）。作为第一个参数，指定解析为关键字的表达式。在第二个参数区域中，指定区域的名称及大小。
```
limit_conn_zone $binary_remote_addr zone=addr:10m;
```
第二，使用 `limit_conn` 指令来对 location、server 或整个 http 上下文使用限制。将共享内存区域的名字作为第一个参数，每个关键字允许的连接数作为第二个参数：
```
location /download/ {
    limit_conn addr 1;
}
```
这个例子中，连接数量限制在一个 IP 地址的基础上，因为关键字是 `$binary_remote_addr` 变量。指定服务器的连接数量可以通过使用 `$server_name` 变量来限制：
```
http {
    limit_conn_zone $server_name zone=servers:10m;

    server {
        limit_conn servers 1000;
    }
}
```
#3. 限制请求速率
要限制请求速率，首先，通过使用 [`limit_req_zone`](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html#limit_req_zone) 指令设置关键字和放计数器的共享内存区域。
```
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
```
这里的关键字和 `limit_conn_zone` 指令的关键字一样。速率参数可以设置为每秒请求数（r/s）或每分钟请求数（r/m）。30 r/m 表示每分钟可以处理 30 个请求。

一旦定义好了共享内存区域，使用 server 或 location 中的 [`limit_req`](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html#limit_req) 指令来限制请求速率：
```
location /search/ {
    limit_req zone=one burst=5;
}
```
上面例子中，Nginx 在这个 location 中每秒钟处理的请求数不多于 1。如果请求速率高于每秒一个，则处理不过来的请求会放入队列中延迟处理，使整体速率不会超过指定值。`burst` 参数设置等待处理的最大请求个数。对于超过这个限制数量的请求，Nginx 会返回 503 错误。

如果在并发期间不希望有延迟，请添加 `nodelay` 参数：
```
limit_req zone=one burst=5 nodelay;
```
#4. 限制带宽（Bandwidth）
使用 [`limit_rate`](https://nginx.org/en/docs/http/ngx_http_core_module.html#limit_rate) 指令可以限制每个连接的带宽：
```
location /download/ {
    limit_rate 50k;
}
```
上面的这个设置会将客户端的一个独立连接的下载速率限制在 50 KB 每秒。然而客户端可以同时打开多个连接。因此，如果目标是阻止下载速度大于指定值，则连接数量也应该受到限制。 例如，每个IP地址一个连接（使用上面指定的共享内存区域）：
```
location /download/ {
    limit_conn addr 1;
    limit_rate 50k;
}
```
使用 [`limit_rate_after`](https://nginx.org/en/docs/http/ngx_http_core_module.html#limit_rate_after) 指令，可以**只在客户端下载了一定数量的数据之后才限制速率**。允许客户快速下载一定数量的数据（例如文件头 - 电影索引）并限制下载其余数据的速率（以使用户观看电影而不是下载）是合理的。
```
limit_rate_after 500k;
limit_rate 20k;
```
下面的例子是限制连接速率和带宽的综合示例。每个客户端 IP 地址的最大连接数量是 5，满足大多数情况（现代浏览器通常最多一次打开 3 个连接）。与此同时，/download/ 对应的 location 只允许 1 个连接：
```
http {
    limit_conn_zone $binary_remote_addr zone=addr:10m

    server {
        root /www/data;
        limit_conn addr 5;

        location / {
        }

        location /download/ {
            limit_conn addr 1;
            limit_rate 1m;
            limit_rate 50k;
        }
    }
}
```