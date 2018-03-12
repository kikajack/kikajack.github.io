[原文地址](https://www.nginx.com/resources/admin-guide/restricting-access-auth-request/)

#1. 概述
Nginx 和 Nginx Plus 可以使用外部服务器或服务对网站的每个请求进行身份验证。为了执行认证，Nginx 向用于验证子请求的外部服务器发出 HTTP 子请求。 如果子请求返回 2xx 响应码，则允许访问，如果返回 401 或 403，则访问被拒绝。 这种类型的认证允许实施各种认证方案，例如多因素认证（multifactor authentication），或允许实施 LDAP 或 OAuth 认证。
#2. 先决条件 Prerequisites
- NGINX Plus 或 NGINX 开源版本。
- 外部认证服务器或认证服务。
#3. 配置 Nginx 或 Nginx Plus
##3.1 添加模块
确保 Nginx 在编译时开启了 `with-http_auth_request_module` 配置选项。运行命令检查输出是否包含这个 `http_auth_request_module` 模块： 
```
$ nginx -V 2>&1 | grep -- 'http_auth_request_module'
```
##3.2 指定认证 location
在需要认证请求的 location 中，通过 [`auth_request`](http://nginx.org/en/docs/http/ngx_http_auth_request_module.html#auth_request) 指令指定一个用于处理认证子请求的内部 location：
``` 
location /private/ {
    auth_request /auth;
    ...
}
```
对于所有路径中含 /private 的请求，会触发一个到内部的 /auth 这个 location 的子请求。
##3.3 实现认证 location
指定内部 location 并在这个 location 中使用 [`proxy_pass`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) 指令会把所有的认证子请求转发到认证服务器或认证服务：
```
location = /auth {
    internal;
    proxy_pass http://auth-server;
    ...
}
```
##3.4 忽略请求体
由于认证子请求忽略请求体，需要将 [`proxy_pass_request_body`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass_request_body) 指令设置为 off，并将 Content-Length 标头设置为空字符串 null：
```
location = /auth {
    internal;
    proxy_pass              http://auth-server;
    proxy_pass_request_body off;
    proxy_set_header        Content-Length "";
    ...
}
```
##3.5 发送完整的原始请求 URI 与参数
通过 [`proxy_set_header`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header) 指令将完整的原始请求 URI 与参数一起传递：
```
location = /auth {
    internal;
    proxy_pass              http://auth-server;
    proxy_pass_request_body off;
    proxy_set_header        Content-Length "";
    proxy_set_header        X-Original-URI $request_uri;
}
```
##3.6 
可以通过 [`auth_request_set`](http://nginx.org/en/docs/http/ngx_http_auth_request_module.html#auth_request_set) 指令设置基于子请求结果的变量值：
```
location /private/ {
    auth_request        /auth;
    auth_request_set $auth_status $upstream_status;
}
```
#4. 完整示例
这个例子将上面的步骤总结到一个配置中：
```
http {
    ...
    server {
    ...
        location /private/ {
            auth_request     /auth;
            auth_request_set $auth_status $upstream_status;
        }

        location = /auth {
            internal;
            proxy_pass              http://auth-server;
            proxy_pass_request_body off;
            proxy_set_header        Content-Length "";
            proxy_set_header        X-Original-URI $request_uri;
        }
    }
}
```