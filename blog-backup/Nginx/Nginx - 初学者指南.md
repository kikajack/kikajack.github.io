[原文地址](http://nginx.org/en/docs/beginners_guide.html)
[NGINX 和 NGINX PLUS 特征对比](https://www.nginx.com/products/feature-matrix/)
Nginx 开源版本官网：http://nginx.org
Nginx Plus 企业版官网：https://www.nginx.com

#1. 启动，停止和重新加载配置（Starting, Stopping, and Reloading Configuration）
不同的 Linux 发行版，Nginx 启动方式不同，对于 CentOS7 的命令是 `systemctl start nginx`。一旦 nginx 启动，就可以通过调用 `nginx -s signal` 来控制。使用以下语法：
```
nginx -s signal
```
signal 可以是下列之一：

- stop - 快速结束进程，**不管是否有连接，直接关闭进程**。
- quit - 平滑结束进程，**如果有连接则等完成当前所有的请求**，并在断开连接后再关闭进程。
- reload - 重新加载配置文件。**可以在加载前自动检查配置文件，没问题的话再平滑加载。一旦主进程（Master process）接收到重新加载配置的信号，它将检查新配置文件的语法有效性，并尝试应用其中提供的配置。如果配置文件没有问题，则主进程启动新的工作进程（Work process），并发送消息给旧工作进程，请求关闭。否则，主进程回滚更改并继续使用旧配置。旧工作进程接收到关闭命令后，停止接受新的连接，并继续服务当前的请求，直到所有已连接请求服务完成后，旧工作进程退出。**
- reopen - 重新打开日志文件。

要停止 Nginx 进程并等待工作进程完成当前请求，可以执行以下命令：
```
nginx -s quit
```
这个命令应该在启动 Nginx 的同一个用户下执行。
Nginx 配置文件更改后，需要重启 Nginx `systemctl restart nginx`（CentOS7 的重启命令） 或重新加载配置 `nginx -s reload`。

也可以通过 Unix 下的工具（比如 kill 程序）给 Nginx 进程发信号。此时通过指定的进程 ID 来发送信号。nginx主进程的进程ID默认写入到 nginx.pid目录 /usr/local/nginx/logs或者 /var/run。例如，如果主进程ID是 1628，要发送 QUIT 信号正常关闭 nginx，可以执行：
```
kill -s QUIT 1628
```
可以通过 Linux 的 ps 程序查看所有正在运行的 nginx 进程的列表。例如：
```
ps -aux | grep nginx
```
关于将信号发送到 nginx 的详细资料请参阅 [控制 nginx](http://nginx.org/en/docs/control.html)。
#2. 配置文件的结构
Nginx 由模块组成，模块由配置文件中指定的指令控制。指令分为简单指令和块指令（block directives）。简单指令由名称和参数组成，以空格分隔，并以分号（;）结束。块指令不以分号结尾，而是用一系列由大括号（{}）包围的附加指令来结束。如果块指令的大括号内可以有其他指令，这个块指令就被称为一个上下文（例如： [events](http://nginx.org/en/docs/ngx_core_module.html#events)， [http](http://nginx.org/en/docs/http/ngx_http_core_module.html#http)， [server](http://nginx.org/en/docs/http/ngx_http_core_module.html#server) 和 [location](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)）。

配置文件中，如果指令在任何上下文之外，则被认为是在主上下文 [main context](http://nginx.org/en/docs/ngx_core_module.html)中。events 和 http 指令在主上下文 main context 中，而 server 指令在 http 中，location 指令在 server 中。

配置文件中，所有注释行用井号开头（`#`）。
#3. 提供静态内容
Web 服务器可以提供文件（如图像或静态 HTML 页面）。

>示例：根据请求，文件将从不同的本地目录 /data/www （包含HTML文件）和 /data/images （包含图像）提供。

首先，创建 `/data/www` 目录和 `/data/images`，并将 index.html 和图片放入对应的目录中。

接下来，打开配置文件。默认的配置文件已经包含了几个server块的例子，大部分都是注释掉的。现在注释掉所有这些块并开始一个新 server块：
```
http {
    server {
    }
}
```
通常，配置文件包含几个用侦听端口（`listen 80;`）和服务器名称（`server_name  admin.local.cn local.cn;`）区分的 server 块 。一旦 nginx 决定哪个进程处理请求，它就会根据块内定义的指令的参数来测试请求头中指定的 URI 。
将下面的 location 块添加到 server 块中：
```
location / {
    root /data/www;
}
```
这个 location 块指定用 `/` 与请求中的 URI 进行比较。对于匹配请求（比如 `xx.com/`，`xx.com/test`），URI 将被添加到 [root](http://nginx.org/en/docs/http/ngx_http_core_module.html#root) 指令中指定的路径 ，构建映射到本地文件系统的 `/data/www` 目录的路径。如果 URI 同时匹配多个 location 块，nginx 会根据最长匹配原则，选择最长的前缀。

接下来，添加第二个location块：
```
location /images/ {
    root /data;
}
```
如果 URI 中含有 `/images/`，将会匹配成功。

所得到的配置server块应该是这样的：
```
server {
    location / {
        root /data/www;
    }

    location /images/ {
        root /data;
    }
}
```
这个配置会默认监听 80 端口，可以在本地机器上访问 `http://localhost/`。为了响应 URI 以 `/images/` 开始的请求，服务器将从 `/data/images` 目录发送文件。例如，响应这个 `http://localhost/images/example.png` 请求，nginx会发送这个 /data/images/example.png 文件。如果这个文件不存在，nginx 会发送一个 404 错误的响应。对于不以 `/images/` 开始的请求，将被映射到 `/data/www` 目录上。例如，响应这个 `http://localhost/some/example.html` 请求，nginx 会发送这个 `/data/www/some/example.html` 文件。

要应用新的配置，请启动 nginx（如果尚未启动）或将 reload 信号发送到 nginx 的主进程，执行：
```
nginx -s reload
```
如果 Nginx 不按预期工作，可以通过查看 `/usr/local/nginx/logs` 或 `/var/log/nginx` 目录下的 access.log 和 error.log 日志文件 。
#4. 设置一个简单的代理服务器
Nginx 经常被用作代理服务器。此时 Nginx 代理服务器接收请求并转发给被代理的服务器，然后从中获取响应信息并将其发送给客户端。

我们将配置一个基本的代理服务器，对于图像请求则去本地文件系统查找文件并响应，对于所有其他请求则发送给被代理的服务器。在这个例子中，两个服务器将在一个 Nginx 实例上定义。

首先，通过向 nginx 的配置文件添加一个 server 块来定义代理服务器，server 块中包含以下内容：
```
server {
    listen 8080;
    root /data/up1;

    location / {
    }
}
```
这个简单的服务器，侦听端口 8080 并将所有请求映射到 /data/up1 本地文件系统上的目录。创建这个目录并把 index.html 文件放进去。请注意，当一个请求匹配到的 location 块中没有 root 指令时，会使用 server 上下文中的 root 指令。

接下来，使用上一节中的服务器配置并对其进行修改，使其成为代理服务器配置。在第一个 location 块中，填入 [proxy_pass](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) 指令与参数（包括代理服务器的协议，名称和端口，例如 `http://localhost:8080`）：
```
server {
    location / {
        proxy_pass http://localhost:8080;
    }

    location /images/ {
        root /data;
    }
}
```
修改第二个 location 块，将带有 /images/  前缀的请求映射到 /data/images 目录下的文件，使其与具有指定文件扩展名的图像的请求匹配。修改的 location 块如下所示：
```
location ~ \.(gif|jpg|png)$ {
    root /data/images;
}
```
该参数是一个正则表达式，匹配三种图片：gif、jpg、png。相应的请求将被映射到该 /data/images 目录。
**Nginx 配置文件中的正则表达式以 `~` 开头。**

**当 nginx 选择 location 块来为请求提供服务时，正则匹配优先，最长匹配优先。**首先检查使用字符串（而不是正则表达式）前缀的 [location](http://nginx.org/en/docs/http/ngx_http_core_module.html#location) 指令，并记住匹配到最长字符串前缀的 location，然后检查使用正则表达式前缀的 location。如果 URI 与正则表达式前缀匹配，nginx 会选择该使用正则表达式前缀的 location，否则，会选择之前记住的 location。

代理服务器的配置如下：
```
server {
    location / {
        proxy_pass http://localhost:8080/;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```
该服务器将指定类型的图片请求映射到 /data/images 目录并将所有其他请求传递到代理服务器。

要应用新配置，请将信号 reload 发送到 nginx。

详细资料参考 [配置代理服务器](http://nginx.org/en/docs/http/ngx_http_proxy_module.html)。
#5. 设置 FastCGI 代理，处理动态内容
nginx 可将请求路由到 FastCGI 服务器（FastCGI 服务器可以运行由各种框架和编程语言（如PHP）构建的应用程序）。

与 FastCGI 服务器配合使用的最基本的 nginx 配置包括使用 `fastcgi_pass` 指令来设置 FastCGI 服务器的位置（比如 php-fpm 服务器通常位于  localhost:9000），以及 `fastcgi_param` 指令来设置传递给 FastCGI 服务器的参数。
在 PHP 中，`SCRIPT_FILENAME` 参数用于确定脚本名称，`QUERY_STRING` 参数用于传递请求参数。最终的配置将是：
```
server {
    location / {
        fastcgi_pass  localhost:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param QUERY_STRING    $query_string;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```
这个服务器将除了静态图像请求之外的所有请求路由到监听 localhost:9000 的FastCGI 服务器。