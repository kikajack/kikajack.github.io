[原文地址](https://docs.docker.com/registry/)

>需要 Docker Trusted Registry 吗？
>Docker Trusted Registry（DTR）是一款商业产品，支持完整的镜像管理工作流程，包括 LDAP 集成，镜像签名，安全扫描以及与通用控制平面的集成。DTR 作为标准版或更高版本的 Docker Enterprise 订阅的附件提供。
# 1. 是什么
Registry 是一个无状态，高度可扩展的服务器端应用程序，用于存储和分发 Docker镜像。Registry 是开放源代码的，采用 [Apache 授权协议](http://en.wikipedia.org/wiki/Apache_License)。
# 2. 为什么使用
以下场景需要使用 Registry：

- 严格控制镜像存储位置
- 完全拥有镜像分配管道
- 将镜像存储和分发紧密集成到你的内部开发工作流程中
# 3. 替换方案
对于寻求零维护、随时可用的解决方案的用户，建议前往 [Docker Hub](https://hub.docker.com/)，这里提供免费使用的托管 registry，以及其他功能（组织帐户，自动构建等）。

寻求注册商业版支持的用户应该查看 Docker Trusted Registry。
# 4. 需求
Registry 与 Docker engine 1.6.0 或更高版本兼容。
# 5. 基本命令
启动你的 registry
```
docker run -d -p 5000:5000 --name registry registry:2
```
从 Hub Pull（或 build）镜像
```
docker pull ubuntu
```
为镜像添加标签使其指向你的 registry
```
docker image tag ubuntu localhost:5000/myfirstimage
```
Push 镜像
```
docker push localhost:5000/myfirstimage
```
Pull 镜像
```
docker pull localhost:5000/myfirstimage
```
停止 registry，删除所有数据
```
docker container stop registry && docker container rm -v registry
```
