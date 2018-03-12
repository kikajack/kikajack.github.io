[Nginx 官方教程](https://www.nginx.com/blog/using-free-ssltls-certificates-from-lets-encrypt-with-nginx/)
[Certbot （证书安装工具）的官网](https://certbot.eff.org/)
[Certbot  工具的文档](https://certbot.eff.org/docs)
[参考资料](http://frontenddev.org/article/using-certbot-deployment-let-s-encrypt-free-ssl-certificate-implementation-https.html)
[lets encrypt 的文档](http://letsencrypt.readthedocs.io/en/latest/intro.html)

SSL / TLS加密会为您的用户带来更高的搜索排名和更好的安全性。
Let's Encrypt 是一个认证机构（CA）。它可以提供免费证书，并且已经被大多数浏览器所信任。另外，通过工具 Certbot 可以让我们完全自动化证书的安装和更新。
安装证书的前提条件：

- 安装服务器（这里用 NGINX）。
- 注册域名。
- 创建一个DNS记录，将域名和服务器的 IP 地址相关联。

记得安装完成后，防火墙需要打开 443 端口，否则无法访问！！！
#1. 安装 Let's Encrypt 客户端
所有的证书相关的操作，都可以通过 Certbot 软件实现。
***注意：HTTPS 作为重要的基础服务，一旦出问题就需要立刻把软件更新到官网提供的最新版本，所以不推荐用各个 Linux 发行版的默认仓库进行安装，而是通过官方工具 certbot-auto 安装。***

```
wget https://dl.eff.org/certbot-auto # 下载到本地
chmod a+x ./certbot-auto # 添加可执行权限
./certbot-auto --help all # 查看帮助
```
#2. 验证域名所有权
配置 Nginx，使要获取证书的域名对应的 80 端口可以正常访问。在 `/etc/nginx/conf.d/` 目录下为域名创建新文件 `www.example.com.conf`，并添加相关配置信息：
```
server {
    root /home/kikakika/trunk;
    server_name kikakika.com;
    index  index.html index.htm index.php;

    location ~ \.php(.*)$  {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        fastcgi_param  PATH_INFO  $fastcgi_path_info;
        fastcgi_param  PATH_TRANSLATED  $document_root$fastcgi_path_info;
        include        fastcgi_params;
    }
}
```
重启 Nginx
```
systemctl restart nginx
```
#3. 生成证书
```
certbot-auto --nginx -d kikakika.com -d www.kikakika.com
```
正常情况下，进入交互式界面，提示你输入邮箱（在证书失效前收到通知邮件），并同意官方协议。

```
[root@VM_120_242_centos tmp]# ./certbot-auto --nginx -d scall2.szhuizhong.cn
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for scall2.szhuizhong.cn
Waiting for verification...
Cleaning up challenges
Deployed Certificate to VirtualHost /etc/nginx/nginx.conf for scall2.szhuizhong.cn

Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
```
证书生成成功后，会让你选择**是否将所有的 HTTP 请求重定向到 HTTPS**（输入 1 或者 2）。如果选 1，则通过 HTTP 和 HTTPS 都可以访问。如果选 2，则所有通过 HTTP 来的请求，都会被 301 重定向到 HTTPS。参考下面的 `5. 配置 Nginx`。
输完 1 或者 2 回车后，会有成功提示，并说明证书放在 `/etc/letsencrypt/live/证书的域名` 这个位置：
```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/kikakika.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/kikakika.com/privkey.pem
   Your cert will expire on 2018-04-21. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```
#4. 开启 443 端口
开启443端口：`firewall-cmd --permanent --add-port=443/tcp`
查看是否成功：`firewall-cmd --permanent --query-port=443/tcp`
端口添加成功后，需要reload重新载入：`firewall-cmd --reload`
#5. 配置 Nginx
certbot-auto 会自动调用 Nginx 的插件，改写配置文件。可以通过参数 `certonly ` 关闭自动改写配置文件的功能，只获取证书。
`./certbot-auto certonly -d  kikakika.com`

下面是 certbot-auto 自动改写过的配置文件，所有改写过的行都在后面加了注释：`# managed by Certbot`。
```
# 生成证书时选 1: No redirect，不把 HTTP 请求重定向到 HTTPS，一个 server 同时监听 80 和 443 端口
server {
    listen    80;
    server_name scall2.szhuizhong.cn;
    root    /home/szhuizhong/trunk;

    location / {
    if (-f $request_filename) {
            expires max;
            break;
        }
        if (!-e $request_filename) {
            rewrite ^/(.*)$ /index.php/$1 last;
        }
        index  index.html index.htm index.php l.php;
        autoindex  off;
    }

    error_page 404 /404.html;
        location = /40x.html {
    }
    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
  location ~ \.php(.*)$  {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        fastcgi_param  PATH_INFO  $fastcgi_path_info;
        fastcgi_param  PATH_TRANSLATED  $document_root$fastcgi_path_info;
        fastcgi_param  CI_ENV  'testing';
        include        fastcgi_params;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/scall2.szhuizhong.cn/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/scall2.szhuizhong.cn/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
```
```
# 生成证书时选 2: redirect，HTTP 请求重定向到 HTTPS，两个 server 分别监听 80 和 443 端口
server {
    root /home/kikakika/trunk;
    server_name kikakika.com;
    index  index.html index.htm index.php;

        location ~ \.php(.*)$  {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            fastcgi_param  PATH_INFO  $fastcgi_path_info;
            fastcgi_param  PATH_TRANSLATED  $document_root$fastcgi_path_info;
            include        fastcgi_params;
        }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/kikakika.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/kikakika.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
server {
    server_name kikakika.com www.kikakika.com;
    listen 80;
    return 301 https://$host$request_uri; # managed by Certbot
}
```
#6. 设置定时任务
出于安全策略， Let's Encrypt 签发的证书有效期只有 90 天。通过 `./certbot-auto renew ` 命令可以续签。
- 编辑 crontab 文件：
```
$ crontab -e
```
- 添加 certbot 命令：
在每天凌晨3点运行。该命令将检查服务器上的证书是否将在未来30天内过期，如果是，则进行更新。`--quiet` 指令告诉 certbot 不要生成输出。
```
0 3 * * * /root/letsencrypt/certbot-auto renew --quiet
```
保存并关闭文件。所有安装的证书将自动更新并重新加载。
#7. 知识点
##7.1 certbot-auto 和 certbot
certbot-auto 和 certbot 的不同之处在于运行 certbot-auto 会自动安装它自己所需要的一些依赖，并且**自动更新**，推荐使用（因为各个 Linux 发行版的仓库，更新不是很及时）。

##7.2 certbot 的工作方式
- standalone： certbot 会自己运行一个 web server 来进行验证。如果我们自己的服务器上已经有 web server 正在运行 （比如 Nginx 或 Apache ），用 standalone 方式的话需要先关掉它，以免冲突。
- webroot： certbot 会利用既有的 web server，在其 web root目录下创建隐藏文件， Let's Encrypt 服务端会通过域名来访问这些隐藏文件，以确认你的确拥有对应域名的控制权。
#8. certbot 语法、命令和选项
##8.1 语法
 `certbot-auto [SUBCOMMAND] [options] [-d DOMAIN] [-d DOMAIN] ...`
通过 `./certbot-auto --help all` 可以查看所有支持的子命令和参数。
通过 `./certbot-auto --help 命令` 可以查看具体命令的用法。比如 `./certbot-auto --help delete` 。
可以一次为多个域名申请证书。
##8.2 命令
命令分三大类：获取、安装、更新证书，管理证书，管理 Let's Encrypt 账户。
###8.2.1 获取、安装、更新证书
不写任何子命令时，默认使用 run 命令。`certbot-auto --nginx -d xx.com` 等价于 `certbot-auto run --nginx -d xx.com`。
```
    (default) run   为当前服务器获取并安装一个证书
    certonly        只获取或更新证书，不安装
    renew           更新所有快要过期的证书
   -d DOMAINS       指定要获取证书的域名，可以同时指定多个

  --apache          使用 Apache 服务器插件来认证和安装证书
  --standalone      Run a standalone webserver for authentication
  --nginx           使用 Nginx 服务器插件来认证和安装证书，可以读取 Nginx 配置文件并自动更改。
  --webroot         Place files in a server's webroot folder for authentication
  --manual          交互式获取证书，或者使用钩子脚本

   -n               非交互式运行
  --test-cert       从 staging server 获取一个测试证书
  --dry-run         测试 "renew" 和 "certonly" 命令，不保存证书到磁盘上
```
####1. run 安装证书
完整命令：`certbot-auto [SUBCOMMAND] [options] [-d DOMAIN] [-d DOMAIN] ...`
可选参数：

- `-c CONFIG_FILE, --config CONFIG_FILE`：指定配置文件（默认是: /etc/letsencrypt/cli.ini 和 ~/.config/letsencrypt/cli.ini）。
- `-n, --non-interactive, --noninteractive`：非交互式运行。需要额外的命令行参数。
- `--force-interactive`：强制交互式运行，即使 Certbot 不是在终端运行。
- `-d DOMAIN, --domains DOMAIN, --domain DOMAIN`：要认证的域名。可以用多个 `-d` 参数同时认证多个域名。提供的第一个域名将成为证书的主体 CN，所有域将成为证书上的主体备选名称。除非另有说明，否则第一个域也将用于某些软件用户界面以及证书和相关材料的文件路径，或者您已经拥有同名的证书。 在名称冲突的情况下，它会在文件路径名称后附加一个数字，如0001。（The first domain provided will be the subject CN of the certificate, and all domains will be Subject Alternative Names on the certificate.The first domain will also be used in some software user interfaces and as the file paths for the certificate and related material unless otherwise specified or you already have a certificate with the same name. In the case of a name collision it  will append a number like 0001 to the file path name.）
- `--nginx`：通过 Nginx 获取并安装证书。
- `--apache`：通过 Apache 获取并安装证书。
- `--cert-name CERTNAME`：指定证书名，默认使用第一个域名或相同域名已有的证书名。Certbot 用这个名称来管理证书文件。名称不影响证书内容。可以用 `certbot certificates` 命令查看证书名。
- `--test-cert, --staging`：用 staging server 来获取或撤销测试证书。相当于 `--server https://acme-staging.api.letsencrypt.org/directory`。
- `--keep-until-expiring, --keep, --reinstall`：如果请求的证书与现有的证书相匹配，则始终保留现有的证书，直到到期才重新安装现有的证书。
- `-q, --quiet`：静默安装。用于 cron 定时任务。
####2. renew 更新证书
renew 子命令会尝试更新所有之前获取的并快要过期的证书。命令执行的概要结果会显示出来。默认情况下，renew 会使用获取证书时所使用的参数，或最后一次成功 renew 每个证书时所用的参数。在执行 renew 命令之前或之后，可以调用钩子函数，[详情在这里](https://certbot.eff.org/docs/using.html#renewal)。

完整命令：`certbot-auto renew [--cert-name CERTNAME] [options]`
可选参数：

- `--cert-name CERTNAME`：指定要更新的证书。Certbot 用这个名称来管理证书文件。名称不影响证书内容。可以用 `certbot certificates` 命令查看证书名。
- `--dry-run`：用于测试，只获取测试证书，不保存至磁盘。只有 certonly 和 renew 两个子命令可以用这个参数。仍然会会改写 Apache 或 Nginx 服务器的配置文件并重启服务器。仍然会调用 `--pre-hook` 和 `--post-hook` 命令（只要定义过）。只是不再调用 `--deploy-hook` 命令。
- `--force-renewal, --renew-by-default`：强制更新域名的证书，即使离过期时间还远得很。
- `--allow-subset-of-names`：在域名所有权认证时，即使认证失败，也产生证书。在更新多个域名时有效，因为有可能部分域名不再指向当前主机。注意：不能和参数 `--csr` 同时使用。
- `-q, --quiet`：静默执行。
- `--debug-challenges`：调试模式，提交至 CA 前需要用户确认。
- `--preferred-challenges`：验证域名所有权的方式，"dns" 或 "tls-sni-01,http,dns" 等。每个服务器插件支持有限种类的方式。[参考这里](https://certbot.eff.org/docs/using.html#plugins)
- `--pre-hook PRE_HOOK`：在获取证书前要执行的 shell 命令。比如暂时关闭服务器软件以防止可能的冲突。只有在自动获取/更新证书时才会执行。如果更新多个证书时，只执行第一个命令。
- `--post-hook POST_HOOK`：在获取证书后要执行的 shell 命令。比如**部署新证书，或重启服务器软件。**如果更新多个证书时，只执行第一个命令。
- `--deploy-hook DEPLOY_HOOK`：每个有效的认证都会触发一次的 shell 命令。对这个命令，shell 变量 `$RENEWED_LINEAGE` 表示包含域名证书和私钥的配置目录，比如 `/etc/letsencrypt/live/example.com`。`$RENEWED_DOMAINS` 表示空格分隔的刚更新的域名列表，比如 `example.com www.example.com`。
- `--disable-hook-validation`：验证 `--pre-hook/--post-hook/--deploy-hook` 的 shell 命令是否有效，提前发现错误。
- `--no-directory-hooks`：更新证书时不执行 hook 目录中的命令。
###8.2.2 管理证书
```
    certificates    显示从 Certbot 获取的证书的信息
    revoke          撤销证书（提供证书路径 --cert-path）
    delete          删除证书
```
####1. revoke 撤销证书
完整命令：`certbot-auto revoke --cert-path /path/to/fullchain.pem [options]`
可选参数：

- `--test-cert, --staging`：用 staging server 来获取或撤销测试（无效的）证书。等价于 `--server https://acme-staging.api.letsencrypt.org/directory`
- `--reason {unspecified,keycompromise,affiliationchanged,superseded,cessationofoperation}`：撤销证书的原因。
- `--delete-after-revoke`：撤销后同时删除证书。
- `--no-delete-after-revoke`：撤销后不删除证书。**小心使用，因为 renew 子命令或尝试更新所有的已撤销但未删除的证书。**
- `--cert-path CERT_PATH`：证书保存，安装或撤销时查询的目录位置。
- `--key-path KEY_PATH`：在证书保存，安装或撤销时查询私钥 private key 的目录位置。
####2. delete 删除证书
完整命令：`certbot-auto delete --cert-name CERTNAME`
可选参数：

- `-c CONFIG_FILE, --config CONFIG_FILE`：指定配置文件（默认是: /etc/letsencrypt/cli.ini 和 ~/.config/letsencrypt/cli.ini）。
- `--cert-name CERTNAME`：要删除的证书，可以用 `certbot certificates` 命令查看证书名。证书名默认就是对应的域名。
###8.2.3 管理 Let's Encrypt 账户
```
    register        创建一个 Let's Encrypt ACME 账户
  --agree-tos       统一 ACME 的服务订阅协议
   -m EMAIL         用于获取重要账户通知的邮箱
```


