[原文地址](https://www.nginx.com/resources/admin-guide/compression-and-decompression/)

#1. 概述
通过压缩响应数据，可以减少传输数据的大小。但是，由于压缩发生在请求的处理过程中，会增加相当大的处理开销，增大延时，对性能产生负面影响。Nginx 会在数据发送到客户端之前进行压缩，但是对于已经压缩过的数据不会进行二次压缩（比如代理服务器压缩过一次了）。
#2. 开启压缩功能
[`gzip`](http://nginx.org/en/docs/http/ngx_http_gzip_module.html#gzip) 指令开启压缩功能：
```
gzip on;
```
Nginx 默认只压缩 MIME 类型为 text/html 的响应。需要通过 [`gzip_types`](http://nginx.org/en/docs/http/ngx_http_gzip_module.html#gzip_types) 指令添加要压缩的其他类型的响应。
```
gzip_types text/plain application/xml;
```
[`gzip_min_length`](http://nginx.org/en/docs/http/ngx_http_gzip_module.html#gzip_min_length) 指令指定要压缩响应的最小长度，默认 20 字节。下面例子中改为了 1000 字节：
```
gzip_min_length 1000;
```
默认情况下，Nginx 不压缩要转发给被代理服务器的请求。通过请求头中是否出现 Via 字段来判断这个请求是否来自代理服务器。[`gzip_proxied`](http://nginx.org/en/docs/http/ngx_http_gzip_module.html#gzip_proxied) 指令可以配置压缩这些响应。
`gzip_proxied` 指令有多个参数可以指定 Nginx 要压缩哪些代理请求。例如，压缩不会在代理服务器上缓存的那些请求对应的响应。为此，gzip_proxied 指令的参数指示 Nginx 检查响应中的 Cache-Control 头字段，并在该值为 no-cache，no-store 或 private 时压缩响应。另外，必须包含 expired 参数以检查 Expires 头字段的值。auth 参数检查 Authorization 头字段（授权响应是特定于最终用户的，并且通常不会被缓存）：
```
gzip_proxied no-cache no-store private expired auth;
```
与大多数其他指令一样，配置压缩的指令可以包含在 http 上下文中，或者在 server 或 location 配置块中。

gzip 压缩的整体配置示例：
```
server {
    gzip on;
    gzip_types      text/plain application/xml;
    gzip_proxied    no-cache no-store private expired auth;
    gzip_min_length 1000;
    ...
}
```
#3. 开启解压缩功能
部分客户端不支持 gzip 编码的响应数据。同时，可能需要存储压缩数据，或者即时压缩响应并将它们存储在缓存中。 为了为所有的客户端提供服务，Nginx 可以在将数据发送到不支持 gzip 的客户端时实时解压缩数据。

[`gunzip`](http://nginx.org/en/docs/http/ngx_http_gunzip_module.html#gunzip) 指令可以开启实时解压缩：
```
location /storage/ {
    gunzip on;
    ...
}
```
gunzip 指令可以和 gzip 指令出现在同一个上下文中：
```
server {
    gzip on;
    gzip_min_length 1000;
    gunzip on;
    ...
}
```
>注意，gunzip 指令定义在一个独立的模块中，源码编译安装 Nginx 可能默认不包含这个模块。
#4. 发送压缩过的文件
在合适的上下文中设置 [`gzip_static`](http://nginx.org/en/docs/http/ngx_http_gzip_static_module.html#gzip_static) 可以发送文件的压缩版本到客户端。
```
location / {
    gzip_static on;
}
```
上面例子中，如果请求 /path/to/file，Nginx 会寻找对应文件的压缩过的版本（/path/to/file.gz）并发送。如果压缩版本不存在，或客户端不支持 gzip，Nginx 会寻找并发送非压缩版本的文件。

>注意，gzip_static 指令不支持实时压缩。这个指令只是使用之前用任何压缩工具压缩过的文件。如果要实时压缩内容（不只是静态内容），必须使用 gzip 指令。

>注意，gzip_static 指令定义在一个独立的模块中，源码编译安装 Nginx 可能默认不包含这个模块。