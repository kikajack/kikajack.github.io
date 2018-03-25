[原文地址](https://docs.docker.com/registry/configuration/)

Registry 的配置基于 YAML 文件，下面会详细描述。虽然它带有默认值，但你应该在将系统移至生产环境之前进行详尽的检查。
# 1. 覆盖特定的配置选项
在从官方镜像运行 registry 的典型设置中，可以通过将 `-e` 参数传递环境变量到 `docker run` 或使用 `ENV` 指令从 Dockerfile 中指定一个配置变量。

要覆盖配置选项，请创建一个名为 `REGISTRY_variable` 的环境变量，其中 `variable` 是配置选项的名称，`_`（下划线）表示缩进级别。例如，可以配置 `filesystem` 存储后端的 `rootdirectory`：
```
storage:
  filesystem:
    rootdirectory: /var/lib/registry
```
可以设置环境变量来覆盖这些值：
```
REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/somewhere
```
该变量将 `/var/lib/registry` 值覆盖到 `/somewhere` 目录。

>注意：使用可配置为调整单个值的环境变量创建基本配置文件。不建议使用环境变量覆盖配置节（section）。
# 2. 覆盖整个配置文件
如果默认配置不适合你的使用，或者如果覆盖环境中的 key 时遇到问题，则可以指定 YAML 配置文件并将其作为卷装载到容器中。

通常，从头开始创建一个名为 `config.yml` 的新配置文件，然后在 `docker run` 命令中指定它：
```
$ docker run -d -p 5000:5000 --restart=always --name registry \
             -v `pwd`/config.yml:/etc/docker/registry/config.yml \
             registry:2
```
以此示例 YAML 文件为起点。
# 3. 配置选项列表
这些都是 registry 的配置选项。列表中的某些选项是互斥的。在完成配置之前，阅读有关每个选项的详细参考信息。
```
version: 0.1
log:
  accesslog:
    disabled: true
  level: debug
  formatter: text
  fields:
    service: registry
    environment: staging
  hooks:
    - type: mail
      disabled: true
      levels:
        - panic
      options:
        smtp:
          addr: mail.example.com:25
          username: mailuser
          password: password
          insecure: true
        from: sender@example.com
        to:
          - errors@example.com
loglevel: debug # deprecated: use "log"
storage:
  filesystem:
    rootdirectory: /var/lib/registry
    maxthreads: 100
  azure:
    accountname: accountname
    accountkey: base64encodedaccountkey
    container: containername
  gcs:
    bucket: bucketname
    keyfile: /path/to/keyfile
    rootdirectory: /gcs/object/name/prefix
    chunksize: 5242880
  s3:
    accesskey: awsaccesskey
    secretkey: awssecretkey
    region: us-west-1
    regionendpoint: http://myobjects.local
    bucket: bucketname
    encrypt: true
    keyid: mykeyid
    secure: true
    v4auth: true
    chunksize: 5242880
    multipartcopychunksize: 33554432
    multipartcopymaxconcurrency: 100
    multipartcopythresholdsize: 33554432
    rootdirectory: /s3/object/name/prefix
  swift:
    username: username
    password: password
    authurl: https://storage.myprovider.com/auth/v1.0 or https://storage.myprovider.com/v2.0 or https://storage.myprovider.com/v3/auth
    tenant: tenantname
    tenantid: tenantid
    domain: domain name for Openstack Identity v3 API
    domainid: domain id for Openstack Identity v3 API
    insecureskipverify: true
    region: fr
    container: containername
    rootdirectory: /swift/object/name/prefix
  oss:
    accesskeyid: accesskeyid
    accesskeysecret: accesskeysecret
    region: OSS region name
    endpoint: optional endpoints
    internal: optional internal endpoint
    bucket: OSS bucket
    encrypt: optional data encryption setting
    secure: optional ssl setting
    chunksize: optional size valye
    rootdirectory: optional root directory
  inmemory:  # This driver takes no parameters
  delete:
    enabled: false
  redirect:
    disable: false
  cache:
    blobdescriptor: redis
  maintenance:
    uploadpurging:
      enabled: true
      age: 168h
      interval: 24h
      dryrun: false
    readonly:
      enabled: false
auth:
  silly:
    realm: silly-realm
    service: silly-service
  token:
    realm: token-realm
    service: token-service
    issuer: registry-token-issuer
    rootcertbundle: /root/certs/bundle
  htpasswd:
    realm: basic-realm
    path: /path/to/htpasswd
middleware:
  registry:
    - name: ARegistryMiddleware
      options:
        foo: bar
  repository:
    - name: ARepositoryMiddleware
      options:
        foo: bar
  storage:
    - name: cloudfront
      options:
        baseurl: https://my.cloudfronted.domain.com/
        privatekey: /path/to/pem
        keypairid: cloudfrontkeypairid
        duration: 3000s
  storage:
    - name: redirect
      options:
        baseurl: https://example.com/
reporting:
  bugsnag:
    apikey: bugsnagapikey
    releasestage: bugsnagreleasestage
    endpoint: bugsnagendpoint
  newrelic:
    licensekey: newreliclicensekey
    name: newrelicname
    verbose: true
http:
  addr: localhost:5000
  prefix: /my/nested/registry/
  host: https://myregistryaddress.org:5000
  secret: asecretforlocaldevelopment
  relativeurls: false
  tls:
    certificate: /path/to/x509/public
    key: /path/to/x509/private
    clientcas:
      - /path/to/ca.pem
      - /path/to/another/ca.pem
    letsencrypt:
      cachefile: /path/to/cache-file
      email: emailused@letsencrypt.com
  debug:
    addr: localhost:5001
  headers:
    X-Content-Type-Options: [nosniff]
  http2:
    disabled: false
notifications:
  endpoints:
    - name: alistener
      disabled: false
      url: https://my.listener.com/event
      headers: <http.Header>
      timeout: 500
      threshold: 5
      backoff: 1000
      ignoredmediatypes:
        - application/octet-stream
redis:
  addr: localhost:6379
  password: asecret
  db: 0
  dialtimeout: 10ms
  readtimeout: 10ms
  writetimeout: 10ms
  pool:
    maxidle: 16
    maxactive: 64
    idletimeout: 300s
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
  file:
    - file: /path/to/checked/file
      interval: 10s
  http:
    - uri: http://server.to.check/must/return/200
      headers:
        Authorization: [Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==]
      statuscode: 200
      timeout: 3s
      interval: 10s
      threshold: 3
  tcp:
    - addr: redis-server.domain.com:6379
      timeout: 3s
      interval: 10s
      threshold: 3
proxy:
  remoteurl: https://registry-1.docker.io
  username: [username]
  password: [password]
compatibility:
  schema1:
    signingkeyfile: /etc/registry/key.json
validation:
  enabled: true
  manifests:
    urls:
      allow:
        - ^https?://([^/]+\.)*example\.com/
      deny:
        - ^https?://www\.example\.com/
```
在某些情况下，配置选项是可选的，但它包含标记为必需的子选项。在这些情况下，可以忽略父选项及其所有子选项。但是，如果使用了父选项，则必须包括所有必选的子选项。
# 4. version
```
version: 0.1
```
`version` 选项是**必需**的。它指定配置的版本。它必须是顶级域，以便在解析配置文件的其余部分之前进行一致的版本检查。
# 5. log
日志子部分配置日志记录系统的行为。日志记录系统将所有内容输出到标准输出。可以使用此配置部分调整粒度和格式。
```
log:
  accesslog:
    disabled: true
  level: debug
  formatter: text
  fields:
    service: registry
    environment: staging
```
参数|	是否必须	|描述
-|-
`level`|	否|	设置记出日志的灵敏度。允许的值有 error，warn，info 和 debug。默认是 info。
`formatter`	|否	|选择日志输出的格式。该格式主要影响对日志行的键控属性进行编码的方式。选项是 text，json 和 logstash。默认值是 text。
`fields`	|否|	字段名称到值的映射。这些被添加到上下文的每个日志行。这对于在其他系统中混合后识别日志消息源很有用。
## 5.1 accesslog
```
accesslog:
  disabled: true
```
在 `log` 内，`accesslog` 配置访问日志记录系统的行为。默认情况下，访问日志记录系统以组合日志格式输出到标准输出。可以通过将布尔标志 `disabled` 设置为 `true` 来禁用访问日志记录。
# 6. hooks
```
hooks:
  - type: mail
    levels:
      - panic
    options:
      smtp:
        addr: smtp.sendhost.com:25
        username: sendername
        password: password
        insecure: true
      from: name@sendhost.com
      to:
        - name@receivehost.com
```
`hooks` 子部分配置日志 hooks 的行为。例如，该小节包含一个序列处理程序，可以使用它来发送邮件。请参阅 `loglevel` 以配置打印的消息级别。
# 7. loglevel
>废弃：请使用 log。
```
loglevel: debug
```
允许值有 error, warn, info 和 debug。默认是 info。
# 8. storage
```
storage:
  filesystem:
    rootdirectory: /var/lib/registry
  azure:
    accountname: accountname
    accountkey: base64encodedaccountkey
    container: containername
  gcs:
    bucket: bucketname
    keyfile: /path/to/keyfile
    rootdirectory: /gcs/object/name/prefix
  s3:
    accesskey: awsaccesskey
    secretkey: awssecretkey
    region: us-west-1
    regionendpoint: http://myobjects.local
    bucket: bucketname
    encrypt: true
    keyid: mykeyid
    secure: true
    v4auth: true
    chunksize: 5242880
    multipartcopychunksize: 33554432
    multipartcopymaxconcurrency: 100
    multipartcopythresholdsize: 33554432
    rootdirectory: /s3/object/name/prefix
  swift:
    username: username
    password: password
    authurl: https://storage.myprovider.com/auth/v1.0 or https://storage.myprovider.com/v2.0 or https://storage.myprovider.com/v3/auth
    tenant: tenantname
    tenantid: tenantid
    domain: domain name for Openstack Identity v3 API
    domainid: domain id for Openstack Identity v3 API
    insecureskipverify: true
    region: fr
    container: containername
    rootdirectory: /swift/object/name/prefix
  oss:
    accesskeyid: accesskeyid
    accesskeysecret: accesskeysecret
    region: OSS region name
    endpoint: optional endpoints
    internal: optional internal endpoint
    bucket: OSS bucket
    encrypt: optional data encryption setting
    secure: optional ssl setting
    chunksize: optional size valye
    rootdirectory: optional root directory
  inmemory:
  delete:
    enabled: false
  cache:
    blobdescriptor: inmemory
  maintenance:
    uploadpurging:
      enabled: true
      age: 168h
      interval: 24h
      dryrun: false
    readonly:
      enabled: false
  redirect:
    disable: false
```
存储选项是必需的，它定义了哪个存储后端正在使用中。必须完全配置一个后端。如果配置了多个存储后端，则 registry 会返回错误。可以选择以下任何后端存储驱动程序：

存储驱动程序|	描述
-|-
`filesystem`|	使用本地磁盘来存储 registry 文件。它非常适合开发，可能适用于一些小规模生产应用。请参阅 [驱动程序的参考文档](https://github.com/docker/docker.github.io/tree/master/registry/storage-drivers/filesystem.md)。
`azure`|	使用 Microsoft Azure Blob Storage。参考 [驱动程序的手册](https://github.com/docker/docker.github.io/tree/master/registry/storage-drivers/azure.md)。
`gcs`|	使用 Google Cloud Storage。参考 [驱动程序的手册](https://github.com/docker/docker.github.io/tree/master/registry/storage-drivers/gcs.md)。
`s3`|	使用 Amazon Simple Storage Service (S3) 和 compatible Storage Services。参考 [驱动程序的手册](https://github.com/docker/docker.github.io/tree/master/registry/storage-drivers/s3.md)。
`swift`|	使用 Openstack Swift object storage。参考 [驱动程序的手册](https://github.com/docker/docker.github.io/tree/master/registry/storage-drivers/swift.md)。
`oss`|	使用 Aliyun OSS for object storage。参考 [驱动程序的手册](https://github.com/docker/docker.github.io/tree/master/registry/storage-drivers/oss.md)。
仅用于测试的话，可以使用 inmemory [存储驱动程序](https://github.com/docker/docker.github.io/tree/master/registry/storage-drivers/inmemory.md)。如果你想从易失性存储器运行 registry，请在 ramdisk 上使用 [文件系统驱动程序](https://github.com/docker/docker.github.io/tree/master/registry/storage-drivers/filesystem.md)。

如果在 Windows 上部署 registry，则不推荐从主机挂载 Windows 卷。相反，可以使用 S3 或 Azure 支持数据存储。如果确实使用 Windows 卷，则到安装点的 `PATH` 长度必须在 `MAX_PATH` 限制（通常为255个字符）内，否则会发生此错误：
```
mkdir /XXX protocol error and your registry will not function properly.
```
## 8.1 maintenance
目前，上传清除和只读模式是唯一可用的 `maintenance`（维护）功能。
## 8.2 uploadpurging
上传清除（uploadpurging）是一个后台进程，可以定期从 registry 的上传目录中删除孤立的文件。上传清除默认情况下处于启用状态。要配置上传目录清除，必须设置以下参数。

参数|	是否必须|	描述
-|-
`enabled`|	是	|默认 true。设置为 true 则开启上传清除。
`age`|	是|	默认 168 小时，即 1 周。上传目录大于这个时间后，删除上传目录。`interval`|	是|	上传目录清除的时间间隔。 默认为24小时。
`dryrun`|	是	|将 `dryrun` 设置为 `true` 以获取要删除的目录的摘要。默认为 false。
>注意：`age` 和 `interval` 是包含具有可选的分数和单位后缀的数字的字符串。例如：45m，2h10m，168h。
## 8.3 readonly
如果将 `maintenance` 下的 `readonly` 部分的 `enabled` 设置为 `true`，则不允许客户端写入 registry。此模式对于临时阻止写入后端存储非常有用，此时可以运行垃圾回收。在运行垃圾回收之前，registry 应该在 `readonly` 的 `enabled` 设置为 true 的情况下重新启动。垃圾收集过程完成后，registry 可能会重新启动，这次只能从配置中只读取（或设置为 false）。
## 8.4 delete
使用 `delete` 可以通过摘要删除镜像的 blob 和清单（Use the delete structure to enable the deletion of image blobs and manifests by digest）。默认是 false，但可以通过在配置文件中写入以下内容来启用它：
```
delete:
  enabled: true
```
## 8.5 cache
使用 `cache` 来启用在存储后端访问的数据的缓存。目前，唯一可用的缓存提供对层元数据（layer metadata）的快速访问，层元数据使用 `blobdescriptor` 字段（如果已配置）。

可以将 `blobdescriptor` 字段设置为 `redis` 或 `inmemory`。如果设置为 `redis`，则 Redis 池会缓存层元数据。如果设置为 `inmemory`，则内存映射会缓存层元数据。

>注意：以前，`blobdescriptor` 被称为 `layerinfo`。虽然这是一样的，但 `layerinfo` 已被弃用。
## 8.6 redirect
`redirect` 子部分提供了用于管理来自内容后端的重定向的配置。对于支持它的后端，默认情况下启用重定向。在某些部署方案中，你可能决定将所有数据路由到 registry，而不是重定向到后端。当使用不共存的后端或者当 registry 实例积极缓存时，这可能会更有效（This may be more efficient when using a backend that is not co-located or when a registry instance is aggressively caching）。

要禁用重定向，请在 `redirect` 下添加一个 `disable` 标志，将其设置为 true：
```
redirect:
  disable: true
```
# 9. auth
```
auth:
  silly:
    realm: silly-realm
    service: silly-service
  token:
    realm: token-realm
    service: token-service
    issuer: registry-token-issuer
    rootcertbundle: /root/certs/bundle
  htpasswd:
    realm: basic-realm
    path: /path/to/htpasswd
```
`auth` 选项是可选的。可能的 `auth` provider 包括：

- silly
- token
- htpasswd

只能配置一个 `auth` provider 。
## 9.1 silly
`silly` 身份验证程序只适合开发。它只是检查 HTTP 请求中是否存在 `Authorization` 标头。它不检查 header 的值。如果 header 不存在，那么 `silly` 认证返回 challenge 响应，回应拒绝访问的 realm，service 和 scope。

以下值用于配置响应：

参数|	是否必须|	描述
-|-
`realm`|	是|	注册表服务器进行身份验证的 realm （领域）。
`service`|	是|	进行身份验证的服务。
## 9.2 token
基于 token 令牌的身份验证允许你将身份验证系统与 registry 分离。这是一个高度安全的认证范例。

参数|	是否必须	|描述
-|-
`realm`|	是	|注册表服务器进行身份验证的 realm （领域）。
`service`|	是|	进行身份验证的服务。
`issuer`|	是	|token 令牌发行者的名称。发行者将其插入令牌，因此它必须匹配为发行者配置的值（The issuer inserts this into the token so it must match the value configured for the issuer）。
`rootcertbundle`|	是|	根证书包的绝对路径。该捆绑包（bundle）包含用于签署认证令牌的证书的公共部分。
有关基于令牌的认证配置的更多信息，请参阅 [规范](https://docs.docker.com/registry/spec/auth/token/)。
## 9.3 htpasswd
支持的 `htpasswd` 认证允许使用 [Apache htpasswd 文件](https://httpd.apache.org/docs/2.4/programs/htpasswd.html) 配置基本认证。唯一支持的密码格式是 `bcrypt`。其他散列类型的条目将被忽略。`htpasswd`文件在启动时加载一次。如果文件无效，registry 将显示错误并且不会启动。

>警告：由于基本身份验证将密码作为 HTTP header 的一部分发送，因此只能使用配置了 TLS 的 `htpasswd` 身份验证方案。

参数|	是否必须|	描述
-|-
`realm`|	是|	注册表服务器进行身份验证的 realm （领域）。
`path`|	是|	启动时加载的 `htpasswd` 文件的路径。
# 10. middleware
`middleware` 部分是可选的。使用此选项在指定的 hook point 处注入中间件。每个中间件都必须实现与它所包装的对象相同的接口。例如，registry 中间件必须实现 `distribution.Namespace` 接口，repository 中间件必须实现`distribution.Repository`，而 storage 中间件必须实现 `driver.StorageDriver`。

这是 `cloudfront` 中间件（存储中间件）的示例配置：
```
middleware:
  registry:
    - name: ARegistryMiddleware
      options:
        foo: bar
  repository:
    - name: ARepositoryMiddleware
      options:
        foo: bar
  storage:
    - name: cloudfront
      options:
        baseurl: https://my.cloudfronted.domain.com/
        privatekey: /path/to/pem
        keypairid: cloudfrontkeypairid
        duration: 3000s
```
每个中间件条目都有 `name` 和 `options` 条目。该 `name` 必须与中间件 registry 自身的名称相对应。`options` 字段是一个映射，用于详细说明初始化中间件所需的自定义配置。它被视为一个 `map[string]interface{}`。因此，它支持所需的任何有趣的结构，将其留给中间件初始化函数以最好地确定如何处理选项的特定解释（As such, it supports any interesting structures desired, leaving it up to the middleware initialization function to best determine how to handle the specific interpretation of the options.）。
## 10.1 cloudfront

参数	|是否必须|	描述
-|-
`baseurl`|	是|	`SCHEME://HOST[/PATH]`，Cloudfront 提供服务的 url。
`privatekey`|	是|	Cloudfront 的私钥，由 AWS 提供。
`keypairid`|	是|	由 AWS 提供的 key pair ID。
`duration`|	否	|Cloudfront 会话持续时间，由整数和单位组成。有效时间单位是ns，us（或μs），ms，s，m 或 h。例如，`3000s` 是有效的，但 `3000 s` 不是。如果不指定持续时间或者指定一个没有时间单位的整数，则持续时间默认为`20m`（20分钟）。
## 10.2 redirect
可以使用重定向存储中间件为 S3 存储驱动程序存储的层的代理位置指定自定义 URL（You can use the redirect storage middleware to specify a custom URL to a location of a proxy for the layer stored by the S3 storage driver）。

参数|	是否必须|	描述
-|-
`baseurl`|	是|	`SCHEME://HOST`，层的保存位置。例如，`https://example.com:5443`
# 11. reporting
```
reporting:
  bugsnag:
    apikey: bugsnagapikey
    releasestage: bugsnagreleasestage
    endpoint: bugsnagendpoint
  newrelic:
    licensekey: newreliclicensekey
    name: newrelicname
    verbose: true
```
`reporting` 选项是可选的，并配置错误和指标报告工具。目前只支持两种服务：

- Bugsnag
- New Relic

有效的配置可以同时包含这两个服务。
## 11.1 bugsnag
参数|	是否必须	|描述
-|-
`apikey`|	是|	 由 Bugsnag 提供的 API Key。
`releasestage`|	否|	跟踪部署 registry 的位置，使用像 `production`，`staging` 或 `development` 这样的字符串。
`endpoint`|	否|	企业 Bugsnag 端点（The enterprise Bugsnag endpoint）。
## 11.2 newrelic
参数|	是否必须	描述
-|-
`licensekey`|	是|	New Relic 提供的 License key。
`name`|	否	|New Relic 应用名称。
`verbose`|	否|	设置为 true 以在标准输出上启用 New Relic 调试输出。
# 12. http
```
http:
  addr: localhost:5000
  net: tcp
  prefix: /my/nested/registry/
  host: https://myregistryaddress.org:5000
  secret: asecretforlocaldevelopment
  relativeurls: false
  tls:
    certificate: /path/to/x509/public
    key: /path/to/x509/private
    clientcas:
      - /path/to/ca.pem
      - /path/to/another/ca.pem
    letsencrypt:
      cachefile: /path/to/cache-file
      email: emailused@letsencrypt.com
  debug:
    addr: localhost:5001
  headers:
    X-Content-Type-Options: [nosniff]
  http2:
    disabled: false
```
`http` 选项详细配置了 registry 所在的 HTTP 服务器。

参数|	是否必须|	描述
-|-
`addr`|	是|	服务器应该接受连接的地址。格式取决于网络类型（参考 `net` 选项）。对于 TCP 套接字使用 `HOST:PORT`，对于 UNIX 套接字使用 `FILE`。
`net`|	否	|用于创建侦听套接字的网络。可以使用 `unix` 和 `tcp`。
`prefix`|	否|如果服务器未在根路径上运行，用这个参数设置前缀的值。根路径是 `v2` 之前的部分。它需要前斜杠和后斜线，例如 `/path/`。
`host`|	否|	registry 的外部可达地址的完全限定 URL。如果存在，则在创建生成的URL时使用（A fully-qualified URL for an externally-reachable address for the registry. If present, it is used when creating generated URLs）。否则，这些 URL 从客户端请求派生。
`secret`|	否|	用于签署状态的随机数据，可与客户端一起存储以防止篡改。对于生产环境，应该使用密码安全的随机生成器生成随机数据。如果省略 `secret`，registry 将在启动时自动生成一个 `secret`。如果要在负载均衡器后面建立一个 registry 集群，**必须**确保所有 registry 的 `secret` 都一样。
`relativeurls`|	否|	如果为 true，则 registry 将返回 Location header 中的相对 URL。客户端负责解析正确的 URL。该选项与 Docker 1.7 及更早版本不兼容。
## 12.1 tls
`http` 中的 `tls` 选项是可选的，用于配置服务器的 TLS。如果在 registry 所在主机上已经运行了一个 web 服务器，则最好为 web 服务器配置 TLS 并代理到 registry 服务器的连接。

参数|	是否必须|	描述
-|-
`certificate`|	是|	x509 证书文件（certificate file）的绝对路径。
`key`|	是|	x509 私钥文件（private key file）的绝对路径。
`clientcas`|	否|	x509 CA 文件的绝对路径的数组。
## 12.2 letsencrypt
`tls` 中的 `letsencrypt` 是可选的，用于配置由 [Let’s Encrypt](https://letsencrypt.org/how-it-works/) 提供的 TLS 证书。

>注意：使用 Let's Encrypt 时，请确保端口 443 可从外部访问。registry 默认侦听端口 5000。如果将 registry 作为容器运行，请考虑将标志 `-p 443:5000` 添加到  `docker run` 命令或在云配置中使用类似的设置。

参数|	是否必须	|描述
-|-
`cachefile`|	是|	Let’s Encrypt 客户端可以缓存数据的绝对路径。
`email`|	是|	用于注册 Let's Encrypt 的电子邮件地址。
## 12.3 debug
`debug` 选项是可选的，用于配置可以帮助诊断问题的调试服务器。调试端点可用于监控 registry 指标和运行状况以及性能分析。敏感信息可能通过调试端点提供。请确保访问调试端点的权限在生产环境中被锁定。

`debug` 部分采用一个必需的 `addr` 参数，该参数指定调试服务器应该接受来自哪个 `HOST:PORT` 的连接。
## 12.4 headers
`headers` 选项是可选的，用来指定 HTTP 服务器应该包含在响应中的 header。这可以用于安全头如 `Strict-Transport-Security`。

`headers` 选项应该包含每个 header 包含的选项，其中参数名称是 header 名称，参数值是 header 的有效负载值列表。（The headers option should contain an option for each header to include, where the parameter name is the header’s name, and the parameter value a list of the header’s payload values）

推荐包括 `X-Content-Type-Options: [nosniff]`，以便浏览器不会将内容解释为 HTML，如果它们被指示从 registry 中加载页面。该头文件包含在示例配置文件中。
## 12.5 http2
`http` 中的 `http2` 选项是可选的，用于控制 registry 的 HTTP2 设置。

参数|	是否必须|	描述
-|-
`disabled`|	否|	如果是 true，则关闭 HTTP2 支持。
# 13. notifications
```
notifications:
  endpoints:
    - name: alistener
      disabled: false
      url: https://my.listener.com/event
      headers: <http.Header>
      timeout: 500
      threshold: 5
      backoff: 1000
      ignoredmediatypes:
        - application/octet-stream
```
`notifications` 选项是可选的，目前可能包含一个选项 `endpoints`。
## 13.1 endpoints
`endpoints` 结构包含一个命名过的服务（URL）的列表，可以接受事件通知。

参数	|是否必须|	描述
-|-
`name`|	是	|可读的服务名
`disabled`|	否|	如果是 true，禁止向服务发送通知。
`url`|	是|	事件发布到该 URL。
`headers`|	是|	要添加到每个请求的静态 header 列表。每个 header 的名称是 header 下的关键字，每个值都是该标题名称的有效负载列表（Each header’s name is a key beneath headers, and each value is a list of payloads for that header name）。值必须始终为列表。
`timeout`|	是|	HTTP 超时的值。表示时间的正整数和可选的单位后缀，可以是 ns，us，ms，s，m 或 h。如果省略时间单位，则使用 ns。
`threshold`|	是|	一个整数，指定在退出故障之前等待多久。
`backoff`|	是|	系统在发生故障后重试之前需要等待多久。表示时间单位的正整数和可选后缀，可以是 ns，us，ms，s，m 或 h。如果省略时间单位，则使用ns。
`ignoredmediatypes`|	否	|要忽略的目标媒体类型列表。这些目标媒体类型的事件不会发布到端点。
# 14. redis
```
redis:
  addr: localhost:6379
  password: asecret
  db: 0
  dialtimeout: 10ms
  readtimeout: 10ms
  writetimeout: 10ms
  pool:
    maxidle: 16
    maxactive: 64
    idletimeout: 300s
```
声明构建 redis 连接的参数。registry 实例可以将 Redis 实例用于多个应用程序。目前，它缓存有关不可变 blob 的信息。大多数 redis 选项控制 registry 如何连接到 redis 实例。要控制 pool 的行为可以参考下一小节 pool。

应该使用 **allkeys-lru** 策略（allkeys-lru eviction policy）来配置 Redis，因为 registry 不会在密钥上设置过期值。

参数	|是否必须	|描述
-|-
`addr`|	是|	Redis 实例的地址（主机和端口）。
`password`|	否	|用于 Redis 实例认证的密码。
`db`|	否|	用于每个连接的数据库名。
`dialtimeout`|	否|	连接到 Redis 实例的超时时间。
`readtimeout`|	否|	从 Redis 实例的读操作超时时间。
`writetimeout`|	否|	向 Redis 实例的写操作超时时间。
## 14.1 pool
```
pool:
  maxidle: 16
  maxactive: 64
  idletimeout: 300s
```
这些设置用于配置 Redis 连接池的表现。

超时|	是否必须	|描述
-|-
`maxidle`|	否|	连接池中的最大空闲连接数。
`maxactive`|	否|	在阻止连接请求之前可以打开的最大连接数。
`idletimeout`|	否|	关闭不活动的连接之前的等待时间。
# 15. health
```
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
  file:
    - file: /path/to/checked/file
      interval: 10s
  http:
    - uri: http://server.to.check/must/return/200
      headers:
        Authorization: [Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==]
      statuscode: 200
      timeout: 3s
      interval: 10s
      threshold: 3
  tcp:
    - addr: redis-server.domain.com:6379
      timeout: 3s
      interval: 10s
      threshold: 3
```
`health` 选项是可选的，包含用于定期对存储驱动程序后端存储进行健康状况检查的首选项，以及对本地文件，HTTP URI 和 TCP 服务器的定期检查选项。如果启用了调试 HTTP 服务器，则可以在调试 HTTP 服务器上的 `/debug/health` 端点上使用健康检查的结果（请参阅 http 部分）。
## 15.1 storagedriver
`storagedriver` 结构包含用于对配置的存储驱动程序的后端存储进行运行状况检查的选项。只有在 `enabled` 设置为 true 时，健康检查才会激活。

超时|	是否必须|	描述
-|-
`enabled`|	是|	设置为 true 以启用存储驱动程序的状况检查，设置为 false 则禁用。
`interval`|	否|	存储驱动程序运行两次状况检查之间需要等待多长时间。一个正整数和一个可选的时间单位后缀。后缀是 ns，us，ms，s，m 或 h。如果该值被省略，则默认为 10 秒。如果指定了一个值但省略了后缀，则该值将被解释为一个纳秒数。
`threshold`|	否|	正整数，表示在状态标记为不健康之前必须失败的检查次数。如果没有指定，则一次故障就会使标志变为不健康状态。
## 15.2 file
`file` 结构包括一个要定期检查文件是否存在的路径列表。如果文件存在于给定路径中，则运行状况检查将失败。可以使用此机制通过创建文件来使 registry 脱离轮换（You can use this mechanism to bring a registry out of rotation by creating a file）。

参数|	是否必须|	描述
-|-
`file`|	是	|在这个路径上检查文件是否存在。
`interval`|	否|	重复检查的间隔时间。一个正整数和一个可选的时间单位后缀。后缀是 ns，us，ms，s，m 或 h 中的一个。如果该值被省略，则默认为 10 秒。
## 15.3 http
`http` 结构包含一个 HTTP URI 列表，用于定期检查 HEAD 请求。如果 HEAD 请求未完成或返回非预期状态码，则状况检查将失败。

参数	|是否必须|	描述
-|-
`uri`|	是	|要检查的 URI。
`headers`|	否|	要添加到每个请求的静态 header。每个 header 的名称是 header 下的关键字，每个值都是该 header 名称的有效负载列表（Each header’s name is a key beneath headers, and each value is a list of payloads for that header name）。值必须始终为列表。
`statuscode`|	否|	来自 HTTP URI 的预期状态码。默认为 200。
`timeout`|	否|	在 HTTP 请求超时之前等多久。一个正整数和一个可选的时间单位后缀。后缀是 ns，us，ms，s，m 或 h。如果指定了一个值但省略了后缀，则该值将被解释为一个纳秒数。
`interval`|	否|	两次检查之间的间隔。一个正整数和一个可选的时间单位后缀。后缀是 ns，us，ms，s，m 或 h。如果该值被省略，则默认为 10 秒。如果指定了一个值但省略了后缀，则该值将被解释为一个纳秒数。
`threshold`|	否	|在状态标记为不健康之前失败检查必须达到的次数。如果未指定此字段，则一次故障就会将状态标记为不健康。
## 15.4 tcp
`tcp` 结构包含一个 TCP 地址列表，用于定期使用 TCP 连接进行检查。地址必须包含端口号。如果连接尝试失败，健康检查将失败。

参数|	是否必须|	描述
-|-
`addr`|	是|	要连接到的 TCP 地址和端口。
`timeout`|	否|	TCP 连接超时时间。一个正整数和一个可选的时间单位后缀。后缀是 ns，us，ms，s，m 或 h。如果指定了一个值但省略了后缀，则该值将被解释为一个纳秒数。
`interval`|	否|	两次检查之间的间隔。一个正整数和一个可选的时间单位后缀。后缀是 ns，us，ms，s，m 或 h。如果该值被省略，则默认为 10 秒。如果指定了一个值但省略了后缀，则该值将被解释为一个纳秒数。
`threshold`|	否|	在状态标记为不健康之前失败检查必须达到的次数。如果未指定此字段，则一次故障就会将状态标记为不健康。
# 16. proxy
```
proxy:
  remoteurl: https://registry-1.docker.io
  username: [username]
  password: [password]
```
`proxy` 结构允许将 registry 配置为 Docker Hub 的直通缓存（a pull-through cache）。更多信息请参阅 [镜像](https://github.com/docker/docker.github.io/tree/master/registry/recipes/mirror.md)。不支持向配置为直通式高速缓存的 registry 的 push 操作。

参数|	是否必须	|描述
-|-
`remoteurl`|	是	|Docker Hub 上的仓库的 URL。
`username`|	否	|注册到可访问仓库的 Docker Hub 的用户名。
`password`|	否|	用于使用 `username` 中指定的用户名对 Docker Hub 进行身份验证的密码。
要启用私人仓库（例如 batman/robin），请指定用户名（例如 batman）和该用户名的密码。

>注意：这些私人仓库存储在代理缓存的存储中。采取适当措施保护对代理缓存的访问。
# 17. compatibility
```
compatibility:
  schema1:
    signingkeyfile: /etc/registry/key.json
```
使用 `compatibility` 结构来配置处理旧的和弃用的功能。每个小节都定义了这种具有可配置行为的功能。
## 17.1 schema1
参数|	是否必须	|描述
-|-
`signingkeyfile`|	否|	用于向 schema1 清单添加签名的签名私钥。如果未提供签名密钥，则在 registry 启动时会生成新的 ECDSA 密钥。
# 18. validation
```
validation:
  enabled: true
  manifests:
    urls:
      allow:
        - ^https?://([^/]+\.)*example\.com/
      deny:
        - ^https?://www\.example\.com/
```
## 18.1 enabled
使用 `enabled` 标志启用 `validation` 部分中的其他选项。它们默认是禁用的。
## 18.2 manifests
使用 `manifest` 子部分来配置清单验证。

#### URLS
`allow` 和 `deny` 选项每个都是限制推送清单中的URL的 [正则表达式](https://godoc.org/regexp/syntax)列表。 

如果没有设置 `allow`，向清单中包含的 URL 进行 push 操作会失败。

如果设置了 `allow`，则只有在所有 URL 匹配其中一个允许的正则表达式并且以下情况之一成立时才 push manifest：

1. 没设置 `deny`
2. `deny` 设置了但是 manifest 清单中没有 URL 匹配 `deny` 正则表达式
# 19. 示例：部署配置
以下示例可以用于本地配置：
```
version: 0.1
log:
  level: debug
storage:
    filesystem:
        rootdirectory: /var/lib/registry
http:
    addr: localhost:5000
    secret: asecretforlocaldevelopment
    debug:
        addr: localhost:5001
```
本示例将 registry 实例配置为在端口 5000 上运行，绑定到 localhost，并启用调试服务器。registry 数据存储在 `/var/lib/registry` 目录中。日志记录被设置为调试模式，这是最详细的。

有关其他简单配置，请参阅 [config-example.yml](https://github.com/docker/distribution/blob/master/cmd/registry/config-example.yml)。这两个例子通常对本地开发很有用。
# 20. 示例：中间件配置
本示例将 [Amazon Cloudfront](http://aws.amazon.com/cloudfront/) 配置为注册表中的存储中间件。中间件允许 registry 通过内容交付网络（CDN）提供服务。这减少了对存储层的请求。

Cloudfront需要S3存储驱动程序。

这是YAML中的配置：
Cloudfront requires the S3 storage driver.

This is the configuration expressed in YAML:
```
middleware:
  storage:
  - name: cloudfront
    disabled: false
    options:
      baseurl: http://d111111abcdef8.cloudfront.net
      privatekey: /path/to/asecret.pem
      keypairid: asecret
      duration: 60
```
有关配置选项的更多信息，请参阅 Cloudfront 的配置参考。

>注意：Cloudfront 密钥与其他 AWS 密钥分开存在。有关更多信息，请参阅 [AWS 凭证上的文档](http://docs.aws.amazon.com/general/latest/gr/aws-security-credentials.html)。
