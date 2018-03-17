[原文地址](https://docs.docker.com/engine/security/certificates/)

在 [通过 HTTPS 运行 Docker](http://blog.csdn.net/kikajack/article/details/79590728)  中可以知道，默认情况下 Docker 通过非联网的 Unix 套接字运行，为了使 Docker 客户端和守护进程可以安全的通过 HTTPS 通信，必须开启 TLS。TLS 可以认证 registry 端点，并将进出 registry 的流量加密。

本文演示如何确保 Docker registry 服务器与 Docker 守护进程之间的流量经过加密，并使用基于证书的客户端 - 服务器身份验证进行正确验证。

我们将展示如何为 registry 安装证书颁发机构（CA）根证书以及如何设置客户端 TLS 证书以进行验证。
# 1. 理解配置
自定义证书通过在 `/etc/docker/certs.d` 目录下创建与 registry 主机同名（如localhost）的目录来进行配置。所有 `*.crt` 文件都作为 CA 根添加到此目录。

>注意：从 Docker 1.13 开始，在 Linux 上，任何根证书颁发机构都会与系统默认值合并，包括主机的根 CA 集。在以前版本的 Docker 和 Docker Enterprise Edition for Windows Server 上，仅在未配置自定义根证书时才使用系统默认证书。

存在一个或多个 `<filename>.key/cert` 对可以向 Docker 指出访问所需仓库所需自定义证书。

>注意：如果存在多个证书，则按照字母表顺序逐个尝试。如果发生 4xx 或 5xx 级别的认证错误，Docker 会继续尝试下一个认证。

具有自定义证书的配置：
```
    /etc/docker/certs.d/        <-- Certificate directory
    └── localhost:5000          <-- Hostname:port
       ├── client.cert          <-- Client certificate
       ├── client.key           <-- Client key
       └── ca.crt               <-- Certificate authority that signed
                                    the registry certificate
```
前面的示例是特定于操作系统的，仅用于说明目的。你应该查阅你的操作系统文档以创建一个操作系统提供的捆绑证书链。
# 2. 创建客户端证书
使用 OpenSSL 的 `genrsa` 和 `req` 命令首先生成 RSA 密钥，然后使用密钥创建证书。
```
$ openssl genrsa -out client.key 4096
$ openssl req -new -x509 -text -key client.key -out client.cert
```
>注意：这些 TLS 命令仅在 Linux 上生成一组工作证书。macOS 中的 OpenSSL 版本与 Docker 所需的证书类型不兼容。
# 3. 疑难解答提示
Docker 守护进程将 `.crt` 文件解释为 CA 证书，将 `.cert` 文件作为客户端证书。如果 CA 证书意外地被扩展名为 `.cert` 而不是正确的 `.crt` 扩展名，则 Docker 守护进程会记录以下错误消息：
```
Missing key KEY_NAME for client certificate CERT_NAME. CA certificates should use the extension .crt.
```
如果访问 Docker registry 时不需要指定端口号，就不要把端口号添加到目录名中。下面显示的 registry 配置使用默认的 443 端口，并且通过 `docker login my-https.registry.example.com` 访问：
```
    /etc/docker/certs.d/
    └── my-https.registry.example.com          <-- Hostname without port
       ├── client.cert
       ├── client.key
       └── ca.crt
```
