[原文地址](https://www.nginx.com/resources/admin-guide/serving-static-content/)
[TCP queue 的一些问题-博客](http://jaseywang.me/2014/07/20/tcp-queue-%E7%9A%84%E4%B8%80%E4%BA%9B%E9%97%AE%E9%A2%98/)
[关于tcp listen queue的一点事-豆瓣](https://www.douban.com/note/178129553/)

#1. root 目录和索引文件
[root](http://nginx.org/en/docs/http/ngx_http_core_module.html#root) 指令声明了要查找文件的目录。Nginx 会把请求的 URI 添加到 root 指令指定的路径之后，来获取请求文件对应的目录。root 指令可以放在 http、server 或 location 上下文的任何位置。
下面例子中的 root 指令定义在 server 中。所有的没有重新实现 root 指令的 location 都会使用 server 中的 root 指令指定的目录：
```
server {
    root /www/data;

    location / {
    }

    location /images/ {
    }

    location ~ \.(mp3|mp4) {
        root /www/media;
    }
}
```
如果 URI 以 mp3 或 mp4 后缀结尾，Nginx 会在 /www/media/ 目录查找文件。否则在 /www/data 目录中查找。

如果请求以 / 结尾，Nginx 会把这个请求当做一个目录请求，尝试找这个目录中的 index 文件。[index](http://nginx.org/en/docs/http/ngx_http_index_module.html#index) 指令定义了 index 文件的文件名（默认使用 index.html 文件）。例如上面的配置，如果请求是 /images/some/path/，Nginx 会尝试寻找并返回文件 /www/data/images/some/path/index.html，如果文件不存在则返回 404。

[autoindex](http://nginx.org/en/docs/http/ngx_http_autoindex_module.html#autoindex) 指令如果设置为 on，则 Nginx 会返回自动生成的目录列表：
```
location /images/ {
    autoindex on;
}
```
index 指令中可以列出多个文件。Nginx 会按顺序查找文件并返回第一个找到的文件。
```
location / {
    index index.$geo.html index.htm index.html;
}
```
这里使用的 `$geo` 变量是通过 geo 指令设置的自定义变量。变量值由客户端的 IP 地址决定。

为了返回 index 文件，Nginx 首先检查文件是否存在，然后将 index 文件名添加到请求的 URI 之后构成新的 URI，最后内部重定向到这个新的 URI。内部重定向会导致重新搜索 location，并可能在另一个 location 中结束，如下例所示：
```
location / {
    root /data;
    index index.html index.php;
}

location ~ \.php {
    fastcgi_pass localhost:8000;
    ...
}
```
如果请求中的 URI 是 /path/，并且 /data/path/index.html 文件不存在，而 /data/path/index.php 文件存在，对 /path/index.php 文件的内部重定向会被第二个 location 匹配到，在这个 location 内通过 fastcgi_pass 指定的 FastCGI 处理请求。
#2. 检查文件是否存在（try_files 指令）
[`try_files`](http://nginx.org/en/docs/http/ngx_http_core_module.html#try_files) 指令可以检查指定的文件或目录是否存在，从而执行内部重定向或在文件不存在的时候返回指定的 HTTP 状态码。

例如，通过 `try_files` 指令和 `$uri` 变量检查和请求中的 URI 相关的文件是否存在：
```
server {
    root /www/data;

    location /images/ {
        try_files $uri /images/default.gif;
    }
}
```
文件以 URI 的形式指定，并且使用在当前 location 或 server 的上下文中设置的 root 或 alias 指令进行处理。此时如果源 URI 指定的文件不存在，Nginx 会内部重定向到最后一个参数指定的 URI，返回 /www/data/images/default.gif。

最后一个参数也可以是状态码（前面需要加等号）或一个 location 的名字。下面的例子中，如果 `try_files` 指令指定的文件或目录都不存在，则返回 404 错误：
```
location / {
    try_files $uri $uri/ $uri.html =404;
}
```
下面的例子中，如果原始 URI 和带有附加斜线的 URI 指定的文件或目录都不存在，请求就会被重定向到指定名称的 location：
```
location / {
    try_files $uri $uri/ @backend;
}

location @backend {
    proxy_pass http://backend.example.com;
}
```
更多信息可以查看 [Content Caching](https://www.nginx.com/resources/webinars/content-caching-nginx-plus/) 内容缓存，了解如何提高网站性能，深入了解 Nginx 的缓存功能。
#3. 优化 Nginx（Optimizing NGINX Speed for Serving Content）
加载速度是服务器的一个关键指标。对 Nginx 配置进行微小的优化可能会提高生产力并实现最佳性能。
##使能 sendfile 指令
默认情况下，Nginx 自己处理文件传输并在发送文件之前将文件复制到缓冲区中。启用 [sendfile](http://nginx.org/en/docs/http/ngx_http_core_module.html#sendfile) 指令可以去掉将数据复制到缓冲区的步骤，并允许将数据从一个文件描述符（file descriptor）直接复制到另一个文件描述符。 为了防止一个快速连接完全占用工作进程，可以通过定义 [`sendfile_max_chunk`](http://nginx.org/en/docs/http/ngx_http_core_module.html#sendfile_max_chunk) 指令来限制在单个 sendfile 调用中传输的数据量：
```
location /mp3 {
    sendfile           on;
    sendfile_max_chunk 1m;
    ...
}
```
##使能 tcp_nopush 指令
[tcp_nopush](http://nginx.org/en/docs/http/ngx_http_core_module.html#tcp_nopush) 指令需要和 sendfile 指令配合使用。
如果 tcp_nopush 指令和 sendfile 指令同时使能，则 Nginx 在通过sendfile 获取数据块后会立即在一个数据包中发送 HTTP 响应头。
```
location /mp3 {
    sendfile   on;
    tcp_nopush on;
    ...
}
```
##使能 tcp_nodelay 指令
[tcp_nodelay](http://nginx.org/en/docs/http/ngx_http_core_module.html#tcp_nodelay) 选项允许覆盖 Nagle 的算法，最初设计用于解决慢速网络中小数据包的问题。该算法将多个小数据包合并为较大的数据包，并以200毫秒的时延发送数据包。如今，在提供大的静态文件时，无论数据包大小如何，都可以立即发送数据。延迟还会影响在线应用程序（ssh，在线游戏，在线交易）。 默认情况下，tcp_nodelay 指令被使能，禁用 Nagle 的算法。 该选项仅用于保持连接：
```
location /mp3  {
    tcp_nodelay       on;
    keepalive_timeout 65;
    ...
}
```
##优化积压队列（Optimizing the Backlog Queue）
一个重要指标是 Nginx 能够处理传入连接的速度。常用规则是当连接建立时，它被放入侦听套接字的“listen”队列中。 在正常负载下，队列很短，或者根本没有队列。但是在高负载下，队列可能急剧增长，这可能会导致性能不均衡，连接丢失和延迟。
###测量监听队列（Measuring the Listen Queue）
运行下面的命令可以测量监听队列（Linux 下的 netstat 命令不支持 -L 参数，需要使用命令 `ss -l`，[参考这里](https://serverfault.com/questions/432022/linux-netstat-listeting-queue-length)）：
```
netstat -Lan
```
输出如下：
```
Current listen queue sizes (qlen/incqlen/maxqlen)
Listen         Local Address         
0/0/128        *.12345            
10/0/128        *.80       
0/0/128        *.8080
```
上面的输出显示，在 80 端口的监听队列有 10 个未接受的连接，最大连接数限制为 128，这种情况是正常的。

然而，如果输出是下面这样子的：
```
Current listen queue sizes (qlen/incqlen/maxqlen)
Listen         Local Address         
0/0/128        *.12345            
192/0/128        *.80       
0/0/128        *.8080
```
上面显示有 10 个未接受的连接，超过了最大限制 128。在网站访问量大时这种情况挺常见的。为了达到最佳性能，可以修改操作系统和 Nginx 配置，增加 Nginx 可以等待接受的队列中的最大连接数。
###调整操作系统（Linux，FreeBSD）
可以增加 [net.core.somaxconn](http://blog.csdn.net/mawming/article/details/51952411)  参数的值（默认 128）以应对高并发流量：

- 对于 FreeBSD 运行命令 `sudo sysctl kern.ipc.somaxconn=4096`
- 对于 Linux 运行命令 `sudo sysctl -w net.core.somaxconn=4096`

打开文件 /etc/sysctl.conf，添加这一行：`net.core.somaxconn = 4096`
###调整 Nginx
如果设置的 somaxconn 值大于 512，需要更改 Nginx 配置文件中的 backlog 参数匹配这个设置：
```
server {
    listen 80 backlog=4096;
    # The rest of server configuration
}
```