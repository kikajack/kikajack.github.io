[原文地址](https://docs.docker.com/config/thirdparty/prometheus/)

[Prometheus](https://prometheus.io/) 是一个开源系统监控和告警工具包。可以将 Docker 配置为 Prometheus 的目标。本主题向你展示如何配置 Docker，设置 Prometheus 作为 Docker 容器运行，并使用 Prometheus 监控 Docker 实例。

>警告：可用指标和这些指标的名称正在积极开发中，并可能随时更改。

目前，只能监控 Docker 本身，无法监控应用程序。
#1.配置 Docker
要将 Docker 守护进程配置为 Prometheus 的目标，需要制定指标地址。最好的方式是通过默认在以下位置的 `daemon.json` 文件来指定。如果文件不存在，创建它。

- Linux：`/etc/docker/daemon.json`
- Windows Server：`C:\ProgramData\docker\config\daemon.json`
- Docker for Mac / Docker for Windows：单击工具栏中的 Docker 图标，选择 Preferences，然后选择 Daemon。点击高级。

如果文件是空的，把下面内容放入即可：
```
{
  "metrics-addr" : "127.0.0.1:9323",
  "experimental" : true
}
```
如果文件非空，添加这两个字段，并确保文件仍然是有效的 JSON 格式。注意除了最后一行，每一行都需要用逗号结尾。

保存文件或配置，重启 Docker。

现在，Docker 在 9323 端口暴露出兼容 Prometheus 的指标。
#2. 配置并运行 Prometheus
这个例子中，Prometheus 作为 Docker swarm 上的一个 Docker 服务运行。

>先决条件
>
1. 在 Docker swarm 中加入了一个或多个 Docker engine，在 manager 上执行 `docker swarm init` ，在其他 manager 和 worker 节点上执行 `docker swarm join`。
>
2. 需要互联网连接来获取普罗米修斯镜像。

复制下面的配置文件并保存到 `/tmp/prometheus.yml`（Linux 或 Mac）或 `C:\tmp\prometheus.yml`（Windows）。这是一个普通的 Prometheus 配置文件，除了在文件底部添加 Docker 作业定义之外。Docker for Mac 和 Docker for Windows 需要稍微修改配置。

- Docker for Linux（Mac 和 Windows 版本请参考原文）
```
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'codelab-monitor'

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first.rules"
  # - "second.rules"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'docker'
         # metrics_path defaults to '/metrics'
         # scheme defaults to 'http'.

    static_configs:
      - targets: ['localhost:9323']
```
然后，使用这个配置开启一个单副本的 Prometheus 服务。

- Docker for Linux（Mac 和 Windows 版本请参考原文）
```
$ docker service create --replicas 1 --name my-prometheus \
    --mount type=bind,source=/tmp/prometheus.yml,destination=/etc/prometheus/prometheus.yml \
    --publish published=9090,target=9090,protocol=tcp \
    prom/prometheus
```
验证 Docker target 在 `http://localhost:9090/targets/` 的列表中。

![prometheus-targets](http://img.blog.csdn.net/20180311200738495?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
如果你用的是 Docker for Mac 或 Docker for Windows 则无法直接访问终端的 URL。
#3. 使用 Prometheus
创建一个图。点击 Prometheus UI 中的 **Graphs** 链接。从 **Execute** 按钮右侧的组合框中选择一个指标，然后单击 **Execute** 按钮。以下屏幕截图显示了 `engine_daemon_network_actions_seconds_count` 的图形。

![prometheus-graph_idle](http://img.blog.csdn.net/20180311200812267?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

上图显示了一个闲置的 Docker 实例。如果你正在运行活动工作负载，则你的图形可能看起来不同。

为了使图表更有趣，可以通过启动一项包括 10 个不停地 ping 到 Docker 的任务的服务来创建一些网络操作，可以将 ping 目标更改为任何你喜欢的任务：
```
$ docker service create \
  --replicas 10 \
  --name ping_service \
  alpine ping docker.com
```
等待几分钟（默认采集间隔为15秒）并重新加载图形。

![prometheus-graph_load](http://img.blog.csdn.net/20180311200842869?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

准备就绪后，停止并删除 ping_service 服务，以免无缘无故地 ping 主机。
```
$ docker service remove ping_service
```
等待几分钟，你会发现图表又回落到闲置状态。
#4. Next steps
- Read the [Prometheus documentation](https://prometheus.io/docs/introduction/overview/)
- Set up some [alerts](https://prometheus.io/docs/alerting/overview/)