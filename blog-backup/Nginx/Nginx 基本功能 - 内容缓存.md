[原文地址](https://www.nginx.com/resources/admin-guide/content-caching/)

#1. 概述
缓存开启后，Nginx 会把响应保存到磁盘缓存上，并使用缓存中的数据响应客户端，而不必每次都请求代理相同的内容。
要了解更多 Nginx 缓存知识，可以看 [Content Caching with NGINX webinar on demand](https://www.nginx.com/products/nginx/caching/) 并深入了解动态内容缓存，缓存清除和延迟缓存等功能。
#2. 开启对响应的缓存功能
要开启缓存，需要在 http 上下文内使用 [`proxy_cache_path`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_path) 指令。第一个参数是缓存内容对应本地文件系统的路径，`keys_zone` 参数定义用于存储缓存条目的元数据的共享内存区域的名称和大小：
```
http {
    ...
    proxy_cache_path /data/nginx/cache keys_zone=one:10m;
}
```
然后在要为其缓存服务器响应的上下文（protocol type, virtual server 或 location）中包含 [`proxy_cache`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache) 指令，并指定由 `proxy_cache_path` 指令的 `keys_zone` 参数定义的区域名称：
```
http {
    ...
    proxy_cache_path /data/nginx/cache keys_zone=one:10m;

    server {
        proxy_cache one;
        location / {
            proxy_pass http://localhost:8000;
        }
    }
}
```
注意，`keys_zone` 参数定义的缓冲区大小不会限制缓存的响应数据的总大小。缓存的响应本身与元数据的副本一起存储在文件系统上的特定文件中。`proxy_cache_path` 指令的 `max_size` 参数会限制缓存的响应数据的总大小。但是注意，缓存的数据量可能暂时超过此限制，参考下一部分。
#3. 参与缓存的NGINX进程 NGINX Processes Involved in Caching
有两个额外的NGINX进程涉及缓存：

- 缓存管理器（cache manager） 会定期运行来检查缓存的状态。如果缓存大小超过了 `proxy_cache_path` 指令中设置的 `max_size` 参数限制，缓存管理器会删除最后访问的数据。缓存数据大小可以在缓存管理器工作的间隙暂时的超过限制值。
- 缓存加载器（cache loader）只会在 Nginx 启动后立刻执行一次。它会把之前缓存的数据的元数据加载到共享内存区域。一次加载整个缓存可能会消耗大量的资源，导致 Nginx 在启动后的最初几分钟内性能过低。 为了避免这种情况，可以在 `proxy_cache_path` 指令中配置缓存的迭代加载：
 - `loader_threshold` – 迭代的持续时间，以毫秒为单位，默认 200 ms。
 - `loader_files` – 每次迭代时加载文件的最大格式，默认 100。
 - `loader_sleeps` – 两次迭代之间的延时，单位毫秒，默认 50。

下面的例子中，迭代在 300 毫秒或 加载了 200 个文件后停止：
```
proxy_cache_path /data/nginx/cache keys_zone=one:10m loader_threshold=300 loader_files=200;
```
#4. 指定要缓存的请求
默认情况下，Nginx 会缓存使用 HTTP GET 和 HEAD 方法所做请求的所有响应（在第一次从代理服务器接收到此类响应才缓存）。作为请求的关键字（标识符），Nginx 使用请求字符串。如果一个请求和已缓存的响应有相同的关键字，Nginx 会把已缓存的响应发送给客户端。可以在 http、server 或 location 上下文中使用不同指令来控制缓存哪些响应。
[`proxy_cache_key`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_key) 指令可以改变用于计算关键字的请求特性：
To change the request characteristics used in calculating the key, include the proxy_cache_key directive:
```
proxy_cache_key "$host$request_uri$cookie_user";
```
[`proxy_cache_min_uses`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_min_uses) 指令可以定义缓存响应之前使用相同密钥的请求的最小重复次数：
```
proxy_cache_min_uses 5;
```
[`proxy_cache_methods`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_methods) 指令可以声明缓存哪些请求方法对应的响应（默认只有 GET 和 HEAD）：
```
proxy_cache_methods GET HEAD POST;
```
#5. 限制或绕过缓存
默认情况下，响应将无限期地保留在缓存中。只有在缓存超过配置的最大可用空间才会删除，然后按照最后一次请求的时间排序。可以通过在 http、server 或 location 上下文中使用指令控制缓存的有效时间，不管它们是否被使用。

[`proxy_cache_valid`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_valid)  指令可以针对指定状态码限制缓存的有效时间。
```
proxy_cache_valid 200 302 10m;
proxy_cache_valid 404      1m;
```
上面例子中，对于状态码 200 或302 的响应，缓存有效时间是 10 分钟，而对于 404 响应的缓存有 1 分钟的有效时间。
如果要对所有状态码统一处理，需要使用 any 作为第一个参数：
```
proxy_cache_valid any 5m;
```
[`proxy_cache_bypass`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_bypass) 指令可以设置 Nginx 在哪些情况下不使用缓存。每个参数定义一种情况，并且包含多个变量。如果至少一个参数非空且不等于0，Nginx 就不会再去查找缓存中是否有响应，而是直接把请求发送到后面的服务器。
```
proxy_cache_bypass $cookie_nocache $arg_nocache$arg_comment;
```
[`proxy_no_cache`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_no_cache) 指令可以使 Nginx 在任何情况下都不缓存响应。`proxy_no_cache` 指令的参数和 `proxy_cache_bypass` 指令一样。
```
proxy_no_cache $http_pragma $http_authorization;
```
#6. 清除缓存
Nginx 可以删除缓存中的过期文件，防止同时保存网页的新旧两个版本。缓存通过接受一个包含指定的 HTTP 头或使用“PURGE”的 HTTP 方法的“purge”请求来清除。
##6.1 配置清除缓存
让我们设置一个配置，表示使用“PURGE”HTTP 方法的请求，并删除匹配的 URL。
在 http 上下文中，创建新变量。例如依赖 `$request_method` 变量的 `$purge_method` 变量。
```
http {
    ...
    map $request_method $purge_method {
        PURGE 1;
        default 0;
    }
}
```
在配置了缓存的 location 中，使用 `proxy_cache_purge` 指令声明清除缓存的请求。下面例子中，使用之前配置的 `$purge_method`：
```
server {
    listen      80;
    server_name www.example.com;

    location / {
        proxy_pass  https://localhost:8002;
        proxy_cache mycache;

        proxy_cache_purge $purge_method;
    }
}
```
##6.2 发送清除命令
`proxy_cache_purge` 指令配置完成后，需要发送清除缓存的请求才能清除缓存。可以使用多种工具来清除缓存，比如 curl：
```
$ curl -X PURGE -D – "https://www.example.com/*"
HTTP/1.1 204 No Content
Server: nginx/1.5.7
Date: Sat, 01 Dec 2015 16:33:04 GMT
Connection: keep-alive
```
这个例子中，有通用的 URL 部分（由星号通配符指定）的资源将会被删除。然而，这些缓存入口并不会彻底从缓存中删除，他们会一直留在磁盘上，直到被 `proxy_cache_path` 指令的 inactivity 参数删除，或被缓存清除进程处理，或一个客户端尝试访问缓存。
##6.3 限制访问清除命令
注意，最好为可以执行清除缓存命令的机器限定 IP：
```
geo $purge_allowed {
   default         0;  # deny from other
   10.0.0.1        1;  # allow from localhost
   192.168.0.0/24  1;  # allow from 10.0.0.0/24
}

map $request_method $purge_method {
   PURGE   $purge_allowed;
   default 0;
}
```
这个例子中，Nginx 会检查请求中是否使用 PURGE 方法，如果是，会检查 IP 地址是否在白名单。如果在白名单，`$purge_allowed` 变量会被设置到 `$purge_allowed`，1 则允许清除，0 则禁止。
##6.4 从缓存中彻底删除文件
需要激活一个特殊的删除缓存进程来彻底删除匹配到的文件，这个进程会遍历所有的缓存入口，并删除匹配到的入口。在 http 上下文，给 `proxy_cache_path` 指令添加 purger 参数：
```
proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=mycache:10m purger=on;
```
##6.5 清除缓存的示例配置
```
http {
    ...
    proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=mycache:10m purger=on;

    map $request_method $purge_method {
        PURGE 1;
        default 0;
    }

    server {
        listen      80;
        server_name www.example.com;

        location / {
            proxy_pass        https://localhost:8002;
            proxy_cache       mycache;
            proxy_cache_purge $purge_method;
        }
    }

    geo $purge_allowed {
       default         0;
       10.0.0.1        1;
       192.168.0.0/24  1;
    }

    map $request_method $purge_method {
       PURGE   $purge_allowed;
       default 0;
    }
}
```
#7. 字节范围的缓存
初始缓存填充操作需要一些时间，特别是对于大文件时间更长。当第一个请求开始下载视频文件的一部分时，下一个请求将不得不等待整个文件被下载并放入缓存中。
Nginx 可以使用 [cache slice module](http://nginx.org/en/docs/http/ngx_http_slice_module.html) 实现这种大文件的缓存的逐步填充。文件首先分割为片。每个范围的请求选择特定的片，如果该范围还没有缓存，则将其放入缓存中。

开启字节范围的缓存的步骤：
####1. 确保 Nginx 包含了 [slice](http://nginx.org/en/docs/http/ngx_http_slice_module.html) 模块。
####2. 用 [`slice`](http://nginx.org/en/docs/http/ngx_http_slice_module.html#slice) 指令指定分片的大小：
```
location / {
    slice  1m;
}
```
分片应该调整为最适合下载的大小。太小的分片可能导致内存消耗过大并在处理请求时打开太多的文件，太大的分片会增大延时。
####3. 在 `proxy_cache_key` 指令中使用 [`$slice_range`](http://nginx.org/en/docs/http/ngx_http_slice_module.html#var_slice_range) 变量：
```
proxy_cache_key $uri$is_args$args$slice_range;
```
####4. 对于响应码为 206 的响应开启缓存：
```
proxy_cache_valid 200 206 1h;
```
####5. 在 Range 头字段中传递 [`$slice_range`](http://nginx.org/en/docs/http/ngx_http_slice_module.html#var_slice_range) 变量，可以把请求范围传递给代理服务器。
```
proxy_set_header  Range $slice_range;
```
字节范围的缓存示例：
```
location / {
    slice             1m;
    proxy_cache       cache;
    proxy_cache_key   $uri$is_args$args$slice_range;
    proxy_set_header  Range $slice_range;
    proxy_cache_valid 200 206 1h;
    proxy_pass        http://localhost:8000;
}
```
注意，如果开启了分片缓存功能，初始文件不应更改。
#8. 综合配置示例
下面的例子综合了上面提到的几个缓存有关的选项，其中的两个 location 用不同的方式使用的同一个缓存：
```
http {
    ...
    proxy_cache_path /data/nginx/cache keys_zone=one:10m loader_threshold=300 loader_files=200 max_size=200m;

    server {
        listen 8080;
        proxy_cache one;

        location / {
            proxy_pass http://backend1;
        }

        location /some/path {
            proxy_pass http://backend2;
            proxy_cache_valid any 1m;
            proxy_cache_min_uses 3;
            proxy_cache_bypass $cookie_nocache $arg_nocache$arg_comment;
        }
    }
}
```
因为来自 backend1 的响应很少改变，所有没有使用 `cache-control` 指令。第一次请求之后就会缓存响应，并一直保持有效状态。
相比之下，来自 backend2 的响应经常变化，因此响应的缓存只有1分钟的有效期，并且同样的请求出现3次时才会缓存对应的响应。此外，如果请求匹配了绕过缓存指令 `proxy_cache_bypass `，Nginx 会立刻把请求发到服务器而不去查找缓存。