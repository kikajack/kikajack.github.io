[原文地址](https://www.nginx.com/resources/admin-guide/restricting-access-auth-basic/)

#1. 概述
可以通过 用户名加密码 授权机制，限制对整个网站或网站的某些部分的访问。用户名和密码从一个文件获取，这个文件可以通过密码文件创建工具创建和填充，例如 apache2-utils。
HTTP 基本认证可以和其他的访问限制方法结合使用，例如通过 IP 地址或地理位置限制访问。
#2. 先决条件 Prerequisites
- NGINX Plus or NGINX Open Source
- 密码文件创建工具，例如 apache2-utils
#3. 创建密码文件
##3.1 安装 apache2-utils
确保已经安装了 apache2-utils。
##3.2 创建密码文件和第一个用户
用 -c 标志运行 htpasswd 工具，输入文件的路径作为第一个参数，用户名作为第二个参数：
```
$ sudo htpasswd -c /etc/apache2/.htpasswd user1
```
按下回车后，需要两次输入这个用户的密码。
##3.3 创建另外一组用户密码对
此时，不需要 -c 标志：
```
$ sudo htpasswd /etc/apache2/.htpasswd user2
```
##3.4 查看文件内容
```
$ cat /etc/apache2/.htpasswd
```
文件中包含每个用户的用户名和加密后的密码：
```
user1:$apr1$/woC1jnP$KAh0SsVn5qeSMjTtn0E9Q0
user2:$apr1$QdR8fNLT$vbCEEzDj7LyqCMyNpSoBh/
user3:$apr1$Mr5A0e.U$0j39Hp5FfxRkneklXaMrr/
```
#4. 为 HTTP 基本认证配置 Nginx
在要保护的 location 中，指定 [`auth_basic`](http://nginx.org/en/docs/http/ngx_http_auth_basic_module.html#auth_basic) 指令并用需要密码保护的区域名称作为参数。这个区域名称会在需要认证时的用户名/密码对话框中显示：
```
location /status {                                       
    auth_basic “Administrator’s Area”;
    ....
}
```
用包含用户名密码对的 .htpasswd 文件的路径作为 [`auth_basic_user_file`](http://nginx.org/en/docs/http/ngx_http_auth_basic_module.html#auth_basic_user_file) 指令的参数:
```
location /status {                                       
    auth_basic           “Administrator’s Area”;
    auth_basic_user_file /etc/apache2/.htpasswd; 
}
```
另外，可以对整个网站设置为需要 HTTP 基本认证，而部分页面可以不需认证。通过为 [`auth_basic`](http://nginx.org/en/docs/http/ngx_http_auth_basic_module.html#auth_basic) 指令设置 `off` 参数，可以在对应的上下文中（location 等） 从上级配置取消继承：
```
server {
    ...
    auth_basic           "Administrator’s Area";
    auth_basic_user_file conf/htpasswd;

    location /public/ {
        auth_basic off;
    }
}
```
#5. 通过 IP 地址将基本认证结合到访问限制
HTTP 基本认证可以有效地与IP地址访问限制相结合。至少可以实现两个场景：

- 一个用户需要使用有效的 IP 地址且经过认证。
- 一个用户需要使用有效的 IP 地址或者经过认证。

使用 [nignx access](http://nginx.org/en/docs/http/ngx_http_access_module.html) 模块的 [`allow`](http://nginx.org/en/docs/http/ngx_http_access_module.html#allow) 和 [`deny`](http://nginx.org/en/docs/http/ngx_http_access_module.html#deny) 指令可以允许或拒绝来自指定 IP 地址的访问：
```
location /status {
    ...
    deny 192.168.1.2;
    allow 192.168.1.1/24;
    allow 127.0.0.1;
    deny all;
}
```
来自网络段 192.168.1.1/24 且地址不是 192.168.1.2 的访问将被允许。注意，allow 和 deny 指令会按照定义的顺序执行。

`satisfy` 指令可以把 IP 和 HTTP 认证两个限制结合起来。如果参数设置为 all，则客户端需要同时满足两个条件。如果设置为 any，则客户端至少满足一个条件即可：
```
location /status {
    ...
    satisfy all;    

    deny  192.168.1.2;
    allow 192.168.1.1/24;
    allow 127.0.0.1;
    deny  all;

    auth_basic           "Administrator’s Area";
    auth_basic_user_file conf/htpasswd;
}
```
#6. 完整示例
这个例子展示了如何结合 HTTP 认证和 IP 地址来保护 /status 区域：
```
http {
    server {
        listen 192.168.1.23:8080;
        root   /usr/share/nginx/html;

        location /status {
            status;
            satisfy all;

            deny  192.168.1.2;
            allow 192.168.1.1/24;
            allow 127.0.0.1;
            deny  all;

            auth_basic           “Administrator’s area;
            auth_basic_user_file /etc/apache2/.htpasswd; 
        }

        location = /status.html {
        }
    }
}
```
如果用户输入的地址和 /status 页面相关，首先会弹出输入密码对话框：
![auth_required](http://img.blog.csdn.net/20180219113056448?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
如果输入的用户名和密码不匹配密码文件中的任何一条记录，会得到 401 需要授权错误。