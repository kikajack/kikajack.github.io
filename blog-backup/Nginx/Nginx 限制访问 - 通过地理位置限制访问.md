[原文地址](https://www.nginx.com/resources/admin-guide/restricting-access-geoip/)

#1. 概述
Nginx 可以通过地理位置来区分用户。例如，对于不同国家可以显示不同的页面内容，也可以对指定国家或城市限制内容分发。
Nginx 使用第三方 MaxMind 数据库来匹配用户的 IP 地址及其位置。 只要地理位置已知，就可以在 [map](http://nginx.org/en/docs/http/ngx_http_map_module.html?_ga=2.101311497.904408446.1519087829-413045118.1519087829) 或 [split_clients](http://nginx.org/en/docs/http/ngx_http_split_clients_module.html?_ga=2.100806409.904408446.1519087829-413045118.1519087829) 模块中使用基于 geoip 的变量。
HTTP 和 TCP/UDP 协议都支持基于地理位置的访问限制。
#2. 先决条件 Prerequisites
- 安装了 [http geoip](http://nginx.org/en/docs/http/ngx_http_geoip_module.html?_ga=2.88657555.904408446.1519087829-413045118.1519087829) 或 [stream geoip](http://nginx.org/en/docs/stream/ngx_stream_geoip_module.html?_ga=2.88657555.904408446.1519087829-413045118.1519087829) 模块的 NGINX Plus 或开源版本的 NGINX
- [MaxMind](http://dev.maxmind.com/geoip/legacy/geolite/) 的 Geolite Legacy 数据库
#3. 配置 GeoIP
##3.1 安装模块
###3.1.1 对于 NGINX Plus
为 Nginx Plus 安装 GeoIP 动态模块：
```
$ apt-get install nginx-plus-module-geoip
```
通过在配置文件主上下文中添加 [`load_module`](http://nginx.org/en/docs/ngx_core_module.html?&_ga=2.25675957.904408446.1519087829-413045118.1519087829#load_module) 指令在 Nginx Plus 中开启 GeoIP 动态模块：
```
load_module modules/ngx_http_geoip_module.so;
load_module modules/ngx_stream_geoip_module.so;
```
###3.1.2 对于 Nginx
确保开源版本的 Nginx 编译安装时开启 [`--with-http_geoip_module`](http://nginx.org/en/docs/http/ngx_http_geoip_module.html?_ga=2.25675957.904408446.1519087829-413045118.1519087829) 或 [`--with-stream_geoip_module`](http://nginx.org/en/docs/stream/ngx_stream_geoip_module.html?_ga=2.25675957.904408446.1519087829-413045118.1519087829) 配置标志：
```
$ nginx -V 2>&1 | grep -- 'http_geoip_module'
$ nginx -V 2>&1 | grep -- 'stream_geoip_module'
```
或者确保这几个模块可以 [动态链接](https://www.nginx.com/resources/admin-guide/installing-nginx-open-source/#modules_dynamic)。
##3.2 下载 MaxMind 的地理位置数据库
从 [MaxMind 下载页面](http://dev.maxmind.com/geoip/legacy/geolite/) 下载并解压缩 legacy Geo 国家和城市数据库：
```
$ wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz
$ wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz
$ gunzip GeoIP.dat.gz
$ gunzip GeoLiteCity.dat.gz
```
##3.3 配置地理位置数据库的路径
通过 http 的 [`geoip_country`](http://nginx.org/en/docs/http/ngx_http_geoip_module.html?&_ga=2.100200214.904408446.1519087829-413045118.1519087829#geoip_country) 和 [`geoip_city`](http://nginx.org/en/docs/http/ngx_http_geoip_module.html?&_ga=2.33681193.904408446.1519087829-413045118.1519087829#geoip_city) 指令，或 stream 的 [`geoip_country`](http://nginx.org/en/docs/stream/ngx_stream_geoip_module.html?&_ga=2.33681193.904408446.1519087829-413045118.1519087829#geoip_country) 和 [`geoip_city`](http://nginx.org/en/docs/stream/ngx_stream_geoip_module.html?&_ga=2.33681193.904408446.1519087829-413045118.1519087829#geoip_city) 指令添加数据库路径到 Nginx 的配置文件：
```
http {
    ...
    geoip_country GeoIP/GeoIP.dat;
    geoip_city    GeoIP/GeoLiteCity.dat;
    ...
}
```
或：
```
stream {
    ...
    geoip_country GeoIP/GeoIP.dat;
    geoip_city    GeoIP/GeoLiteCity.dat;
    ...
}
```
##3.4 
使用 [`geoip_country`](http://nginx.org/en/docs/http/ngx_http_geoip_module.html?&_ga=2.25763893.904408446.1519087829-413045118.1519087829#geoip_country) 和 [`geoip_city`](http://nginx.org/en/docs/http/ngx_http_geoip_module.html?&_ga=2.25763893.904408446.1519087829-413045118.1519087829#geoip_city) 指令的变量把数据传到 [`map`](http://nginx.org/en/docs/http/ngx_http_map_module.html?_ga=2.25763893.904408446.1519087829-413045118.1519087829) 或 [`split_clients`](http://nginx.org/en/docs/http/ngx_http_split_clients_module.html?_ga=2.25763893.904408446.1519087829-413045118.1519087829) 模块。
例如，使用 [`geoip_city`](http://nginx.org/en/docs/http/ngx_http_geoip_module.html?&_ga=2.25763893.904408446.1519087829-413045118.1519087829#geoip_city) 指令的变量 `$geoip_city_continent_code` 和 [`map`](http://nginx.org/en/docs/http/ngx_http_map_module.html?_ga=2.25763893.904408446.1519087829-413045118.1519087829) 模块，可以创建另一个变量，其值将成为基于大陆位置的最接近的服务器：
```
...
map $geoip_city_continent_code $nearest_server {
    default default {};
    EU      eu;
    NA      na;
    AS      as;
    AF      af;
...
```
然后可以根据 `$nearest_server` 变量传入的值选择一台 upstream 服务器：
```
...
server {
    listen 12346;
    proxy_pass $nearest_server;
}

 upstream eu {
    server eu1.example.com:12345;
    server eu2.example.com:12345;
}

upstream na {
    server na1.example.com:12345;
    server na2.example.com:12345;
}
...
```
如果大陆位置是欧洲，那么 `$nearest_server` 变量的值是 eu，连接将会通过 `proxy_pass` 指令传到 eu upstream。
#4. 完整示例
这个例子可以在 http 和 stream 上下文中实现：
```
 # can be either "http {" or "stream {"
    ...
    geoip_country GeoIP/GeoIP.dat;
    geoip_city    GeoIP/GeoLiteCity.dat;

    map $geoip_city_continent_code $nearest_server {
        default default {};
        EU      eu;
        NA      na;
        AS      as;
        AF      af;


    server {
        listen 12346;
        proxy_pass $nearest_server;
    }


     upstream eu {
        server eu1.example.com:12345;
        server eu2.example.com:12345;
    }


    upstream na {
        server na1.example.com:12345;
        server na2.example.com:12345;
    }
}
```
在这个例子中，通过数据库 GeoLiteCity.dat 来检查 IP 地址，并将结果写入 `$geoip_city_continent_code` 变量。Nginx 将会用这个变量值匹配 map 指令中的值，并将自定义变量中的结果以白色表示（ white the result in the custom variable，这个示例中的变量是 `$nearest_server`）。 根据 `$nearest_server` 的值，proxy_pass 指令将选择相应的 upstream 服务器。