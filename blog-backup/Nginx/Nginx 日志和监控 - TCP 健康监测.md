[原文地址](https://www.nginx.com/resources/admin-guide/tcp-health-check/)

#1. 概述
Nginx 和Nginx Plus可以不断测试 TCP upstream 服务器，避免使用出现故障的服务器，并且将恢复的服务器逐步地添加到负载平衡组中。
#2. 先决条件 Prerequisites
- 已经配置了 [stream](http://nginx.org/en/docs/stream/ngx_stream_core_module.html?&_ga=2.67096102.904408446.1519087829-413045118.1519087829#stream) 上下文中的 TCP upstream 服务器组：
```
stream {
    ...
    upstream stream_backend {

    server backend1.example.com:12345;
    server backend2.example.com:12345;
    server backend3.example.com:12345;
   }
    ...
}
```
- 已经配置了 server 组中转发 TCP 连接的服务器：
```
stream {
    ...
    server {
        listen     12345;
        proxy_pass stream_backend;
    }
    ...
}
```
#3. 被动 TCP 健康检查
如果一个连接到 upstream 服务器的尝试超时或失败，Nginx 会把服务器标记为 unavailable 并在指定的时间内停止向这台服务器发送请求。通过向 [server](http://nginx.org/en/docs/stream/ngx_stream_upstream_module.html?&_ga=2.101312393.904408446.1519087829-413045118.1519087829#server) 指令中添加下面的参数可以定义 Nginx 在哪些情况下认为服务器不可用：

- [fail_timeout](http://nginx.org/en/docs/stream/ngx_stream_upstream_module.html?&_ga=2.101312393.904408446.1519087829-413045118.1519087829#fail_timeout) – 时间段，有两个用途：在这个时间段内连接失败指定次数后，服务器被标记为不可用；服务器被标记为不可用时，不可用的时间段。默认是 10 秒。
- [max_fails](http://nginx.org/en/docs/stream/ngx_stream_upstream_module.html?&_ga=2.101312393.904408446.1519087829-413045118.1519087829#max_fails) – 在 fail_timeout 指定的时间段内，连接失败的次数到达这个值后，服务器被标记为不可用。默认是 1 次。

如果一个连接在 10 秒的时间内超时或失败至少 1 次，则 Nginx 把这个服务器设为 10 秒内不可用。下面例子设置的是 30 秒内失败或超时至少 2 次才认为不可用：
```
upstream stream_backend {
    server backend1.example.com:12345 weight=5;
    server backend2.example.com:12345 max_fails=2 fail_timeout=30s;
    server backend3.example.com:12346 max_conns=3;
}
```
#4. 主动 TCP 健康检查
##4.1 健康检查的特性
可以配置健康检查用来测试各种故障类型。例如，Nginx Plus 可以持续测试 upstream 服务器的响应能力，并避免使用发生故障的服务器。

Nginx Plus 向每台 upstream 服务器发送特殊的健康检查请求，并检查响应是否满足特定条件。如果一个到服务器的连接不能建立，则健康检查失败，并认为服务器故障。Nginx Plus 不会把客户端连接代理到故障服务器。如果服务器组定义了几个健康检查，任何一个健康检查失败都会使相关的服务器被视为故障。
##4.2 开启健康检查
###4.2.1 指定共享内存区域
zone - 共享内存区域可以在 Nginx 的多个工作进程之间共享计数器和连接的状态信息。通过向 upstream 服务器组中添加 [zone](http://nginx.org/en/docs/stream/ngx_stream_upstream_module.html?&_ga=2.59148965.904408446.1519087829-413045118.1519087829#zone) 指令来指定名字和内存大小：
```
stream {
    ...
    upstream stream_backend {

        zone   stream_backend 64k;
        server backend1.example.com:12345;
        server backend2.example.com:12345;
        server backend3.example.com:12345;
    }
    ...
}
```
###4.2.2 开启健康检查
在将连接代理到 upstream 服务器组的 server 上下文中添加 [`health_check`](http://nginx.org/en/docs/stream/ngx_stream_upstream_hc_module.html?&_ga=2.88663955.904408446.1519087829-413045118.1519087829#health_check) 和 [`health_check_timeout`](http://nginx.org/en/docs/stream/ngx_stream_upstream_hc_module.html?&_ga=2.88591251.904408446.1519087829-413045118.1519087829#health_check_timeout) 指令：
```
stream {
    ...
    server {
        listen               12345;
        proxy_pass           stream_backend;
        health_check;
        health_check_timeout 5s;
    }
    ...
}
```
[`health_check`](http://nginx.org/en/docs/stream/ngx_stream_upstream_hc_module.html?&_ga=2.88663955.904408446.1519087829-413045118.1519087829#health_check) 指令开启健康检查功能，[`health_check_timeout`](http://nginx.org/en/docs/stream/ngx_stream_upstream_hc_module.html?&_ga=2.88591251.904408446.1519087829-413045118.1519087829#health_check_timeout) 指令在健康检查时覆盖 [`proxy_timeout`](http://nginx.org/en/docs/stream/ngx_stream_proxy_module.html?&_ga=2.63029287.904408446.1519087829-413045118.1519087829#proxy_timeout) 的值，因为健康检查的超时需要设置的很短。
##4.3 微调TCP健康检查
默认情况下，Nginx Plus 会每隔 5 秒钟向 upstream 服务器组中的每台服务器发起一次连接尝试。如果连接无法建立，则 Nginx Plus 会认为健康检查失败，将对应的服务器标记为故障，并且停止将客户端连接发到这台服务器。

要改变默认行为，可以在 [`health_check`](http://nginx.org/en/docs/stream/ngx_stream_upstream_hc_module.html?&_ga=2.88663955.904408446.1519087829-413045118.1519087829#health_check) 指令中包含下面的参数：

- interval – Nginx Plus 每隔多久发送一次健康检查（默认是 5 秒）
- passes – 服务器连续通过几次检查才能被视为健康（默认是 1 次）
- fails – 服务器连续几次检查失败才能被视为故障（默认是 1 次）
```
stream {
    ...
    server {
        listen       12345;
        proxy_pass   stream_backend;
        health_check interval=10 passes=2 fails=3;
    }
    ...
}
```
这个例子中，TCP 健康检查的间隔增大为 10 秒，服务器在 3 次连续的健康检查失败时被认为发生故障，并且服务器需要连续通过 2 次健康检查才被认为再次可用。

默认情况下，Nginx Plus 向 [upstream](http://nginx.org/en/docs/stream/ngx_stream_upstream_module.html?&_ga=2.92259218.904408446.1519087829-413045118.1519087829#upstream) 块中 [server](http://nginx.org/en/docs/stream/ngx_stream_upstream_module.html?&_ga=2.92259218.904408446.1519087829-413045118.1519087829#server) 指令指定的端口发送健康检查消息。可以**指定另一个端口用于健康检查**，这在同时监控一台主机上的多个服务时特别有用。通过 [`health_check`](http://nginx.org/en/docs/stream/ngx_stream_upstream_hc_module.html?&_ga=2.88663955.904408446.1519087829-413045118.1519087829#health_check) 指令中的 port 参数可以覆盖默认端口：
```
stream {
    ...
    server {
      listen       12345;
      proxy_pass   stream_backend;
      health_check port=8080;
    }
    ...
}
```
##4.4 match 配置块
可以配置一系列测试来验证服务器对健康检查的响应。这些测试定义在 stream 上下文中的 [match](http://nginx.org/en/docs/stream/ngx_stream_upstream_hc_module.html?&_ga=2.66505638.904408446.1519087829-413045118.1519087829#match) 配置块中。指定 match 配置块时同时指定名字 tcp_test：
```
stream {
    ...
    match  tcp_test {
        ...
    }
}
```
然后在 [`health_check`](http://nginx.org/en/docs/stream/ngx_stream_upstream_hc_module.html?&_ga=2.88663955.904408446.1519087829-413045118.1519087829#health_check) 指令中通过 match 参数指明要使用哪个 match 块来验证响应：
```
stream {
    ...
    server {
        listen       12345;
        health_check match=tcp_test;
        proxy_pass   stream_backend;
    }
    ...
}
```
成功运行健康检查的条件或测试，是用 set 和 send 参数设置的（The conditions or tests under which a health check succeeds are set with send and expect parameters）：

- [send](http://nginx.org/en/docs/stream/ngx_stream_upstream_hc_module.html?&_ga=2.57811874.904408446.1519087829-413045118.1519087829#match_send) – 要发送到服务器的文本字符串或十六进制文字（“/ x”后跟两个十六进制数字）。
- [expect](http://nginx.org/en/docs/stream/ngx_stream_upstream_hc_module.html?&_ga=2.57811874.904408446.1519087829-413045118.1519087829#match_expect) – 从服务器返回的数据需要匹配的字符串或正则表达式。

这两个参数可以有不同的组合，但是每次最多只能指定一个 send 和一个 expect：

- 如果未指定 send 和 expect 参数，则只测试连接到服务器的能力。

- 如果只指定了 expect 参数，则期望服务器会无条件发送数据：
```
match pop3 {
    expect ~* "\+OK";
}
```
- 如果只指定了 send 参数，则期望连接会成功建立，并且指定的字符串会成功发送到服务器：
```
match pop_quit {
    send QUIT;
}
```
- 如果同时指定 send 和 expect 参数，通过 send 参数从服务器获取的字符串必须匹配 expect 参数指定的正则表达式：
```
stream {
    ...
    upstream   stream_backend {
        zone   upstream_backend 64k;
        server backend1.example.com:12345;
    }

    match http {
        send      "GET / HTTP/1.0\r\nHost: localhost\r\n\r\n";
        expect ~* "200 OK";
    }


    server {
    listen       12345;
    health_check match=http;
    proxy_pass   stream_backend;
    }
}
```
这个例子中，健康检查要通过，则 HTTP 请求一定要发送到服务器，并且从服务器返回的结果包含 `200 OK` 来指示这是一个成功的 HTTP 响应。