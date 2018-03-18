[原文地址](https://docs.docker.com/engine/security/trust/trust_sandbox/)

本页面介绍了如何设置和使用沙盒（sandbox）进行信任实验。沙盒允许你在本地配置和尝试信任操作，而不会影响生产镜像。

在开始这个沙盒实验之前，应该仔细阅读 [信任概述](http://blog.csdn.net/kikajack/article/details/79600066)。
# 1. 先决条件
这些说明假定你正在 Linux 或 macOS 中运行。可以在本地机器或虚拟机上运行此沙箱。需要拥有在本地机器或虚拟机上运行 docker 命令的权限。

此沙箱需要安装两个 Docker 工具：Docker Engine >= 1.10.0 和 Docker Compose >= 1.6.0。要安装 Docker 引擎，请从 [支持的平台列表](https://docs.docker.com/engine/installation/) 中进行选择。要安装 Docker Compose，请参阅 [此处](https://docs.docker.com/compose/install/)。
# 2. 沙盒 sandbox 是什么
如果你只是开箱即用，则只需要 Docker Engine 客户端并访问 Docker Hub。沙盒模拟生产中的信任环境，并设置这些附加组件。

Container	|Description
-|-
trustsandbox|	A container with the latest version of Docker Engine and with some preconfigured certificates. This is your sandbox where you can use the docker client to test trust operations.
Registry server	|A local registry service.
Notary server	|The service that does all the heavy-lifting of managing trust

这意味着你运行你自己的内容信任（Notary）服务器和 registry。如果你只使用 Docker Hub 工作，则不需要这些组件。它们为你构建在 Docker Hub 中。但是，对于沙箱，可以构建自己的整个模拟生产环境。

在 `trustsandbox` 容器中，可以与本地 registry 而不是 Docker Hub 进行交互。这意味着你的日常镜像仓库不被使用。他们在你练习的过程中受到保护。

在沙盒中玩时，还可以创建 root 和 repository 密钥。沙盒被配置为存储 `trustsandbox` 容器内的所有密钥和文件。由于你在沙盒中创建的密钥仅用于练习，因此销毁容器也会破坏它们。

通过在 `trustsandbox` 容器中使用 docker-in-docker 镜像，不会污染真正的 Docker 守护进程缓存，并且不会影响你推送和获取的任何镜像。这些镜像存储在与此容器相连的匿名卷中，并且可以在销毁容器后销毁。
# 3. 构建沙盒
在本节中，将使用 Docker Compose 来指定如何设置 `trustsandbox` 容器，Notary 服务器和 Registry 服务器并将其链接在一起。
#### 1. 创建并进入新的 `trustsandbox` 目录
```
 $ mkdir trustsandbox
 $ cd trustsandbox
```
#### 2. 创建名为 `docker-compose.yml` 的文件
```
 $ touch docker-compose.yml
 $ vim docker-compose.yml
```
#### 3. 把下面内容复制到刚创建的文件
```
 version: "2"
 services:
   notaryserver:
     image: dockersecurity/notary_autobuilds:server-v0.4.2
     volumes:
       - notarycerts:/go/src/github.com/docker/notary/fixtures
     networks:
       - sandbox
     environment:
       - NOTARY_SERVER_STORAGE_TYPE=memory
       - NOTARY_SERVER_TRUST_SERVICE_TYPE=local
   sandboxregistry:
     image: registry:2.4.1
     networks:
       - sandbox
     container_name: sandboxregistry
   trustsandbox:
     image: docker:dind
     networks:
       - sandbox
     volumes:
       - notarycerts:/notarycerts
     privileged: true
     container_name: trustsandbox
     entrypoint: ""
     command: |-
         sh -c '
             cp /notarycerts/root-ca.crt /usr/local/share/ca-certificates/root-ca.crt &&
             update-ca-certificates &&
             dockerd-entrypoint.sh --insecure-registry sandboxregistry:5000'
 volumes:
   notarycerts:
     external: false
 networks:
   sandbox:
     external: false
```
#### 4. 保存并关闭文件
#### 5. 本地运行容器
```
 $ docker-compose up -d
```
第一次运行这个时，docker-in-docker，Notary 服务器和 registry 镜像会从 Docker Hub 下载。
# 4. 运行沙盒
现在所有的东西都已经设置好了，你可以进入你的 `trustsandbox` 容器并开始测试 Docker 内容信任。在你的主机上，在 `trustsandbox` 容器中获取一个 shell。
```
$ docker container exec -it trustsandbox sh
/ #
```
## 4.1 测试 trust 操作
现在，从 `trustsandbox` 容器中获取镜像。

#### 1. 下载要测试的 Docker 镜像
```
 / # docker pull docker/trusttest
 docker pull docker/trusttest
 Using default tag: latest
 latest: Pulling from docker/trusttest

 b3dbab3810fc: Pull complete
 a9539b34a6ab: Pull complete
 Digest: sha256:d149ab53f8718e987c3a3024bb8aa0e2caadf6c0328f1d9d850b2a2a67f2819a
 Status: Downloaded newer image for docker/trusttest:latest
```
#### 2. 将其标记为推送到我们的沙箱注册表中：
```
 / # docker tag docker/trusttest sandboxregistry:5000/test/trusttest:latest
```
#### 3. 开启内容信任
```
 / # export DOCKER_CONTENT_TRUST=1
```
#### 4. 识别信任服务器
```
 / # export DOCKER_CONTENT_TRUST_SERVER=https://notaryserver:4443
```
这一步是必要的，因为沙盒正在使用自己的服务器。通常，如果使用的是 Docker Public Hub，则此步骤不是必需的。
#### 5. 获取测试镜像
```
 / # docker pull sandboxregistry:5000/test/trusttest
 Using default tag: latest
 Error: remote trust data does not exist for sandboxregistry:5000/test/trusttest: notaryserver:4443 does not have trust data for sandboxregistry:5000/test/trusttest
```
你看到一个错误，因为这个内容在 `notaryserver` 上还不存在。
#### 6. Push 并且 sign trusted 镜像
```
 / # docker push sandboxregistry:5000/test/trusttest:latest
 The push refers to a repository [sandboxregistry:5000/test/trusttest]
 5f70bf18a086: Pushed
 c22f7bc058a9: Pushed
 latest: digest: sha256:ebf59c538accdf160ef435f1a19938ab8c0d6bd96aef8d4ddd1b379edf15a926 size: 734
 Signing and pushing trust metadata
 You are about to create a new root signing key passphrase. This passphrase
 will be used to protect the most sensitive key in your signing system. Please
 choose a long, complex passphrase and be careful to keep the password and the
 key file itself secure and backed up. It is highly recommended that you use a
 password manager to generate the passphrase and keep it safe. There will be no
 way to recover this key. You can find the key in your config directory.
 Enter passphrase for new root key with ID 27ec255:
 Repeat passphrase for new root key with ID 27ec255:
 Enter passphrase for new repository key with ID 58233f9 (sandboxregistry:5000/test/trusttest):
 Repeat passphrase for new repository key with ID 58233f9 (sandboxregistry:5000/test/trusttest):
 Finished initializing "sandboxregistry:5000/test/trusttest"
 Successfully signed "sandboxregistry:5000/test/trusttest":latest
```
由于你第一次推送此存储库，因此 Docker 会创建新的 root 和 repository 密钥，并要求你提供用于加密它们的密码短语。如果在此之后再次推送，它只会要求输入存储库密码，以便它可以解密密钥并再次签名。
#### 7. 尝试获取刚上传的镜像
```
 / # docker pull sandboxregistry:5000/test/trusttest
 Using default tag: latest
 Pull (1 of 1): sandboxregistry:5000/test/trusttest:latest@sha256:ebf59c538accdf160ef435f1a19938ab8c0d6bd96aef8d4ddd1b379edf15a926
 sha256:ebf59c538accdf160ef435f1a19938ab8c0d6bd96aef8d4ddd1b379edf15a926: Pulling from test/trusttest
 Digest: sha256:ebf59c538accdf160ef435f1a19938ab8c0d6bd96aef8d4ddd1b379edf15a926
 Status: Downloaded newer image for sandboxregistry:5000/test/trusttest@sha256:ebf59c538accdf160ef435f1a19938ab8c0d6bd96aef8d4ddd1b379edf15a926
 Tagging sandboxregistry:5000/test/trusttest@sha256:ebf59c538accdf160ef435f1a19938ab8c0d6bd96aef8d4ddd1b379edf15a926 as sandboxregistry:5000/test/trusttest:latest
```
## 4.2 测试篡改镜像
在启用信任并尝试获取镜像时，如果数据损坏会发生什么？ 在本节中，将进入 `sandboxregistry` 并篡改一些数据。然后，尝试 pull 它。
#### 1. 保持 `trustsandbox` shell 和容器的运行
#### 2. 从主机打开一个新的交互式终端，并在 `sandboxregistry` 容器中获取一个 shell
```
$ docker container exec -it sandboxregistry bash
root@65084fc6f047:/#
```
#### 3. 列出你推送的 `test/trusttest` 镜像的层：
```
root@65084fc6f047:/# ls -l /var/lib/registry/docker/registry/v2/repositories/test/trusttest/_layers/sha256
total 12
drwxr-xr-x 2 root root 4096 Jun 10 17:26 a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4
drwxr-xr-x 2 root root 4096 Jun 10 17:26 aac0c133338db2b18ff054943cee3267fe50c75cdee969aed88b1992539ed042
drwxr-xr-x 2 root root 4096 Jun 10 17:26 cc7629d1331a7362b5e5126beb5bf15ca0bf67eb41eab994c719a45de53255cd
```
#### 4. 切换目录到其中一个层的 registry 存储位置（位于不同的目录中）：
```
root@65084fc6f047:/# cd /var/lib/registry/docker/registry/v2/blobs/sha256/aa/aac0c133338db2b18ff054943cee3267fe50c75cdee969aed88b1992539ed042
```
#### 5. 将恶意数据添加到其中一个 `trusttest` 层：
```
root@65084fc6f047:/# echo "Malicious data" > data
```
#### 6. 回到 `trustsandbox` 终端
#### 7. 列出 `trusttest` 镜像
```
/ # docker image ls | grep trusttest
REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
docker/trusttest                      latest              cc7629d1331a        11 months ago       5.025 MB
sandboxregistry:5000/test/trusttest   latest              cc7629d1331a        11 months ago       5.025 MB
sandboxregistry:5000/test/trusttest   <none>              cc7629d1331a        11 months ago       5.025 MB
```
#### 8. 从本地缓存中删除 `trusttest:latest` 镜像
```
/ # docker image rm -f cc7629d1331a
Untagged: docker/trusttest:latest
Untagged: sandboxregistry:5000/test/trusttest:latest
Untagged: sandboxregistry:5000/test/trusttest@sha256:ebf59c538accdf160ef435f1a19938ab8c0d6bd96aef8d4ddd1b379edf15a926
Deleted: sha256:cc7629d1331a7362b5e5126beb5bf15ca0bf67eb41eab994c719a45de53255cd
Deleted: sha256:2a1f6535dc6816ffadcdbe20590045e6cbf048d63fd4cc753a684c9bc01abeea
Deleted: sha256:c22f7bc058a9a8ffeb32989b5d3338787e73855bf224af7aa162823da015d44c
```
Docker does not re-download images that it already has cached, but we want Docker to attempt to download the tampered image from the registry and reject it because it is invalid.

#### 9. 再次获取镜像。这会从 registry 中下载。
```
/ # docker pull sandboxregistry:5000/test/trusttest
Using default tag: latest
Pull (1 of 1): sandboxregistry:5000/test/trusttest:latest@sha256:35d5bc26fd358da8320c137784fe590d8fcf9417263ef261653e8e1c7f15672e
sha256:35d5bc26fd358da8320c137784fe590d8fcf9417263ef261653e8e1c7f15672e: Pulling from test/trusttest

aac0c133338d: Retrying in 5 seconds
a3ed95caeb02: Download complete
error pulling image configuration: unexpected EOF
```
pull 无法完成，因为信任系统无法验证镜像。
# 5. 在沙盒中的更多练习
现在，你的本地系统上有一个完整的 Docker 内容信任沙箱，可以随时使用它并查看它的行为。如果发现 Docker 存在任何安全问题，请随时通过security@docker.com向我们发送电子邮件。
# 6. 清理沙盒
完成后，要清理所有已启动的服务和已创建的所有匿名卷，只需在创建 Docker Compose 文件的目录中运行以下命令：
```
    $ docker-compose down -v
```
