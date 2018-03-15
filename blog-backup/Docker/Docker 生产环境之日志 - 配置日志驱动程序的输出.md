[原文地址](https://docs.docker.com/config/containers/logging/log_tags/)

`tag` 日志选项指定如何格式化标识容器日志消息的标记。默认情况下，系统使用容器 ID 的前 12 个字符。要覆盖此行为，请指定一个标记选项：
```
$ docker run --log-driver=fluentd --log-opt fluentd-address=myhost.local:24224 --log-opt tag="mailer"
```
在指定 tag 值时，Docker 支持一些特殊的模板标记：

Markup|	描述
-|-
`{{.ID}}`|	容器 ID 的前 12 个字符
`{{.FullID}}`|	完整的容器 ID
`{{.Name}}`|	容器名
`{{.ImageID}}`|	容器镜像 ID 的前 12 个字符
`{{.ImageFullID}}`|容器的完整镜像 ID
`{{.ImageName}}`|	容器的镜像名
`{{.DaemonName}}`|	docker 的程序名

例如，指定 `--log-opt tag="{{.ImageName}}/{{.Name}}/{{.ID}}"` 值对应的 `syslog` 日志行如下：
```
Aug  7 18:33:19 HOSTNAME hello-world/foobar/5790672ab6a0[9103]: Hello from Docker.
```
在启动时，系统在标签中设置 `container_nam`字段和 `{{.Name}}`。如果使用 `docker rename` 重命名容器，则新名称不会反映在日志消息中。相反，这些消息继续使用原始容器名称。
