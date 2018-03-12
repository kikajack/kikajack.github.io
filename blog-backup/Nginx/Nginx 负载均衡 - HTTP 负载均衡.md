[原文地址](https://www.nginx.com/resources/admin-guide/load-balancer/)

#1. 负载均衡概述
跨多个应用程序实例进行负载均衡是优化资源利用率，最大化吞吐量，减少延迟并确保容错配置的常用技术。
观看NGINX负载平衡软件网络研讨会点播，深入了解NGINX用户用于构建大规模，高可用性Web服务的技术。

深入了解构建大规模，高可用性 Web 服务的技术可以参考： [`NGINX Load Balancing Software Webinar On Demand`](https://www.nginx.com/resources/webinars/nginx-plus-for-load-balancing-30-min/) 。
Nginx 可以在不同的部署场景中用作高效的 HTTP 负载均衡器，[参考这里](https://www.nginx.com/blog/nginx-load-balance-deployment-models/)。
#2. 将流量代理到一组服务器
通过 [`upstream`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream) 指令定义 group 组后，可以为一组服务器使用 Nginx。`upstream` 指令在 [http](http://nginx.org/en/docs/http/ngx_http_core_module.html#http) 上下文中使用。

服务器组中的每台服务器通过 server 指令来配置（不要和 Nginx 上运行的虚拟服务器配置中的 sever 块搞混了）。下面的配置定义了一个名叫 backend 的分组，包含三台服务器的配置（可能需要不止三台真实服务器）：
```
http {
    upstream backend {
        server backend1.example.com weight=5;
        server backend2.example.com;
        server 192.0.0.1 backup;
    }
}
```
要把请求发送到服务器组，需要用 `proxy_pass` 指令（或根据具体协议采用 `fastcgi_pass`，`memcached_pass`，`uwsgi_pass`，`scgi_pass ` 指令）指定组名。下面例子中，Nginx 中运行的一个虚拟服务器把所有请求转发到上面例子中定义的 backend 服务器组：
```
server {
    location / {
        proxy_pass http://backend;
    }
}
```
下面的例子总结了上面的两个例子，三台服务器中，有两台运行相同应用的实例，一台备份：
```
http {
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;
        server 192.0.0.1 backup;
    }
    server {
        location / {
            proxy_pass http://backend;
        }
    }
}
```
#3. 选择一个负载均衡策略
Nginx 支持四种负载均衡策略，Nginx Plus 支持五种。

##3.1 Round-robin
默认方法，没有对应的指令。根据服务器权重，请求在各个服务器上均匀分发：
```
upstream backend {
   server backend1.example.com;
   server backend2.example.com;
}
```
##3.2 [least_conn](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#least_conn)
考虑 [服务器权重](https://www.nginx.com/resources/admin-guide/load-balancer/#weight) 的同时，将请求发送到有效连接数量最少的服务器：
```
upstream backend {
    least_conn;

    server backend1.example.com;
    server backend2.example.com;
}
```
##3.3 [ip_hash](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#ip_hash)
根据客户端 IP 地址决定请求发送到哪台服务器。在这种情况下，使用 IPv4 地址的前三个八位字节或整个 IPv6 地址来计算散列值。该方法保证来自同一地址的请求到达同一个服务器，除非它不可用。
```
upstream backend {
    ip_hash;

    server backend1.example.com;
    server backend2.example.com;
}
```
如果其中一台服务器需要临时移除，则可以使用 [`down`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#down) 参数进行标记，以保留客户端IP地址的当前散列。由该服务器处理的请求会自动发送到组中的下一台服务器：
```
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com down;
}
```
##3.4 通用 [hash](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#hash)
通过用户自定义的键值来决定请求发送到哪一台服务器，键值可以是文本、变量或其组合。例如，键值可以是源 IP，端口或 URI：
```
upstream backend {
    hash $request_uri consistent;

    server backend1.example.com;
    server backend2.example.com;
}
```
hash 指令的可选一致性参数使得 ketama 能够保持哈希负载平衡。根据用户定义的散列键值，请求将在所有 upstream 服务器上均匀分布。 如果将 upstream 服务器添加到 upstream group 中或从 upstream group 中删除，则仅重新映射几个密钥，这将在负载平衡缓存服务器和其他积聚状态的应用程序的情况下最小化缓存未命中。
##3.5 least_time (NGINX Plus) 
对于每个请求，NGINX Plus 会选择平均延迟最低，活动连接数最少的服务器，其中最低平均延迟时间是根据 [`least_time`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#least_time) 指令中包含以下哪个参数计算的：

- header – 从服务器收到第一个字节的时间。
- last_byte – 从服务器收到完整响应的时间。
```
upstream backend {
    least_time header;

    server backend1.example.com;
    server backend2.example.com;
}
```
注意：配置 `round-robin` 策略之外的其他策略时，把相关的指令放在 upstream 块内的 server 指令列表之上。
#4. 服务器权重
使用 `round-robin` 策略时，默认情况下，Nginx 会根据权重在服务器组中分发请求。[`server`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#server) 指令的 [`weight`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#weight) 参数用于设置一台服务器的权重，默认是 1：
```
upstream backend {
    server backend1.example.com weight=5;
    server backend2.example.com;
    server 192.0.0.1 backup;
}
```
这个例子中，backend1.example.com 的权重是 5，另外两台服务器使用默认权重 1，但是 IP 地址是 192.0.0.1 的服务器被标记为备份服务器，只有在其他服务器都不可访问时才会接受请求。在这个权重配置下，每六个请求会有五个发到 backend1.example.com。
#5. 服务器慢启动
服务器慢启动功能可防止最近恢复的服务器被连接淹没，这可能会超时并导致服务器再次被标记为宕机。
NGINX Plus 中，慢启动功能可以让一台 upstream 服务器在恢复或可用时，权重从零开始逐步增加到正常值。可以通过在 server 指令中使用 [`slow_start`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#slow_start) 参数实现：
```
upstream backend {
    server backend1.example.com slow_start=30s;
    server backend2.example.com;
    server 192.0.0.1 backup;
}
```
[`slow_start`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#slow_start) 参数后面的值用来设定慢启动过程的时间。

注意，如果只有一台服务器，`max_fails`，`fail_timeout` 和 `slow_start ` 参数都会被忽略，这台服务器永远都不会被认为不可用。
#6. 启用会话持久性（Session persistence）功能
会话持久性意味着 NGINX Plus 识别用户 session 并将来自此 session 的请求路由到同一 upstream 服务器。

NGINX Plus 支持三种会话持久性策略，通过 [`sticky`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#sticky) 指令指定：

##6.1 [sticky cookie](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#sticky_cookie)
通过这种方法，NGINX Plus 为来自上游组的第一个响应添加一个会话 cookie，并识别发送响应的服务器。 当客户端发出下一个请求时，它将包含 cookie 值，NGINX Plus 会将请求路由到同一上游服务器：
```
upstream backend {
    server backend1.example.com;
    server backend2.example.com;

    sticky cookie srv_id expires=1h domain=.example.com path=/;
}
```
这个例子中，`srv_id` 参数用于设置要设置或检查的 cookie 名字，可选的 `expires` 参数用于设置浏览器保存 cookie 的时间，可选的 `path` 参数用于设置 cookie 的保存路径。这是最简单的 session 持久化示例。
##6.2 [sticky route](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#sticky_route)
通过这种方法，NGINX Plus 会在收到第一个请求的时候为客户端分配一个 route。随后的所有请求都将与服务器伪指令的 route 参数进行比较，以识别代理请求的服务器。 路由信息取自 cookie 或 URI。
```
upstream backend {
    server backend1.example.com route=a;
    server backend2.example.com route=b;

    sticky route $route_cookie $route_uri;
}
```
##6.3 [sticky learn](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#sticky_learn)
通过这种方法，NGINX Plus 首先通过检查请求和响应来查找会话标识符（session identifier）。然后 NGINX Plus “learns”哪个 upstream 服务器跟会话标识符相关。这些标识符通常通过 HTTP cookie 来传递。如果一个请求中包含的会话标识符已经被 learned 了，NGINX Plus 会把请求发送到相关的服务器：
```
upstream backend {
   server backend1.example.com;
   server backend2.example.com;

   sticky learn
       create=$upstream_cookie_examplecookie
       lookup=$cookie_examplecookie
       zone=client_sessions:1m
       timeout=1h;
}
```
这个例子中， upstream 中的服务器会通过在响应中设置名为 `EXAMPLECOOKIE` 的 cookie 来创建一个 session。参数意义如下：

- create：必选参数，指定了一个变量，该变量指示如何创建新会话。 在我们的示例中，新会话是从 upstream 服务器发送的名为 `EXAMPLECOOKIE` 的 cookie 来创建的。
- lookup：必选参数，指定了如何查找已存在的 session。这个例子中，通过客户端发送的名为 `EXAMPLECOOKIE` 的 cookie 来查找已存在的 session。
- zone：必选参数，指定了用于存储 sticky session 的相关信息的共享内存空间。这个例子中，这个内存空间名称是 client_sessions，大小 1 MB。

这是一种更复杂的会话持久性方法，因为它不需要在客户端保留任何cookie：所有信息都保存在服务器端的共享内存区域。
#7. 限制连接数量
通过使用 [`max_conns`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#max_conns) 参数可以限制 NGINX Plus 的连接，保持所需的连接数量。
如果已达到 `max_conns` 的限制，则只要指定了 [`queue`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#queue) 指令，就可以将请求放入队列以进一步处理。 `queue` 指令设置了队列中可以同时保持的最大请求数量：
```
upstream backend {
    server backend1.example.com  max_conns=3;
    server backend2.example.com;

    queue 100 timeout=70;
}
```
如果队列中请求数量超限，或者在可选的 timeout 参数中指定的超时期间无法选择 upstream 服务器，则客户端将收到错误。
注意，如果其他 [工作进程](http://nginx.org/en/docs/ngx_core_module.html#worker_processes) 打开空闲的 [`keepalive`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive) 连接，则 `max_conns` 限制将被忽略。 因此，在 [多个工作进程共享内存](https://www.nginx.com/resources/admin-guide/load-balancer/#zone) 的配置中，与服务器的连接总数可能会超过 `max_conns` 值。
#8. 被动健康监测
当 Nginx 认为一台服务器不可用时，会暂时停止将请求发送到这台服务器，直到 Nginx 认为这台服务器可用才会继续。server 指令的以下参数配置了服务器在哪些情况下不可用：

- [`fail_timeout`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#fail_timeout) ：设置时间段，这段时间内如果发生指定数量的失败尝试，则认为服务器在这段时间内不可用（sets the time during which the specified number of failed attempts should happen and still consider the server unavailable.）。换句话说，[`fail_timeout`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#fail_timeout) 参数设置服务器不可用的区间。
- [`max_fails`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#max_fails) ：设置失败次数，在指定时间段内失败这个次数时认为服务器不可用。

默认设置是 10 秒钟失败 1 次就认为服务器不可用。此时 Nginx 只要有一个请求发送失败或没有应答，就认为服务器在 10 秒之内不可用。示例：
```
upstream backend {
    server backend1.example.com;
    server backend2.example.com max_fails=3 fail_timeout=30s;
    server backend3.example.com max_fails=2;
}
```
接下来是一些更复杂的功能，用于跟踪 NGINX Plus 服务器的可用性。
#9. 主动健康监测
定期向每个服务器发送特殊请求并检查满足特定条件的响应，可以监视服务器的可用性。
为了开启这种健康监测，需要在 `nginx.conf` 配置文件中的转发请求到服务器组的 location 中包含 [`health_check`](http://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check) 指令。此外，服务器组还需要用 [`zone`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone) 指令动态配置：
```
http {
    upstream backend {
        zone backend 64k;

        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
        server backend4.example.com;
    }


    server {
        location / {
            proxy_pass http://backend;
            health_check;
        }
    }
}
```
上面的配置定义了一个服务器组和一台包含一个将请求转发到这个服务器组的 location 的虚拟服务器。同时，通过使用默认参数使能了健康监测功能。这种情况下，Nginx 每 5 秒钟会发送 “/ ” 请求到服务器组中的每台服务器。如果发生任何通信错误或超时（或代理服务器返回 2XX 和 3XX 以外的状态码），对这台服务器的健康监测失败。Nginx 会停止向健康监测失败的服务器发送客户端的请求，直到这台服务器通过健康监测为止。
[`zone`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone) 指令定义了一个在工作进程之间 [共享的内存区域](http://nginx.org/en/docs/http/load_balancing.html#shared)，用于存储服务器组的配置。这使工作进程能够使用同一组计数器来跟踪组中服务器的响应。`zone` 指令还使服务器组[动态可配置](http://nginx.org/en/docs/http/ngx_http_upstream_conf_module.html)。
这个特征可以被 [`health_check`](http://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check) 指令覆盖：
```
location / {
    proxy_pass http://backend;
    health_check interval=10 fails=3 passes=2;
}
```
这里，使用 `interval` 参数将两个连续的健康检查之间的间隔时间增加到 10 秒。此外，通过设置 fails=3，在连续 3 次失败的健康检查之后，服务器将被认为是不健康的。最后，使用 `passes` 参数，不健康的服务器需要连续通过 2 次健康监测才能再次被认为是健康的。
可以将一个特殊的 URI 设置为健康监测的请求。使用 `uri` 参数指定这个特殊的 URI：
```
location / {
    proxy_pass http://backend;
    health_check uri=/some/path;
}
```
这个特定的 URI 将会被添加到 `upstream` 指令中的服务器的域名或 IP 地址之后。例如，对应上面例子中声明的 backend 服务器组，健康监测的 URI 类似：http://backend1.example.com/some/path URI。
最后，可以设置一个健康监测的响应应该满足的定制条件。定制条件在 [match](http://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#match) 块中指定，在 `health_check` 指令中通过 match 参数使用指定的这个 match 块。
```
http {
    # ...

    match server_ok {
        status 200-399;
        body !~ "maintenance mode";
    }


    server {
        # ...


        location / {
            proxy_pass http://backend;
            health_check match=server_ok;
        }
    }
}
```
对于这个健康监测，如果响应状态码在 200 至 399 之间，并且报文的报文体不满足指定的正则表达式时，通过监测。

`match` 指令允许 Nginx 检查响应的状态，头部字段，报文体。用这个指令可以验证状态是否在指定范围内，响应是否包含 header，header 和 body 是否匹配正则表达式。`match` 指令可以包含一个 status 条件、一个 body 条件和多个 header 条件。对应于匹配块，响应必须满足指定的所有条件。

例如，下面的 `match` 指令会查找响应码是 200，包含值为 text/html 的 Content-Type 头字段，并且 body 中含有“Welcome to nginx!”的响应：
```
match welcome {
    status 200;
    header Content-Type = text/html;
    body ~ "Welcome to nginx!";
}
```
下面的例子中，感叹号（！）表示取反，状态码不是 301，302，303，307，并且 header 中不包含 Refresh 字段。
```
match not_redirect {
    status ! 301-303 307;
    header ! Refresh;
}
```
非 HTTP 协议也可以开启健康监测，比如 FastCGI，uwsgi，SCGI 和 memcached。
#10. 多个工作进程之间共享数据
如果 [`upstream`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream) 指令中不包含 [`zone`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone) 指令，每个工作进程都保留自己的服务器组配置副本，并维护自己的一组相关计数器。计数器包含服务器组中每台服务器的当前连接数，和每台服务器的失败请求次数。因此，服务器组的配置不可更改。
如果 `upstream` 指令中包含 `zone` 指令，服务器组的配置文件被所有的工作进程共享。这个配置是动态配置的，因为工作进程访问服务器组配置的同一个副本，并使用相同的相关计数器。
`zone` 指令对于 [健康检查](http://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check) 和服务器组的 [即时重新配置](http://nginx.org/en/docs/http/ngx_http_upstream_conf_module.html) 是必需的。然而，服务器组的其他特性也可以受益于此指令的使用。
例如，如果一个服务器组的配置是非共享的，每个工作进程会对所有服务器的失败请求维护它自己的计数器（参考 [`max_fails`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#max_fails) 参数）。这种情况下，每个请求只对应一个工作进程。当处理请求失败时只有这个请求对应的工作进程知道这件事，其他工作进程对此没有感知。结果就是，部分工作进程认为服务器不可用，而其他工作进程仍然向这台服务器发送请求。对于一台明确不可用的服务器，`max_fails` 乘以工作进程失败的尝试次数应会在设定的时间 `fail_timeout`（For a server to be definitively considered unavailable, max_fails multiplied by the number of workers processes of failed attempts should happen within the timeframe set by fail_timeout. ）。另一方面，`zone` 指令保证了预期的行为。
没有 `zone` 指令的话，`least_conn` 负载均衡方法可能不会按照预期工作，至少在小负载下。这个 tcp 和 http 负载均衡方法将请求发送到活跃连接数最小的服务器。同样，如果服务器组的配置不共享，每个进程都使用自己的计数器来计算连接的数量。如果一个工作进程将请求传递给服务器，另一个工作进程也可以将请求传递给同一服务器。但是，可以增加请求数量以减少这种影响。在高负载的情况下，请求在工作进程之间均匀分配，`least_conn` 负载平衡方法会按照预期工作。
##设置共享内存空间 zone 的大小
由于使用场景不同，没有确切的设置。每个特性，如 sticky cookie /route/learn 负载平衡、健康检查或重新解析（re-resolving）都会影响 `zone` 的大小。

例如，256 kb 的 `zone` 与 sticky_route 会话保持方法和一次健康检查的情况下可容纳：

- 128 台服务器 (通过指定 IP 和端口增加单个对等点，adding a single peer by specifying IP:port)
- 88 台服务器 (通过指定域名和端口增加单个对等点，域名可解析为单独的 IP)
- 12 台服务器 (通过指定域名和端口增加单个对等点，域名可解析为多个 IP)
#11. 通过 DNS 配置负载均衡
可以在运行中通过使用 DNS 编辑服务器组的配置。
Nginx Plus 可以监控对应一个服务器域名的 IP 地址所发生的变化，并且自动在 Nginx 中应用这些更改，不需重启。这可以通过在 [`http`](http://nginx.org/en/docs/http/ngx_http_core_module.html#http) 块中指定的 [`resolver`](http://nginx.org/en/docs/http/ngx_http_core_module.html#resolver) 指令和服务器组中 [`server`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#server) 指令的  [`resolve`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#resolve) 参数来完成：
```
http {
    resolver 10.0.0.1 valid=300s ipv6=off;
    resolver_timeout 10s;

    server {
        location / {
            proxy_pass http://backend;
        }
    }


    upstream backend {
        zone backend 32k;
        least_conn;
        # ...
        server backend1.example.com resolve;
        server backend2.example.com resolve;
    }
}
```
这个例子中，[`server`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#server) 指令的  [`resolve`](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#resolve) 参数会定期重新解析（re-resolve） backend1.example.com 和 backend2.example.com 服务器为 IP 地址。默认情况下，Nginx 根据 TTL 重新解析 DNS 记录，但 TTL 值可以用 resolver 指令的有效参数覆盖，在我们的例子中为 5 分钟。
可选的 `ipv6=off` 参数允许只解析 IPv4 地址，虽然 IPv4 和 IPv6 解析都支持。
如果域名解析为多个IP地址，则地址将保存到 upstream 配置并进行负载平衡。 在我们的示例中，服务器将根据 [least_conn](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#least_conn) 负载平衡方法进行负载平衡。 如果一个或多个 IP 地址已被更改或添加/删除，则服务器将被重新平衡。
#12. Load Balancing of Microsoft Exchange Servers
Microsoft Exchang 是邮件服务端，不用。
#13. 即时配置（On-the-Fly Configuration）
使用 Nginx Plus 时，服务器组的配置可以通过 API 实时定义。通过 API，可以查看所有服务器或服务器组中指定的服务器，为一台指定的服务器定义参数，添加或删除服务器。更多信息查看 [Nginx 负载均衡 - 实时配置](https://nginx.com/admin-guide/nginx-load-balancing-api)。