[原文地址](https://docs.docker.com/engine/security/https/)

默认情况下，Docker 通过非联网的 Unix 套接字运行。它也可以选择使用 HTTP 套接字进行通信。

如果需要通过网络以安全方式访问 Docker，则可以通过指定 `tlsverify` 标志并将 Docker 的 `tlscacert` 标志指向受信任的 CA 证书来启用 TLS。

在守护进程模式下，它只允许客户端通过由该 CA 签名的证书进行身份验证。在客户端模式下，它仅连接到具有由该 CA 签名的证书的服务器。


>高级主题
>使用 TLS 和管理 CA 是高级主题。在生产环境中使用之前，请先了解 OpenSSL，x509 和 TLS。
# 1. 使用 OpenSSL 创建 CA，服务器和客户端密钥
>注意：将下面示例中所有 `$HOST` 替换为你的 Docker 守护进程所在主机的 DNS 名称。

首先，在 **Docker 守护进程所在主机上**，创建 CA 公私钥：
```
$ openssl genrsa -aes256 -out ca-key.pem 4096
Generating RSA private key, 4096 bit long modulus
............................................................................................................................................................................................++
........++
e is 65537 (0x10001)
Enter pass phrase for ca-key.pem:
Verifying - Enter pass phrase for ca-key.pem:

$ openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
Enter pass phrase for ca-key.pem:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:Queensland
Locality Name (eg, city) []:Brisbane
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Docker Inc
Organizational Unit Name (eg, section) []:Sales
Common Name (e.g. server FQDN or YOUR name) []:$HOST
Email Address []:Sven@home.org.au
```
现在你已经有 CA 了，可以创建一个服务器密钥和证书签名请求（certificate signing request，CSR）。确保“通用名称”（Common Name）与用于连接到 Docker 的主机名相匹配：
```
$ openssl genrsa -out server-key.pem 4096
Generating RSA private key, 4096 bit long modulus
.....................................................................++
.................................................................................................++
e is 65537 (0x10001)

$ openssl req -subj "/CN=$HOST" -sha256 -new -key server-key.pem -out server.csr
```
接下来，用我们的 CA 签署公钥：

由于 TLS 连接可以通过 IP 地址和 DNS 域名建立，因此创建证书时需要指定 IP 地址。例如，要允许使用 10.10.10.20 和 127.0.0.1 进行连接：
```
$ echo subjectAltName = DNS:$HOST,IP:10.10.10.20,IP:127.0.0.1 >> extfile.cnf
```
将 Docker 守护进程密钥的扩展使用属性设置为仅用于服务器身份验证：
```
$ echo extendedKeyUsage = serverAuth >> extfile.cnf
```
现在，生成签名证书：
```
$ openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem \
  -CAcreateserial -out server-cert.pem -extfile extfile.cnf
Signature ok
subject=/CN=your.host.com
Getting CA Private Key
Enter pass phrase for ca-key.pem:
```
[授权插件](https://docs.docker.com/engine/extend/plugins_authorization) 提供更细致的控制，以相互补充 TLS 的认证。除了上述文档中描述的其他信息之外，在 Docker 守护程序上运行的授权插件会接收用于连接 Docker 客户端的证书信息。

对于客户端身份验证，请创建客户端密钥和证书签名请求：

>注意：为了简化接下来的几个步骤，也可以在 Docker 守护进程的主机上执行此步骤。
```
$ openssl genrsa -out key.pem 4096
Generating RSA private key, 4096 bit long modulus
.........................................................++
................++
e is 65537 (0x10001)

$ openssl req -subj '/CN=client' -new -key key.pem -out client.csr
```
要使密钥适合客户端认证，请创建一个扩展配置文件：
```
$ echo extendedKeyUsage = clientAuth >> extfile.cnf
```
现在，生成签名证书：
```
$ openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem \
  -CAcreateserial -out cert.pem -extfile extfile.cnf
Signature ok
subject=/CN=client
Getting CA Private Key
Enter pass phrase for ca-key.pem:
```
在生成 `cert.pem` 和 `server-cert.pem` 后，可以安全的删除这两个证书签名请求：
```
$ rm -v client.csr server.csr
```
通过默认的 022 `umask`，你的密钥对你和你的组来说，是完全可写可读的。

为防止意外损坏你的密钥，请删除其写入权限。要让它们只能被你读取，请按如下方式更改文件模式：
```
$ chmod -v 0400 ca-key.pem key.pem server-key.pem
```
证书可以被所有人读，但是最好删除写权限以防止意外损坏：
```
$ chmod -v 0444 ca.pem server-cert.pem cert.pem
```
现在，可以使 Docker 守护进程只接受提供能够通过你的 CA 认证的证书的客户端的连接：
```
$ dockerd --tlsverify --tlscacert=ca.pem --tlscert=server-cert.pem --tlskey=server-key.pem \
  -H=0.0.0.0:2376
```
要连接到 Docker 并验证其证书，请提供你的客户端密钥，证书和可信 CA：

>在客户端机器上运行这个命令
>这一步应该运行在你的 Docker 客户端上。因此，需要将 CA 证书，服务器证书和客户端证书复制到该机器。
```
$ docker --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=key.pem \
  -H=$HOST:2376 version
```
注意：使用 TLS 的 Docker 应该运行在 TCP 的 2376 端口。

>警告：如上例所示，使用证书验证身份时，不需要使用 `sudo` 或 `docker` 组运行 `docker` 客户端。这意味着任何拥有密钥的人都可以给你的 Docker 守护进程提供任何指令，让他们可以访问托管守护进程的机器。保护这些密钥，就像你使用 root 密码一样！
# 2. 默认安全
如果想在默认情况下确保 Docker 客户端的连接安全，可以将文件移动到你的主目录下的 `.docker` 目录 - 并设置 `DOCKER_HOST` 和 `DOCKER_TLS_VERIFY` 变量以及（而不是在每次调用时传递  `-H=tcp://$HOST:2376` 和 `--tlsverify`）。
原文：
If you want to secure your Docker client connections by default, you can move the files to the .docker directory in your home directory -- and set the DOCKER_HOST and DOCKER_TLS_VERIFY variables as well (instead of passing -H=tcp://$HOST:2376 and --tlsverify on every call).
```
$ mkdir -pv ~/.docker
$ cp -v {ca,cert,key}.pem ~/.docker

$ export DOCKER_HOST=tcp://$HOST:2376 DOCKER_TLS_VERIFY=1
```
Docker 现在默认就是安全连接：
```
$ docker ps
```
# 3. 其他模式
如果你不需要完整的双向认证，可以通过组合使用各种标志来以各种其他模式运行Docker。
## 3.1 守护进程模式
- `tlsverify`、`tlscacert`、`tlscert`、`tlskey`：认证客户端。
- `tls`、`tlscert`、`tlskey`：不认证客户端。
## 3.2 客户端模式
- `tls`：基于公共或默认的 CA 池验证服务器
- `tlsverify`、`tlscacert`：基于给定的 CA 验证服务器
- `tls`、`tlscert`、`tlskey`：根据给定的 CA 验证客户端证书并验证服务器验证客户端证书，但不验证服务器
- `tlsverify`、`tlscacert`、`tlscert`、`tlskey`：根据给定的 CA 验证客户端证书并验证服务器

如果找到了，客户端会发送它的客户端证书，所以只需将你的密钥放入 `~/.docker/{ca,cert,key}.pem`。或者，如果要将密钥存储在其他位置，则可以使用环境变量 `DOCKER_CERT_PATH` 指定该位置。
```
$ export DOCKER_CERT_PATH=~/.docker/zone1/
$ docker --tlsverify ps
```
## 3.3 使用 curl 连接到安全的 Docker 端口
要使用 curl 来创建测试 API 请求，需要使用三个额外的命令行标志：
```
$ curl https://$HOST:2376/images/json \
  --cert ~/.docker/cert.pem \
  --key ~/.docker/key.pem \
  --cacert ~/.docker/ca.pem
```
