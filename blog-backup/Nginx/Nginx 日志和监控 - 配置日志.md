[原文地址](https://www.nginx.com/resources/admin-guide/logging-and-monitoring/)

#1. 配置错误日志 Error Log
Nginx 将遇到的不同严重级别问题的信息写入错误日志。 [error_log](https://nginx.org/en/docs/ngx_core_module.html?&_ga=2.32539702.904408446.1519087829-413045118.1519087829#error_log) 指令设置对特定文件，stderr 或 syslog 的日志记录，并指定要记录的消息的最低严重级别。 默认情况下，错误日志位于 logs/error.log（绝对路径取决于操作系统和安装），并且默认记录所有严重级别的消息。
下面的配置将记录错误信息的最低级别从 error 改为 warn：
```
error_log logs/error.log warn;
```
此时，warn、error crit、alert 和 emerg 这几种级别的消息会写入日志。

错误日志的默认设置在全局范围内有效。为了覆盖这个设置，可以在主配置文件的顶级上下文中使用 [error_log](https://nginx.org/en/docs/ngx_core_module.html?&_ga=2.32539702.904408446.1519087829-413045118.1519087829#error_log) 指令。主配置文件的顶级上下文中的设置可以被其他的配置上下文继承。error_log 还可以在 http、stream、server 和 location 上下文中制定，从而覆盖上层上下文中的设置。发生错误时，相关信息只会写入和错误最相关的上下文指定的那个错误日志。然而，如果在同一个上下文中使用了多个 error_log 指令，错误信息会写入所有指定的日志中。

注意：Nginx 开源版本 1.5.2 之后才增加了在同一个上下文中使用了多个 error_log 指令写入所有指定的日志中这个功能。
#2. 配置访问日志 Access Log
Nginx 会在客户端请求处理完成后，立刻把相关信息写入访问日志。默认情况下，访问日志位于 logs/access.log，并且信息会以预定义的组合格式写入日志。要覆盖默认设置，可以使用 `log_format` 指令更改日志消息的格式， `access_log` 指令指明日志位置和格式。日志格式用变量定义。

以下示例定义了将预定义的组合格式与指示响应的 gzip 压缩比率的值一起扩展的日志格式。 然后将格式应用于启用压缩的虚拟服务器。
```
http {
    log_format compression '$remote_addr - $remote_user [$time_local] '
                           '"$request" $status $body_bytes_sent '
                           '"$http_referer" "$http_user_agent" "$gzip_ratio"';

    server {
        gzip on;
        access_log /spool/logs/nginx-access.log compression;
        ...
    }
}
```
日志格式的另一个例子，可以跟踪 Nginx 和上游服务器之间的不同时间值，如果您的网站体验变慢，可能有助于诊断问题。 可以使用以下变量记录指示的时间值：

- [`$upstream_connect_time`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html?&_ga=2.134334585.904408446.1519087829-413045118.1519087829#var_upstream_connect_time) – 用于和上游服务器建立连接的时间。
- [`$upstream_header_time`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html?&_ga=2.134334585.904408446.1519087829-413045118.1519087829#var_upstream_header_time) – 连接建立到从上游服务器接受响应头中的第一个字节数据之间的时间。
- [`$upstream_response_time`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html?&_ga=2.134334585.904408446.1519087829-413045118.1519087829#var_upstream_response_time) – 连接建立到从上游服务器接受响应体中的最后一个字节数据之间的时间。
- [`$request_time`](https://nginx.org/en/docs/http/ngx_http_log_module.html?&_ga=2.134334585.904408446.1519087829-413045118.1519087829#var_request_time) – 用于处理一个请求的所有时间。

所有时间值均以秒为单位进行测量，以毫秒为分辨率。
```
http {
    log_format upstream_time '$remote_addr - $remote_user [$time_local] '
                             '"$request" $status $body_bytes_sent '
                             '"$http_referer" "$http_user_agent"'
                             'rt=$request_time uct="$upstream_connect_time" uht="$upstream_header_time" urt="$upstream_response_time"';

    server {
        access_log /spool/logs/nginx-access.log upstream_time;
        ...
    }
}
```
下面是几个如何读取结果时间值的规则：

- 当一个请求需要几台服务器来处理时，变量包含几个用逗号分隔的值。
- 当有 upstream 服务器组之间的内部重定向，结果值用分号分隔。
- 当一个请求无法到达 upstream 服务器或者无法接受到完整的头字段，变量包含“0”。
- 连接到 upstream 服务器时发生错误，或者从缓存中获取响应，变量包含“-”。

日志记录可以通过启用日志消息的缓冲区以及名称包含变量的常用日志文件描述符的缓存（the cache of descriptors of frequently used log files whose names contain variables）来优化。 要启用缓冲，请使用 [access_log](https://nginx.org/en/docs/http/ngx_http_log_module.html?&_ga=2.96061332.904408446.1519087829-413045118.1519087829#access_log) 指令的 buffer 参数指定缓冲区的大小。 当下一条日志消息不适合缓冲区（填满缓冲区）以及 [其他一些情况](https://nginx.org/en/docs/http/ngx_http_log_module.html?&_ga=2.256495235.904408446.1519087829-413045118.1519087829#access_log) 时，将缓冲的消息写入日志文件。

通过 [open_log_file_cache](https://nginx.org/en/docs/http/ngx_http_log_module.html?&_ga=2.134211590.904408446.1519087829-413045118.1519087829#open_log_file_cache) 指令开启日志文件描述符的缓存

[access_log](https://nginx.org/en/docs/http/ngx_http_log_module.html?&_ga=2.96061332.904408446.1519087829-413045118.1519087829#access_log) 指令和 error_log 指令类似，指定层级的配置会覆盖上一层级的配置。当请求处理完成后，消息写入当前层级指定的日志文件，或从上面层级继承的日志文件。如果一个层级中定义了多个访问日志，则消息写入到所有的日志中。
#3. 启用条件日志记录（Conditional Logging）
条件日志记录允许从访问日志中排除不重要日志（trivial）或不重要的日志条目（unimportant log entries）。 在 Nginx 中，条件日志记录由 [access_log](https://nginx.org/en/docs/http/ngx_http_log_module.html?&_ga=2.254902272.904408446.1519087829-413045118.1519087829#access_log) 指令的 if 参数启用。

下面的例子中，将 HTTP 状态码为 2XX（成功）和 3XX（重定向）的请求排除：
```
map $status $loggable {
    ~^[23]  0;
    default 1;
}

access_log /path/to/access.log combined if=$loggable;
```
#4. 写入系统日志（Syslog）
syslog 实用程序是计算机消息日志记录的标准，允许从不同设备收集日志消息到单个系统日志服务器上。 在 Nginx 中，使用 [error_log](https://nginx.org/en/docs/ngx_core_module.html?&_ga=2.96397204.904408446.1519087829-413045118.1519087829#error_log) 和 [access_log](https://nginx.org/en/docs/http/ngx_http_log_module.html?&_ga=2.96397204.904408446.1519087829-413045118.1519087829#access_log) 指令中的 `syslog:` 前缀配置如何记录日志到 syslog。

syslog 的消息可以发送到 `server=` 参数，可以是域名，IP 地址或 Unix 域名套接字路径。域名或 IP 地址可以通过制定端口来覆盖默认的 514 端口。Unix 域名套接字路径可以通过 `unix:` 前缀制定：
```
error_log server=unix:/var/log/nginx.sock debug;
access_log syslog:server=[2001:db8::1]:1234,facility=local7,tag=nginx,severity=info;
```
这个例子中，Nginx 错误日志消息中的 debug 级别消息会写入制定的 Unix 域名套接字路径，访问日志会写入通过 IPv6 地址和端口号 1234 制定的 syslog 服务器。

`facility=` 参数指定记录消息的程序的类型。 默认值是 local7。 其他可能的值是：auth, authpriv, daemon, cron, ftp, lpr, kern, mail, news, syslog, user, uucp, local0 ... local7.

`tag=` 参数将自定义 tag 标记应用于系统日志消息（这里制定为 nginx）。

`severity=` 参数设置访问日志的系统日志消息的严重级别。可用值按照严重级别排列如下：debug、info、notice、warn、error (default)、crit、alert 和 emerg。所有指定级别和更加严重级别的消息会写入日志。指定的严重级别为 error 时，严重级别为 crit、alert 和 emerg 的消息也会写入日志。
#5. 实时活动监控
Nginx Plus 提供了一个实时活动监控接口，显示 HTTP 和 TCP 上游 upstream 服务器的关键负载和性能指标。更多信息 [参考这里](https://www.nginx.com/resources/admin-guide/Monitoring/)。
#6. 用 Amplify 监测 Nginx
Nginx 有一个自带的监控工具，称为 [NGINX Amplify](https://www.nginx.com/products/nginx-amplify/)，是一个 SaaS 工具，可以用来监控服务器。

Nginx Amplify 上手资料 [参考这里](https://amplify.nginx.com/docs/?_ga=2.96987927.904408446.1519087829-413045118.1519087829)。 在十分钟内，可以获得所有关键 Nginx 指标的开箱即用图表，而且还可以获得 Linux 操作系统，PHP-FPM，Docker 容器等的指标。 Nginx Amplify 会自动使用来自 [stub_status](https://nginx.org/en/docs/http/ngx_http_stub_status_module.html?_ga=2.30000183.904408446.1519087829-413045118.1519087829) 和 [访问日志](https://www.nginx.com/resources/admin-guide/logging-and-monitoring/#access_log) 的指标，并且还可以收集各种操作系统信息。

更多资料 [参考这里](https://www.nginx.com/products/nginx-amplify/)。