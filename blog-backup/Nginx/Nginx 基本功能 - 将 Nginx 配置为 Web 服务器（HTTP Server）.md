[原文地址](https://www.nginx.com/resources/admin-guide/nginx-web-server/)

将 Nginx 配置为 Web 服务器，实际上就是指明服务器处理哪些 HTTP 请求（用 URL 区分）。需要在配置文件中定义虚拟服务器，用于处理对特定域名或 IP 地址的请求。
针对 HTTP 流量的每个虚拟服务器都要通过定义 location 来处理特定的 URI 请求或返回文件。另外，可以修改 URI，以便将请求重定向到其他 location 或虚拟服务器。 此外，还可以返回特定的错误代码，可以为每个错误代码配置特定的页面。
#1. 设置虚拟服务器 server
Nginx 配置文件中至少要包含一个 server 指令定义的虚拟服务器。在处理每个请求之前，需要先选中一个 server。
在 http 上下文中通过 server 来定义虚拟服务器，可以定义多个：
```
http {
    server {
        # Server1 configuration
    }
    server {
        # Server2 configuration
    }
}
```
在 server 的配置块中，通常包含 listen 指令指定要监听的 IP 地址和端口号 (or Unix domain socket and path)。IPv4 和 IPv6 地址都可以。
```
server {
    listen 127.0.0.1:8080; # 监听本机发出的，连到 8080 端口的请求
    # The rest of server configuration
}
```
参数省略端口号时，使用标准 80 端口。省略地址时，监听所有地址。如果整个 listen 指令都省略了，根据超级用户的权限不同，标准端口是 80/tcp，默认端口是 8000/tcp。
如果有几个虚拟服务器同时匹配到请求的 IP 地址和端口号，Nginx 会用 HTTP 头中的域名（Host 头）匹配服务器配置块中的 `server_name` 指令。`server_name` 指令的参数可以是完整域名，通配符（`*`）或正则表达式。
```
server {
    listen      80;
    server_name example.org www.example.org;
    ...
}
```
如果匹配 Host 头的服务器名有多个，Nginx 会按以下顺序搜索并使用第一个匹配到的服务器：

1. 精确名称搜索，比如 www.baidu.com。
2. 以通配符 * 开头的最长匹配项，比如 *.baidu.com。
3. 以通配符 * 结尾的最长匹配项，比如 mail.*。
4. 第一个匹配到的正则表达式。

如果 Host 头字段匹配不到任何服务器，Nginx 会把请求路由到默认服务器。默认服务器可以通过 `listen` 指令后的 `default_server` 参数来声明，如果没有声明则配置文件中的第一个服务器就成为默认服务器。
```
server {
    listen      80 default_server;
    ...
}
```
#2. 设置位置 locations
Nginx 可以将流量发送到不同的代理服务器，或者根据请求 URI 提供不同的文件。通过在 server 指令中使用 location 指令实现这些功能。
例如，可以在虚拟服务器中定义 3 个 location 块，将一部分请求发到被代理的服务器，另一部分请求发到另一个被代理的服务器，其他请求则映射到本地的文件系统。
Nginx 会根据 location 指令的参数来测试请求的 URI，并使用匹配到的 location 指令。location 块通常可以嵌套，实现对请求的分组处理。
location 指令有两种参数：前缀字符串（路径名）和正则表达式。要匹配前缀字符串的请求 URI 必须以这个前缀字符串开头才会匹配成功。
##2.1 前缀字符串（路径名）
下面的 location 指令参数使用了路径名，请求 URI 要匹配这个 location，就必须以 /some/path/开头，比如 /some/path/document.html。而 /my-site/some/path 就不匹配。
```
location /some/path/ {
    ...
}
```
##2.2 正则表达式
- 以 `~` 开头时，表示区分大小写；
- 以 `~*` 开头时，表示不区分大小写。

下面的例子中，如果请求 URI 中的任何位置出现 .html 或 .htm 就匹配成功：
```
location ~ \.html? {
    ...
}
```
##2.3 location 匹配原则
为了找出最匹配 URI 的 location，Nginx 会先比较前缀字符串（路径名），再比较正则表达式。
正则表达式的优先级较高，除非使用 ^〜 修饰符。在前缀字符串中，Nginx 选择最特定的字符串（即最长和最完整的字符串）。 下面给出了选择处理请求的位置的确切逻辑：

1. 用所有的前缀字符串测试 URI。
2. 等号 `=` 定义了前缀字符串和 URI 的精确匹配关系。如果找到了这个精确匹配，则停止查找。
3. 如果 ^〜（caret-tilde）修饰符预先匹配到最长的前缀字符串，则不检查正则表达式。
4. 存储最长的匹配前缀字符串。
5. 用正则表达式测试 URI。
6. 匹配到第一个正则表达式后停止查找，使用对应的 location。
7. 如果没有匹配到正则表达式，则使用之前存储的前缀字符串对应的 location。

等号 `=` 最常见的用途就是 `/` (forward slash)。如果对 / 的请求很多，可以把 `= / ` 指定为 location 的参数来提高响应速度。因为在第一次比较之后就停止搜索匹配。
```
location = / {
    ...
}
```
##2.4 root 指令和 proxy_pass 指令
location 上下文中可以定义处理请求（提供静态文件或转发请求给被代理的服务器）的指令。
```
server {
  # 响应静态文件
    location /images/ {
        root /data;
    }
  # 转发请求到 http://www.example.com
    location / {
        proxy_pass http://www.example.com;
    }
}
```
###2.4.1 root 指令
root 指令指定静态文件在文件系统中的路径。请求的 URI 中，和 location 相关的部分会和 root 指定的路径组合成完整的目录名和文件名，从而获取到静态文件。例如，上面的例子中对 `/images/example.png` 的请求，会对应文件 `/data/images/example.png`。

###2.4.2 proxy_pass 指令
proxy_pass 指令将请求转发给和 URL 相关联的被代理的服务器，然后把被代理的服务器的响应转发给客户端。上面的例子中，所有 URI 不以 /images/ 开头的请求，都会被转发给被代理的服务器。
#3. 使用变量
可以通过在配置文件中使用变量，实现不同的情况下用不同的方式处理请求。变量是在运行时计算的有名字的值，用作指令的参数。Nginx 中的变量用美元符号 `$` 开头。变量根据 Nginx 的状态定义信息，例如当前正在处理的请求的属性。

Nginx 中有一系列的预定义的变量，比如 [core HTTP 变量](http://nginx.org/en/docs/http/ngx_http_core_module.html#variables)。还可以通过 [set](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#set)、[map](http://nginx.org/en/docs/http/ngx_http_map_module.html#map) 和 [geo](http://nginx.org/en/docs/http/ngx_http_geo_module.html#geo) 指令来自定义变量。多数变量在运行时才计算值，并包含针对特定请求的信息。例如，`$remote_addr` 包含客户端的 IP 地址信息，`$uri` 包含当前的 URI 值。
#4. return 指令，返回指定的状态码（Status Codes）
有的网页的 URI 需要立刻返回包含指定错误码或跳转码的响应，比如页面被暂时或永久移动。实现这个目的的最简单方法是使用 return 指令。
```
location /wrong/url {
    return 404;
}
```
return 指令的第一个参数是响应码。第二个参数可选，可以是重定向的 URL（用于响应码 301，302，303，307）或响应体中的文本。
```
location /permanently/moved/url {
    return 301 http://www.example.com/moved/here;
}
```
location 和 server 上下文中都可以使用 return 指令。
#5. 重写请求的 URI
通过 rewrite 指令，在请求的处理过程中可以多次改写请求的 URI。
rewrite 指令有3个参数（两个必填，一个选填）：第一个参数是用于匹配 URI 的正则表达式，第二个参数是用于替代匹配到的 URI 的新的 URI，第三个参数可选，是一个标志，可以停止处理其他重写指令或发送重定向（代码301或302）。
```
location /users/ {
    rewrite ^/users/(.*)$ /show?user=$1 break;
}
```
在 server 和 location 上下文中可以使用多个 rewrite 指令，Nginx 会按照先后顺序依次执行。在 server 上下文中的 rewrite 指令会在上下文被选中时执行一次。
Nginx 在处理完一系列的 rewrite 指令后，会根据新的 URI 选择一个 location 上下文。如果选中的 location 包含 rewrite 指令，会依次执行。
rewrite 指令示例：
```
server {
    ...
    rewrite ^(/download/.*)/media/(.*)\..*$ $1/mp3/$2.mp3 last;
    rewrite ^(/download/.*)/audio/(.*)\..*$ $1/mp3/$2.ra  last;
    return  403;
    ...
}
```
这个例子会区分两组 URI：/download/some/media/file 这样的 URI 会被变为 /download/some/mp3/file.mp3。因为第三个参数使用了 last 标志，后续指令（第二个重写和返回指令）被跳过，但 Nginx 继续处理请求（该请求现在已经具有不同的 URI）。如果 URI 不匹配 rewrite 指令，Nginx 会向客户端返回 403 错误代码。
有两个参数可以中断 rewrite 指令：

- last – 在当前的 server 或 location 上下文中中断 rewrite 指令的执行。此时 Nginx 会查找要重写 URI 的 location，并且应用新 location 中的任何 rewrite 指令（意味着 URI 可以再次更改）。
- break – 跟 break 指令类似，在当前的上下文中停止 rewrite 指令，停止查找匹配新的 URI 的 location。新的 location 中的 rewrite 指令不会执行。
#6. 重写 HTTP 响应（Rewriting HTTP Responses）
`sub_filter` 指令可以定义如何重写或改变 HTTP 响应的内容，将一个字符串替换为另一个字符串。。这个指令支持变量和链式改变，从而可以实现复杂操作。
示例：
```
location / {
    sub_filter      /blog/ /blog-staging/;
    sub_filter_once off;
}
```
可以改变 HTTP 协议，将 `http://` 改为 `https://`，也可以把请求头字段中的 localhost 改为主机名。`sub_filter_once` 指令控制 Nginx 是否在一个 location 内连续应用 `sub_filter` 指令。
```
location / {
    sub_filter     'href="http://127.0.0.1:8080/'    'href="https://$host/';
    sub_filter     'img src="http://127.0.0.1:8080/' 'img src="https://$host/';
    sub_filter_once on;
}
```
请注意，`sub_filter` 替换最多只会发生一次。如果已使用 `sub_filter` 修改的响应部分再次被另一个 `sub_filter` 匹配，是不会再次被替换的。
#7. 处理错误（Handling Errors）
`error_page` 指令可以配置 Nginx 返回自定义页面以及错误代码，在响应中替换不同的错误代码，或将浏览器重定向到指定 URI。
以下示例中，`error_page` 指令指定要返回 404 错误页面（/404.html）。
```
error_page 404 /404.html;
```
注意 `error_page` 指令并不会立刻返回错误（只有 return 指令才会立刻返回），只是表明如何发生错误时如何处理。错误代码可以来自一个被代理的服务器或在 Nginx 处理过程中发生（比如 Nginx 找不到客户端指定的页面时会 404）。

下面例子中，当 Nginx 找不到页面，会把 404 代码改为 301，并把客户端重定向到 `http:/example.com/new/path.html`。
```
location /old/path.html {
    error_page 404 =301 http:/example.com/new/path.html;
}
```
下面的配置会在文件找不到时把请求转发到后端。由于在 `error_page` 指令的等号后没有指定状态码，因此对客户端的响应的状态码由代理服务器返回（不一定是404）。
```
server {
    ...
    location /images/ {
        # Set the root directory to search for the file
        root /data/www;

        # Disable logging of errors related to file existence
        open_file_cache_errors off;

        # Make an internal redirect if the file is not found
        error_page 404 = /fetch$uri;
    }

    location /fetch/ {
        proxy_pass http://backend/;
    }
}
```
`error_page` 指令指示 Nginx 在文件找不到时做内部跳转。`error_page` 指令的最后一个参数中可以使用 `$uri` 变量，代表当前的请求中的 URI，该请求将在重定向中传递。
例如，如果文件  /images/some/file 找不到，会被替换为 /fetch/images/some/file 并从第一个 location 开始再次查找。最终，匹配第二个 location 上下文并跳转到代理服务器。
`open_file_cache_errors` 指令可防止在找不到文件时写入错误消息。这里因为找不到文件时立刻处理了，所以没有必要使用。