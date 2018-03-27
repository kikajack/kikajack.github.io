[原文地址](https://docs.docker.com/registry/notifications/)

Registry 支持发送 webhook 通知以响应 registry 中发生的事件。通知是为了响应 push 和 pull 以及对 layer 的 push 这些事件而发送的。这些操作被串行化成事件。这些事件排队到 registry 内部的广播系统，该系统将事件排入队列并将事件分派到 [端点](https://docs.docker.com/registry/notifications/#endpoints)。

![notifications](https://img-blog.csdn.net/20180325232109859?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpa2FqYWNr/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
# 1. 端点 Endpoint
通知通过 HTTP 请求发送到端点。每个配置过的端点都有独立的队列，在 registry 的每个实例中重试配置和 http 目标（retry configuration and http targets within each instance of a registry）。当 registry 中发送某个动作时，会被转换成一个放入内存队列的事件。当事件到达队列尽头时，会向端点发出 http 请求，直到请求成功。事件串行发送到每个端点，但订单不能保证（The events are sent serially to each endpoint but order is not guaranteed）。
# 2. 配置
要设置 registry 实例以将通知发送到端点，必须将其添加到配置中。一个简单的例子如下：
```
  notifications:
    endpoints:
      - name: alistener
        url: https://mylistener.example.com/event
        headers:
          Authorization: [Bearer <your token, if needed>]
        timeout: 500ms
        threshold: 5
        backoff: 1s
```
以上内容将配置 registry 与端点，以便将事件发送到 `https://mylistener.example.com/event`，标题为“Authorization：Bearer <你的令牌，如果需要的话>”。请求将在 500 毫秒后超时。如果连续发生 5 次失败，则 registry 将在 1 秒钟后再次尝试。

有关这些字段的详细信息，请参阅 [配置文档](https://docs.docker.com/registry/configuration/#notifications)。

在 registry 启动时，配置正确的端点应该记录日志消息：
```
INFO[0000] configuring endpoint alistener (https://mylistener.example.com/event), timeout=500ms, headers=map[Authorization:[Bearer <your token if needed>]]  app.id=812bfeb2-62d6-43cf-b0c6-152f541618a3 environment=development service=registry
```
# 3. 事件
事件具有明确定义的 JSON 结构，并作为通知请求的请求体发送。一个或多个事件通过称为 `envelope` 的结构发送。如果需要，每个事件都可以有唯一的 ID 用于唯一标识传入的请求。除此之外，还为目标提供了一个操作，用于识别事件期间发生变化的对象。

下面是事件中可用的字段：
字段|	类型|	描述
-|-
id|	string|	事件的唯一标识符
timestamp|	Time	|事件发生时的时间戳
action|	string|	表示所提供的事件包含哪些操作
target	|distribution.Descriptor	|唯一地描述事件的目标 target
length	|int|	内容长度，单位字节。与 Descriptor 中的 Size 字段一样
repository	|string|	标识指定的仓库
fromRepository|	string|	如果合适的话，指定了一个用于挂载 BLOB 的仓库
url|	string	|URL 提供到内容的直接链接
tag|	string	|标识 tag 事件中的标签名称
request|	RequestRecord|	包含生成事件的请求
actor|	ActorRecord.|	指定发起事件的代理。大多数情况下，这可能来自请求的授权上下文（this could be from the authorization context of the request）
source	|SourceRecord	|标识生成该事件的 registry 节点。换句话说，当 actor “发起”事件时，source “产生”它（while the actor “initiates” the event, the source “generates” it）

下面是 JSON 事件的示例，在发起 pull 操作时触发：
```
{
   "events": [
      {
         "id": "320678d8-ca14-430f-8bb6-4ca139cd83f7",
         "timestamp": "2016-03-09T14:44:26.402973972-08:00",
         "action": "pull",
         "target": {
            "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
            "size": 708,
            "digest": "sha256:fea8895f450959fa676bcc1df0611ea93823a735a01205fd8622846041d0c7cf",
            "length": 708,
            "repository": "hello-world",
            "url": "http://192.168.100.227:5000/v2/hello-world/manifests/sha256:fea8895f450959fa676bcc1df0611ea93823a735a01205fd8622846041d0c7cf",
            "tag": "latest"
         },
         "request": {
            "id": "6df24a34-0959-4923-81ca-14f09767db19",
            "addr": "192.168.64.11:42961",
            "host": "192.168.100.227:5000",
            "method": "GET",
            "useragent": "curl/7.38.0"
         },
         "actor": {},
         "source": {
            "addr": "xtal.local:5000",
            "instanceID": "a53db899-3b4b-4a62-a067-8dd013beaca4"
         }
      }
   ]
}
```
events 中的 target 结构体会在删除清单和 Blob 时发送 Get 和 Put 事件中包含的数据的子集（仅发送摘要和存储库）。
```
"target": {
            "digest": "sha256:d89e1bee20d9cb344674e213b581f14fbd8e70274ecf9d10c514bab78a307845",
            "repository": "library/test"
},
```
>注意：从版本 2.1 开始，target 中的 length 字段改为使用 size 字段，从而使 target 符合通用命名法。在可预见的将来，两者将继续同时可用。较新的代码应该支持 size。
# 4. Envelope
envelope 通过下面的 JSON 结构包含一个或多个 event：
```
{
	"events": [ ... ],
}
```
虽然 event 可能在同一个 envelope 中发送，但 envelope 内的一组 event 没有隐含关系。例如，registry 可以选择对不相关的 event 进行分组，并将它们发送到同一 envelope 中以减少请求总数。

完整软件包的媒体类型为“application/vnd.docker.distribution.events.v1+json”，它在请求到达端点时就会设置。

event 完整的例子如下所示：
```
GET /callback
Host: application/vnd.docker.distribution.events.v1+json
Authorization: Bearer <your token, if needed>
Content-Type: application/vnd.docker.distribution.events.v1+json

{
   "events": [
      {
         "id": "asdf-asdf-asdf-asdf-0",
         "timestamp": "2006-01-02T15:04:05Z",
         "action": "push",
         "target": {
            "mediaType": "application/vnd.docker.distribution.manifest.v1+json",
            "length": 1,
            "digest": "sha256:fea8895f450959fa676bcc1df0611ea93823a735a01205fd8622846041d0c7cf",
            "repository": "library/test",
            "url": "http://example.com/v2/library/test/manifests/sha256:c3b3692957d439ac1928219a83fac91e7bf96c153725526874673ae1f2023f8d5"
         },
         "request": {
            "id": "asdfasdf",
            "addr": "client.local",
            "host": "registrycluster.local",
            "method": "PUT",
            "useragent": "test/0.1"
         },
         "actor": {
            "name": "test-actor"
         },
         "source": {
            "addr": "hostname.local:port"
         }
      },
      {
         "id": "asdf-asdf-asdf-asdf-1",
         "timestamp": "2006-01-02T15:04:05Z",
         "action": "push",
         "target": {
            "mediaType": "application/vnd.docker.container.image.rootfs.diff+x-gtar",
            "length": 2,
            "digest": "sha256:c3b3692957d439ac1928219a83fac91e7bf96c153725526874673ae1f2023f8d5",
            "repository": "library/test",
            "url": "http://example.com/v2/library/test/blobs/sha256:c3b3692957d439ac1928219a83fac91e7bf96c153725526874673ae1f2023f8d5"
         },
         "request": {
            "id": "asdfasdf",
            "addr": "client.local",
            "host": "registrycluster.local",
            "method": "PUT",
            "useragent": "test/0.1"
         },
         "actor": {
            "name": "test-actor"
         },
         "source": {
            "addr": "hostname.local:port"
         }
      },
      {
         "id": "asdf-asdf-asdf-asdf-2",
         "timestamp": "2006-01-02T15:04:05Z",
         "action": "push",
         "target": {
            "mediaType": "application/vnd.docker.container.image.rootfs.diff+x-gtar",
            "length": 3,
            "digest": "sha256:c3b3692957d439ac1928219a83fac91e7bf96c153725526874673ae1f2023f8d5",
            "repository": "library/test",
            "url": "http://example.com/v2/library/test/blobs/sha256:c3b3692957d439ac1928219a83fac91e7bf96c153725526874673ae1f2023f8d5"
         },
         "request": {
            "id": "asdfasdf",
            "addr": "client.local",
            "host": "registrycluster.local",
            "method": "PUT",
            "useragent": "test/0.1"
         },
         "actor": {
            "name": "test-actor"
         },
         "source": {
            "addr": "hostname.local:port"
         }
      }
   ]
}
```
# 5. 响应 Response
registry 接受来自端点的响应代码。如果端点响应任何 2xx 或 3xx 响应代码（在重定向之后），则认为该消息传递成功和丢弃。

反过来，建议端点也接受传入的响应。虽然 event envelope 的格式是通过媒体类型标准化的，但任何有关验证的“pickyness”（挑剔）都可能导致队列在 registry 中备份。
# 6. 监控 Monitoring
端点的状态通过 `debug/vars` http 接口报告，通常配置为 `http://localhost:5001/debug/vars`。终端可以获得配置和指标等信息。

以下提供了一些经历过几次失败并已经恢复的端点示例：
```
"notifications":{
   "endpoints":[
      {
         "name":"local-5003",
         "url":"http://localhost:5003/callback",
         "Headers":{
            "Authorization":[
               "Bearer \u003can example token\u003e"
            ]
         },
         "Timeout":1000000000,
         "Threshold":10,
         "Backoff":1000000000,
         "Metrics":{
            "Pending":76,
            "Events":76,
            "Successes":0,
            "Failures":0,
            "Errors":46,
            "Statuses":{

            }
         }
      },
      {
         "name":"local-8083",
         "url":"http://localhost:8083/callback",
         "Headers":null,
         "Timeout":1000000000,
         "Threshold":10,
         "Backoff":1000000000,
         "Metrics":{
            "Pending":0,
            "Events":76,
            "Successes":76,
            "Failures":0,
            "Errors":28,
            "Statuses":{
               "202 Accepted":76
            }
         }
      }
   ]
}
```
如果将通知用作较大应用程序的一部分，则监视端点队列的大小（上面的“Pending”）至关重要。如果故障或队列大小增加，则可能表明存在较大的问题。

日志也是监视问题的宝贵资源。失败的端点导致类似于以下内容的消息：
```
ERRO[0340] retryingsink: error writing events: httpSink{http://localhost:5003/callback}: error posting: Post http://localhost:5003/callback: dial tcp 127.0.0.1:5003: connection refused, retrying
WARN[0340] httpSink{http://localhost:5003/callback} encountered too many errors, backing off
```
以上表明几个错误导致退避（backoff）并且 registry 在重试之前等待。
# 7. 注意事项
目前，这些队列在内存中，所以端点应该可靠。它们会尽最大努力发送消息，但如果实例丢失，则可能会丢弃消息。如果端点关闭，应注意确保 registry 实例在端点恢复或消息丢失之前不可终止。

这可以通过在靠近 registry 实例的位置运行端点来缓解。可以运行一个端点将页面转到磁盘，然后转发请求以提供更好的耐用性。

通知系统是围绕一系列可互换的 sink（水槽）设计的，可以通过连接来实现有趣的行为。如果此系统不提供可接受的保证，则向 registry 添加交易 `Sink ` 是可能的，但它可能会影响请求服务时间（If this system doesn’t provide acceptable guarantees, adding a transactional Sink to the registry is a possibility, although it may have an effect on request service time）。有关更多信息，请参阅 [godoc](http://godoc.org/github.com/docker/distribution/notifications#Sink)。
