[原文地址](https://www.nginx.com/resources/admin-guide/nginx-ssl-termination/)
[OPENSSL入門](http://csc.ocean-pioneer.com/docum/ssl_basic.html)

#1. 配置 HTTPS 服务器
通过给 server 块中的 [`listen`](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen) 指令添加 ssl 参数，再为 location 指定相关的证书和私钥文件，可以配置 HTTPS 服务器：
```
server {
    listen              443 ssl;
    server_name         www.example.com;
    ssl_certificate     www.example.com.crt;
    ssl_certificate_key www.example.com.key;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    ...
}
```
`ssl_certificate` 指令的参数是证书，每一个连接到这台服务器的客户端都会拿到一份拷贝。`ssl_certificate_key` 指令的参数是证书对应的私钥，必须要保护好，同时要允许 Nginx 的主进程访问。证书和私钥可以使用同一个文件：
```
ssl_certificate www.example.com.cert;
ssl_certificate_key www.example.com.cert;
```
证书和私钥使用同一个文件时，文件的访问权限应该被限制，只有证书才会被发送到客户端。


可以通过 [`ssl_protocols`](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols)  指令和 [`ssl_ciphers`](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_ciphers)  指令限制连接，只允许使用了 SSL/TLS. 的加强版和密码的连接。
从 1.0.5 版本起，Nginx 默认使用 `ssl_protocols SSLv3 TLSv1` 和 `ssl_ciphers HIGH:!aNULL:!MD5`。从 1.1.13  版本起，Nginx 默认使用 `ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2`。
老旧的密码设计（the design of older ciphers）经常会被发现漏洞，可以在 Nginx 中关闭这个功能（不幸的是，因为向后兼容的问题，默认配置并不会改这个地方）。请注意，CBC 模式加密可能容易受到一些攻击，特别是 BEAST （见 CVE-2011-3389），最好使用 SSLv3 来避免这些麻烦。
#2. 优化 HTTPS 服务器
SSL 操作会消耗额外的 CPU 资源，尤其是 SSL 握手这一步。有两个办法可以最小化每个客户端的这些操作数：

- 通过开启 keepalive，在一次连接中发送多个请求。
- 重复利用 SSL session 的参数来避免对并发请求和后面的请求进行 SSL 握手。

[`ssl_session_cache`](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_cache) 指令用于配置 SSL 的 session。保存在 SSL 的 session 缓存中的每个 session 都可以在工作进程之间共享。**1 MB 的缓存空间可以储存大约 4000 个 session。**缓存默认储存 5 分钟，可以通过 [`ssl_session_timeout`](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_timeout) 指令增加这个默认时间。
下面的示例中，分配了 10 MB 的共享 session 缓存，存储时间 10 分钟：

worker_processes auto;
```
http {
    ssl_session_cache   shared:SSL:10m;
    ssl_session_timeout 10m;

    server {
        listen              443 ssl;
        server_name         www.example.com;
        keepalive_timeout   70;

        ssl_certificate     www.example.com.crt;
        ssl_certificate_key www.example.com.key;
        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers         HIGH:!aNULL:!MD5;
        ...
    }
}
```
#3. SSL 证书链
[证书链 - 简书](https://www.jianshu.com/p/46e48bc517d0)
[证书链是啥 - 七牛](https://developer.qiniu.com/ssl/kb/3703/the-certificate-chain-is-what)
[What is the SSL Certificate Chain?](https://support.dnsimple.com/articles/what-is-ssl-certificate-chain/)
[What is CA bundle?](https://www.namecheap.com/support/knowledgebase/article.aspx/986/69/what-is-ca-bundle)

CA bundle 是一个包含了证书链中的根证书和中间证书的文件。为每一个域名颁发的证书都有构成 CA bundle 的证书链。

>完整的证书内容一般分为3级，服务端证书-中间证书-根证书，即 end-user certificates， intermediates Certificates 和 root Certificates。
<br/>end-user ：用来加密传输数据的公钥的证书，是 HTTPS 中使用的证书，部署在域名对应的服务器上。
<br/>intermediates：CA 用来认证公钥持有者身份的证书，即确认 HTTPS 使用的 end-user 证书是属于当前域名的证书。
<br/>root：用来认证 intermediates 证书是合法证书的证书。

对于由一个众所周知的证书颁发机构签署的证书，一些浏览器可能会不信任，而另一些浏览器可能没有任何问题地接受了。之所以发生这种情况，是因为颁发机构已经在特定的浏览器中分发了一个中间证书并用它对服务器证书签名，该中间证书不存在于众所周知的可信证书颁发机构的基础上。在这种情况下，证书颁发机构提供了一组证书链，这些证书应该连接到签过名的服务器证书。创建证书链时，服务器证书必须出现在 bundle 之前：
```
$ cat www.example.com.crt bundle.crt > www.example.com.chained.crt
```
用 [`ssl_certificate`](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate) 指令指明证书链文件：
```
server {
    listen              443 ssl;
    server_name         www.example.com;
    ssl_certificate     www.example.com.chained.crt;
    ssl_certificate_key www.example.com.key;
    ...
}
```
如果服务器证书和 bundle 连接时顺序有误，则 Nginx 无法启动并报错如下：
```
SSL_CTX_use_PrivateKey_file(" ... /www.example.com.key") failed
   (SSL: error:0B080074:x509 certificate routines:
    X509_check_private_key:key values mismatch)
```
之所以发生这个错误是因为 Nginx 会试图用私钥和 bundle 的第一个证书而不是服务器证书。

浏览器通常会存储由可信机构签名的中间证书。所以广泛使用的浏览器可能已经有了所需要的中间证书了，从而不会怀疑没有与证书链一起发送的证书。为了确保服务器发送完整的证书链，可以使用 OpenSSL 命令行工具：
```
$ openssl s_client -connect www.godaddy.com:443
...
Certificate chain
 0 s:/C=US/ST=Arizona/L=Scottsdale/1.3.6.1.4.1.311.60.2.1.3=US
     /1.3.6.1.4.1.311.60.2.1.2=AZ/O=GoDaddy.com, Inc
     /OU=MIS Department/CN=www.GoDaddy.com
     /serialNumber=0796928-7/2.5.4.15=V1.0, Clause 5.(b)
   i:/C=US/ST=Arizona/L=Scottsdale/O=GoDaddy.com, Inc.
     /OU=http://certificates.godaddy.com/repository
     /CN=Go Daddy Secure Certification Authority
     /serialNumber=07969287
 1 s:/C=US/ST=Arizona/L=Scottsdale/O=GoDaddy.com, Inc.
     /OU=http://certificates.godaddy.com/repository
     /CN=Go Daddy Secure Certification Authority
     /serialNumber=07969287
   i:/C=US/O=The Go Daddy Group, Inc.
     /OU=Go Daddy Class 2 Certification Authority
 2 s:/C=US/O=The Go Daddy Group, Inc.
     /OU=Go Daddy Class 2 Certification Authority
   i:/L=ValiCert Validation Network/O=ValiCert, Inc.
     /OU=ValiCert Class 2 Policy Validation Authority
     /CN=http://www.valicert.com//emailAddress=info@valicert.com
...
```
上面例子中，s 表示 subject 資料，i 表示 issuer 证书颁发机构。
这个例子中，www.GoDaddy.com 服务器的第一个证书的 issurer 是第二个证书的 subject，第二个证书的 issurer 是第三个证书的 subject，第三个证书是由众所周知的 ValiCert 公司签名，这个公司的证书在浏览器中有。

如果证书的 bundle 没有添加，则只会显示一个服务器证书。
#4. 集成的 HTTP/HTTPS 服务器
可以通过两个 listen 指令使一个服务器使其同时处理 HTTP 和 HTTPS 请求：
```
server {
    listen              80;
    listen              443 ssl;
    server_name         www.example.com;
    ssl_certificate     www.example.com.crt;
    ssl_certificate_key www.example.com.key;
    ...
}
```
>在 0.7.13 及更早的版本中，不能像上面的例子那样为单个侦听套接字选择性地启用SSL，HTTP 或 HTTPS 服务器只能使用 `ssl` 指令单独配置。
#5. 基于名字的 HTTPS 服务器
当两个或更多的 HTTPS 服务器同时监听一个 IP 地址时会有个问题：
```
server {
    listen          443 ssl;
    server_name     www.example.com;
    ssl_certificate www.example.com.crt;
    ...
}

server {
    listen          443 ssl;
    server_name     www.example.org;
    ssl_certificate www.example.org.crt;
    ...
}
```
在这个配置下，浏览器会收到默认服务器的证书。此时  www.example.com 会忽略请求中的 server name。这是 SSL 协议的特征，**SSL 连接在浏览器发送 HTTP 请求之前建立，此时 Nginx 并不知道请求的域名。**因此 Nginx 只能提供默认服务器的证书。

这个问题的最佳解决方案是为每个 HTTPS 服务器指定一个独立的 IP 地址（**同一个 IP 地址上不能有两个 HTTPS 服务器**）：
```
server {
    listen          192.168.1.1:443 ssl;
    server_name     www.example.com;
    ssl_certificate www.example.com.crt;
    ...
}

server {
    listen          192.168.1.2:443 ssl;
    server_name     www.example.org;
    ssl_certificate www.example.org.crt;
    ...
}
```
注意，对于 HTTPS upstreams（[`proxy_ssl_ciphers` ](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ssl_ciphers)/[`proxy_ssl_protocols`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ssl_protocols) 和 [`proxy_ssl_session_reuse`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ssl_session_reuse)） 有一些具体的代理设置可以用来调整 Nginx 和 upstreams 之间的 SSL。可以参考文档 [http proxy](http://nginx.org/en/docs/http/ngx_http_proxy_module.html)。
##匹配多个域名的 SSL 证书
有多个方法可以使一个 IP 地址用于多个 HTTPS 服务器。然而，这些方法都有缺点。

- 使用 SubjectAltName  字段有多个域名的证书。注意这个字段长度是有限制的。
- 使用有通配符域名的证书。通配符域名证书会保护指定域名的所有子域名，比如：*.example.org 会保护 www.example.org，但是不会保护 example.org 和 www.sub.example.org。

这两种方式可以组合使用。一个证书可以在 SubjectAltName  字段同时包含精确的域名和通配符域名。
最好的方式是将多域名证书文件和对应的私钥放在配置文件的 http 上下文中，这样所有的 server 配置段会继承这些配置：
```
ssl_certificate     common.crt;
ssl_certificate_key common.key;

server {
    listen          443 ssl;
    server_name     www.example.com;
    ...
}

server {
    listen          443 ssl;
    server_name     www.example.org;
    ...
}
```
##服务器名称指示
在一个单独的 IP 地址上运行多台 HTTPS 服务器的更通用的解决方案是 [`TLS Server Name Indication`](https://en.wikipedia.org/wiki/Server_Name_Indication) 扩展（SNI, RFC 6066），通过这个扩展可以允许浏览器在 SSL 握手时发送请求的域名，从而知道该为这个连接使用哪一个证书。然而浏览器对 SNI 的支持有限，浏览器从以下版本开始支持：
```
Opera 8.0;
MSIE 7.0 (but only on Windows Vista or higher);
Firefox 2.0 and other browsers using Mozilla Platform rv:1.8.1;
Safari 3.2.1 (Windows version supports SNI on Vista or higher);
and Chrome (Windows version supports SNI on Vista or higher, too).
```
只有域名可以通过 SNI 传递。但是如果请求包含字面 IP 地址，一些浏览器会把服务器的 IP 地址作为它的名称。最好忽略这样的请求。
为了在 Nginx 中使用 SNI，必须要确保在编译 Nginx 二进制文件时和运行时动态链接过程，OpenSSL 库可用。
从 OpenSSL 的 0.9.8f 版本起，构建时指定 `--enable-tlsext` 选项可以使 OpenSSL 支持 SNI。从 OpenSSL 的 0.9.8j 版本起，这个选项默认开启。如果 Nginx 构建时添加了对 SNI 的支持，可以用下面的命令查看：
```
$ nginx -V
...
TLS SNI support enabled
...
```
如果 Nginx 支持 SNI，但是动态链接的 OpenSSL 库却不支持 SNI，那么上面的命令会报错：
```
NGINX was built with SNI support, however, now it is linked
dynamically to an OpenSSL library which has no tlsext support,
therefore SNI is not available
```
#6. 兼容性注意事项
- 从版本 0.8.21 和 0.7.62 起，SNI 的支持状态可以通过 -V 选项查看。
- 从版本 0.7.14 起 listen 指令支持 ssl 参数。在版本 0.8.21 之前，它只能与默认参数一起指定。
- 从版本 0.5.32 起开始支持 SNI。
- 从版本 0.5.6 起开始支持共享的 SSL session 缓存。
- 从版本 0.7.65, 0.8.19 起开始默认的 SSL 协议是 SSLv3, TLSv1, TLSv1.1, and TLSv1.2（需要 OpenSSL 库支持）。之前的 SSL 协议是 SSLv2, SSLv3, and TLSv1。
- 从版本 1.0.5 起，默认的 SSL 密码是 `HIGH:!aNULL:!MD5`。
- 从版本 0.7.65, 0.8.20 and later 起，默认的 SSL 密码是 `HIGH:!ADH:!MD5`。
- 从版本 0.8.19 起，默认的 SSL 密码是 `ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM`。
- 从版本 0.7.64, 0.8.18 之前，默认的 SSL 密码是 `ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP`。