[原文地址](https://www.nginx.com/resources/admin-guide/ip-blacklisting/)

本文讲述了如何开启并动态维护一个 IP 地址黑名单。
#1. 概述
使用 NGINX Plus [R13](https://www.nginx.com/resources/admin-guide/nginx-plus-releases/#r13) 时，可以将某些 IP 地址列入黑名单，可以创建并维护黑名单 IP 地址的数据库。相对的，还可以将某些 IP 地址明确列入白名单。 IP地址数据库使用 NGINX Plus [API](http://nginx.org/en/docs/http/ngx_http_api_module.html?_ga=2.87629840.904408446.1519087829-413045118.1519087829) 和 Nginx Plus [keyval](http://nginx.org/en/docs/http/ngx_http_keyval_module.html?_ga=2.263661188.904408446.1519087829-413045118.1519087829) 模块进行管理。
#2. 先决条件 Prerequisites
- NGINX Plus [R13](https://www.nginx.com/resources/admin-guide/nginx-plus-releases/#r13)
#3. 设置
首先，通过 Nginx [keyval](http://nginx.org/en/docs/http/ngx_http_keyval_module.html?_ga=2.263661188.904408446.1519087829-413045118.1519087829) 模块开启数据库以存储 IP 地址的黑名单或白名单。
##3.1 设置存储区域 zone
在 Nginx 配置文件中，开启用于储存关键字和值的区域，例如，1 MB 大小的区域 one。在 [http](http://nginx.org/en/docs/http/ngx_http_core_module.html?&_ga=2.263334916.904408446.1519087829-413045118.1519087829#http) 中使用 [`keyval_zone`](http://nginx.org/en/docs/http/ngx_http_keyval_module.html?&_ga=2.259639170.904408446.1519087829-413045118.1519087829#keyval_zone) 指令可以开启：
```
http {
    ...
    keyval_zone zone=one:1m;
}
```
##3.2 设置文件
在 [`keyval_zone`](http://nginx.org/en/docs/http/ngx_http_keyval_module.html?&_ga=2.259639170.904408446.1519087829-413045118.1519087829#keyval_zone) 指令中，可以指定一个用于保存键值数据库的文件，并且在 Nginx 重启时文件保持不变。例如 one.keyval：
```
keyval_zone zone=one:1m state=one.keyval;
```
##3.3 设置 Nginx 模式
通过 [`api`](http://nginx.org/en/docs/http/ngx_http_api_module.html?&_ga=2.29894839.904408446.1519087829-413045118.1519087829#api) 指令将 Nginx API 使能为读写模式：
```
...
    server {
        listen 80;
        server_name www.example.com;

        location /api {
            api write=on;
        }
    }
```
##3.4 限制访问
强烈建议，对这个 location [限制访问](https://www.nginx.com/resources/admin-guide/restricting-access/#restrict)，例如只允许来自 127.0.0.1 的访问：
```
...
    server {
        listen 80;
        server_name www.example.com;

        location /api {
            api write=on;
            allow 127.0.0.1;
            deny  all;
        }
    }
```
##3.5 填充键值数据库
通过以 JSON 格式发送 [POST API 命令](http://nginx.org/en/docs/http/ngx_http_api_module.html?&_ga=2.264168452.904408446.1519087829-413045118.1519087829#postHttpKeyvalZoneData) 来填充键值数据库。 可以通过 CURL 发送这个命令。 如果区域 zone 为空，则可以一次输入多个键值对，如果区域中已经有一个或多个键值对，则只能添加一对：
```
curl -X POST -d '{
   "10.0.0.1": "1",
   "10.0.0.2": "1",
   "10.0.0.3": "0",
   "10.0.0.4": "0"
 }' -s http://www.example.com/api/1/http/keyvals/one
```
##3.6 创建映射关系
在 [http](http://nginx.org/en/docs/http/ngx_http_core_module.html?&_ga=2.263334916.904408446.1519087829-413045118.1519087829#http) 中使用 [`keyval`](http://nginx.org/en/docs/http/ngx_http_keyval_module.html?&_ga=2.264168452.904408446.1519087829-413045118.1519087829#keyval) 指令可以创建用户 IP 地址与键值对的映射。该指令将在键值数据库（用 zone = parameter 指定）中创建一个新变量（用指令的第二个参数指定），该变量的值由键（用指令的第一个参数指定）查找：
```
http {
    ...
    keyval_zone zone=one:1m state=one.keyval;
    keyval $remote_addr $target zone=one; # Client address is the key, $target is the value;
}
```
##3.6 创建映射关系
用 if 指令创建允许或拒绝 IP 地址的规则：
```
if ($target) {
    return 403;
}
```
#4. 管理键值数据库
可以用 API 命令动态热更新 IP 地址数据库，不需要 reload 重新加载 Nginx Plus。
发送下面的 curl 命令可以获取某个 zone 中的所有数据库入口的列表：
```
curl -X GET http://www.example.com/api/1/http/keyvals/one
```
可以发送下面的 curl 命令更新一个已经存在的入口，例如把 IP 地址为 10.0.0.4 的值从 allow 改为 deny：
```
curl -X PATCH -d '{"10.0.0.4": "1"}' -s http://www.example.com/api/1/http/keyvals/one
```
可以发送下面的 curl 命令删除一个已经存在的入口：
```
curl -X DELETE -d '{"10.0.0.4": "1"}' -s http://www.example.com/api/1/http/keyvals/one
```
#5. 完整示例
一个 Nginx 配置文件：
```
http {
    ...
    keyval_zone zone=one:1m state=one.keyval;
    keyval $remote_addr $target zone=one;

    server {
        listen 80;
        server_name www.example.com;


        location /api {
            api write=on;
            allow 127.0.0.1;
            deny  all;
        }


        if ($target) {
            return 403;
        }
    }
}
```
用 curl 命令填充黑名单（value 1）和白名单（value 0）到空 zone：
```
curl -X POST -d '{
   "10.0.0.1": "1",
   "10.0.0.2": "1",
   "10.0.0.3": "0",
   "10.0.0.4": "0"
 }' -s http://www.example.com/api/1/http/keyvals/one
```
这个例子的配置详情如下：

- 创建了 1 MB 大小且名为 one 的 keyval 区域，创建了 one.keyval 文件。都可以存储键值对。
- Nginx Plus API 设置为写模式，因此 keyval 区域可以用 IP 地址填充。
- 通过发送 curl API 命令，用黑名单和白名单的 IP 地址的键和值填充 keyval 区域，其中来自黑名单的 IP 对应值是 1，来自白名单的 IP 对应值是 0。
- 用 `$remote_addr` 作为键在键值数据库中查找 IP 地址，并将找到的键对应的值放入 `$target` 变量。
- 设置简单规则来检查结果值：如果 `$target` 变量值是 1（IP 黑名单），返回 403 错误到客户端。
#6. See Also
- [结合 NGINX Plus 和 fail2ban 的 IP 地址动态黑名单](https://www.nginx.com/blog/dynamic-ip-blacklisting-with-nginx-plus-and-fail2ban/)