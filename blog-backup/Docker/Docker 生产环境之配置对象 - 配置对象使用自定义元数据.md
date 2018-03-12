[原文地址](https://docs.docker.com/config/labels-custom-metadata/)

标签（label）是一种将元数据应用于 Docker 对象的机制，包括：

- Images
- Containers
- Local daemons
- Volumes
- Networks
- Swarm nodes
- Swarm services

可以使用标签来组织镜像，记录许可信息，注释容器、卷和网络之间的关系，或以任何对你的业务或应用程序有意义的方式进行。
#1. 标签的键和值
标签是键值对，存储为字符串。可以为一个对象指定多个标签，但是同一个对象中的键值对必须独一无二。如果同一个键指定了多个值，后面的值会覆盖前面的。
##1.1 建议使用的键格式
标签的键是键值对中左边的元素。键是可以包含句点（`.`）和中横线（`-`）的字母数字字符串。大多数 Docker 用户使用由其他组织创建的镜像，并且以下指导原则有助于防止无意间的跨对象重复标签，特别是如果你打算将标签用作自动化机制。

- 第三方工具的作者应在每个标签的键前加上他们的域名的倒序 DNS 标记，如 `com.example.some-label`。
- 未经域名所有者许可，请勿在标签的键中使用这个域名。
- `com.docker.*`、`io.docker.*` 和 `org.dockerproject.*` 命名空间是 Docker 保留用于内部使用的。
- 标签键应以小写字母开头和结尾，并且只能包含小写字母数字字符，句点字符（`.`）和连字符（`-`）。不允许连续的句点或连字符。
- 句点字符（`.`）分隔命名空间“字段”。没有命名空间的标签的键被保留用于 CLI 使用，允许CLI的用户使用更短的键入友好字符串交互地标记Docker对象。

这些准则目前尚未实施，对于特定用例会需要更多的规则。
##1.2 值的规则
标签的值可以包含任意能表示为字符串的数据类型，包括但不限于 JSON、XML、CSV 或 YAML。唯一要求是必须使用特定于结构类型的机制将值序列化为字符串。例如，要将 JSON 序列化为字符串，可以使用JavaScript 中的 `JSON.stringify()` 方法。

由于 Docker 并未反序列化该值，因此在按标签值查询或过滤时，不能将 JSON 或 XML 文档视为嵌套结构，除非你将此功能构建到第三方工具中。
#2. 管理对象上的标签
支持标签的每种类型的对象都具有添加和管理标签的机制，并可以在该类型对象上使用它们。这些链接提供了一个开始学习如何在 Docker 部署中使用标签的好地方。

镜像、容器、本地守护进程、卷和网络上的标签在对象的生命周期内是静态的。必须要重新创建对象才能改变这些标签。swarm 节点和服务上的标签可以动态更新。

- 镜像和容器
 - [Adding labels to images](https://docs.docker.com/engine/reference/builder/#label)
 - [Overriding a container’s labels at runtime](https://docs.docker.com/engine/reference/commandline/run/#set-metadata-on-container--l---label---label-file)
 - [Inspecting labels on images or containers](https://docs.docker.com/engine/reference/commandline/inspect/)
 - [Filtering images by label](https://docs.docker.com/engine/reference/commandline/images/#filtering)
 - [Filtering containers by label](https://docs.docker.com/engine/reference/commandline/ps/#filtering)
- 本地 Docker 守护进程
 - [Adding labels to a Docker daemon at runtime](https://docs.docker.com/engine/reference/commandline/dockerd/)
 - [Inspecting a Docker daemon’s labels](https://docs.docker.com/engine/reference/commandline/info/)
- 卷
 - [Adding labels to volumes](https://docs.docker.com/engine/reference/commandline/volume_create/)
 - [Inspecting a volume’s labels](https://docs.docker.com/engine/reference/commandline/volume_inspect/)
 - [Filtering volumes by label](https://docs.docker.com/engine/reference/commandline/volume_ls/#filtering)
- 网络
 - [Adding labels to a network](https://docs.docker.com/engine/reference/commandline/network_create/)
 - [Inspecting a network’s labels](https://docs.docker.com/engine/reference/commandline/network_inspect/)
 - [Filtering networks by label](https://docs.docker.com/engine/reference/commandline/network_ls/#filtering)
- Swarm 节点
 - [Adding or updating a swarm node’s labels](https://docs.docker.com/engine/reference/commandline/node_update/#add-label-metadata-to-a-node)
 - [Inspecting a swarm node’s labels](https://docs.docker.com/engine/reference/commandline/node_inspect/)
 - [Filtering swarm nodes by label](https://docs.docker.com/engine/reference/commandline/node_ls/#filtering)
- Swarm 服务
 - [Adding labels when creating a swarm service](https://docs.docker.com/engine/reference/commandline/service_create/#set-metadata-on-a-service-l-label)
 - [Updating a swarm service’s labels](https://docs.docker.com/engine/reference/commandline/service_update/)
 - [Inspecting a swarm service’s labels](https://docs.docker.com/engine/reference/commandline/service_inspect/)
 - [Filtering swarm services by label](https://docs.docker.com/engine/reference/commandline/service_ls/#filtering)