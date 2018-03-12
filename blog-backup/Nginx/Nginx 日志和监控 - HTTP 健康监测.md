[原文地址](https://www.nginx.com/resources/admin-guide/http-health-check/)

如何在 Nginx 中配置和使用 HTTP 健康监测。
#1. 概述
Nginx 和 Nginx Plus 可以持续的测试 upstream 服务器，避免使用发生故障的服务器，并将恢复的服务器平滑的添加到负载平衡组中。
#2. 先决条件 Prerequisites
- Nginx 开源版本和 Plus 版本都支持被动健康监测（passive health checks）。
- NGINX Plus 支持被动和主动健康检查以及监控仪表板。
- HTTP [upstream 服务器](https://www.nginx.com/admin-guide/load-balancer) 的一个负载平衡组。
#3. 被动健康监测
对于被动健康监测，一旦传输开始，Nginx 就会监控这个传输，并且会尝试恢复失败的连接。如果传输一直无法恢复，Nginx 会把服务器标记为不可用，并且在服务器被标记为可用之前向这台服务器停止发送请求。

在 upstream 块中使用 [server](https://nginx.org/en/docs/http/ngx_http_upstream_module.html?&_ga=2.255771392.904408446.1519087829-413045118.1519087829#server) 指令指定服务器时，可以通过参数描述一台 upstream 服务器在什么情况下会被标记为不可用（The conditions under which an upstream server is marked unavailable are defined for each upstream server with the parameters to the server directive in the upstream block:）：

- [fail_timeout](https://nginx.org/en/docs/http/ngx_http_upstream_module.html?&_ga=2.255771392.904408446.1519087829-413045118.1519087829#fail_timeout) – 时间段，有两个用途：在这个时间段内连接失败指定次数后，服务器被标记为不可用；服务器被标记为不可用时，不可用的时间段。默认是 10 秒。
- [max_fails](https://nginx.org/en/docs/http/ngx_http_upstream_module.html?&_ga=2.255771392.904408446.1519087829-413045118.1519087829#max_fails) – 在 `fail_timeout` 指定的时间段内，连接失败的次数到达这个值后，服务器被标记为不可用。默认是 1 次。

下面的例子中，如果 Nginx 在 30 秒内有 3 个请求发送失败或没有接收到响应，则标记服务器为不可用：
```
upstream backend {
    server backend1.example.com;
    server backend2.example.com max_fails=3 fail_timeout=30s;
}
```
#4. 服务器慢启动
服务器慢启动功能可防止最近恢复的服务器被连接淹没，这可能会超时并导致服务器再次被标记为宕机。 
NGINX Plus 中，慢启动功能可以让一台 upstream 服务器在恢复或可用时，权重从零开始逐步增加到正常值。可以通过在 [server](https://nginx.org/en/docs/http/ngx_http_upstream_module.html?&_ga=2.134358009.904408446.1519087829-413045118.1519087829#server) 指令中使用 [slow_start](https://nginx.org/en/docs/http/ngx_http_upstream_module.html?&_ga=2.255771392.904408446.1519087829-413045118.1519087829#slow_start) 参数实现：
```
upstream backend {
    server backend1.example.com slow_start=30s;
    server backend2.example.com;
    server 192.0.0.1 backup;
}
```
slow_start 参数后面的值用来设定慢启动过程的时间。
注意，如果只有一台服务器，[`max_fails`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html?&_ga=2.87485200.904408446.1519087829-413045118.1519087829#max_fails)，[`fail_timeout`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html?&_ga=2.87485200.904408446.1519087829-413045118.1519087829#fail_timeout) 和 [`slow_start`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html?&_ga=2.100632342.904408446.1519087829-413045118.1519087829#slow_start) 参数都会被忽略，这台服务器永远都不会被认为不可用。
#5. 主动健康监测
Nginx 可以通过主动向每台服务器发送特殊的健康检查请求并检查响应来实现对 upstream 服务器的周期性的健康监测。

要开启主动健康监测：
##5.1 使用 `health_check` 指令
在转发请求到 upstream 服务器组（[proxy_pass](https://nginx.org/en/docs/http/ngx_http_proxy_module.html?&_ga=2.33156022.904408446.1519087829-413045118.1519087829#proxy_pass)）的 [location](https://nginx.org/en/docs/http/ngx_http_core_module.html?&_ga=2.28229812.904408446.1519087829-413045118.1519087829#location) 中使用 [`health_check`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html?&_ga=2.33156022.904408446.1519087829-413045118.1519087829#health_check) 指令：
```
server {
    location / {
        proxy_pass http://backend;
        health_check;
    }
}
```
##5.2 指定共享内存区域
通过 [zone](https://nginx.org/en/docs/http/ngx_http_upstream_module.html?&_ga=2.33156022.904408446.1519087829-413045118.1519087829#zone) 指令为 upstream 服务器组指定共享内存区域：
```
http {
    upstream backend {
        zone backend 64k;

        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
        server backend4.example.com;
    }
}
```
上面的配置定义了一个名为 backend 的 upstream 服务器组和一个虚拟服务器，还有一个将所有请求转发到 upstream 服务器组的 location。这还会使用默认参数打开高级健康监测：每 5 秒钟 Nginx 会发送一个请求到 backend 组中的每台服务器。如果发生任何通信错误或超时（或代理服务器返回 2XX 或 3XX 以外的状态码），服务器的健康检查失败，标记为异常服务器，Nginx 在这台服务器通过健康检查之前不会再发送客户端的请求到这台服务器。

zone 指令定义了一个在工作进程之间共享的内存区域，用来存储 upstream 服务器组的配置。这会使得工作进程可以使用相同的一组计数器来追踪从服务器组中的服务器的响应。zone 指令还可以 [动态配置](https://nginx.org/en/docs/http/ngx_http_upstream_conf_module.html?&_ga=2.99732118.904408446.1519087829-413045118.1519087829#upstream_conf)组。

主动健康监测的默认参数可以用 `health_check` 指令的参数覆盖：
```
location / {
    proxy_pass http://backend;
    health_check interval=10 fails=3 passes=2;
}
```
这里的 `interval` 参数把健康检查之间的延时增大为 10 秒（默认 5 秒）。fails 参数意味着服务器的健康检查失败三次后，被标记为不健康（高于默认值 1）。 最后，passes 参数意味着服务器必须再次通过两次连续的检查（而不是默认值 1）。
##5.3 指定请求 URI
使用 uri 参数设置健康检查中的请求要发送到哪个  URI：
```
location / {
    proxy_pass http://backend;
    health_check uri=/some/path;
}
```
指定的 URI 会添加到 upstream 块中的服务器的域名或 IP 地址之后。对于上面示例中的服务器组 backend 中的第一个服务器，健康检查会请求这个 URI：`http://backend1.example.com/some/path`。
##5.4 定义自定义条件
最终，可以定义自定义条件，响应满足这些条件时才认为服务器通过了健康检查。条件定义在 [match](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html?&_ga=2.99732118.904408446.1519087829-413045118.1519087829#match) 块中，通过 `health_check` 指令中的 match 参数关联。
```
http {
    ...

    match server_ok {
        status 200-399;
        body !~ "maintenance mode";
    }


    server {
        ...
        location / {
            proxy_pass http://backend;
            health_check match=server_ok;
        }
    }
}
```
这个例子中，如果响应的状态码在 200 - 399 之间则通过健康检查，报文体不包含字符串维护模式（and its body does not contain the string maintenance mode）。

match 指令使 Nginx Plus 检查状态码、头字段和响应体。通过这个指令可以验证状态是否在指定范围内，响应是否包含一个头字段，头字段或响应体是否匹配一个正则表达式。match 指令可以包含一个 status 条件、一个 body 条件和多个 header 条件。只有在响应满足 match 块中为这台服务器定义的所有条件时，才认为通过健康检查。

例如，下面的 match 指令匹配响应码为 200 的状态码，头字段中的 Content-Type 必须为“text/html”，并且报文体中必须包含“Welcome to nginx!”：
```
match welcome {
    status 200;
    header Content-Type = text/html;
    body ~ "Welcome to nginx!";
}
```
下面的例子使用感叹号 `!` 来定义要通过健康检查，响应不能匹配哪些条件。当状态码不是 301，302，303，307 且响应头字段不包含 Refresh 时，通过健康检查：
```
match not_redirect {
    status ! 301-303 307;
    header ! Refresh;
}
```
健康检查也可以用于非 HTTP 协议，例如 [FastCGI](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html?_ga=2.20898992.904408446.1519087829-413045118.1519087829)，[memcached](https://nginx.org/en/docs/http/ngx_http_scgi_module.html?_ga=2.20898992.904408446.1519087829-413045118.1519087829)，[SCGI](https://nginx.org/en/docs/http/ngx_http_scgi_module.html?_ga=2.20898992.904408446.1519087829-413045118.1519087829)，[uwsgi](https://nginx.org/en/docs/http/ngx_http_uwsgi_module.html?_ga=2.20898992.904408446.1519087829-413045118.1519087829)，当然也包括 [TCP 和 UDP](http://nginx.org/en/docs/stream/ngx_stream_upstream_hc_module.html?&_ga=2.20898992.904408446.1519087829-413045118.1519087829#health_check)。