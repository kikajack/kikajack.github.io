GraphQL HTTP 服务器一般都可以处理 HTTP 的 POST 方法，有的还可以处理 GET 方法。
#1. GET 请求 
用 GET 请求查询 GraphQL 服务器时，应当将查询的文档，变量和操作名称作为 GET 参数传给服务器。
完整的参数格式：`?query=查询文档&variables=变量&operationName=操作名称`。

- query：查询文档，必填。
- variables：变量，选填。
- operationName：操作名称，选填，查询文档有多个操作时必填。

```
// 请求
{
  viewer {
    name
  }
}
```
对应的通过 HTTP GET 发送的示例：
`http://api.com/graphql?query={viewer{name}}`

#2. POST 请求 
标准的 GraphQL POST 请求应当在 HTTP header 中声明 `Content-Type: application/json`，并且使用 JSON 格式的内容。
POST 报文体中的 JSON 数据中的三个字段跟 GET 请求类似：

- query：查询文档，必填。
- variables：变量，选填。
- operationName：操作名称，选填，查询文档有多个操作时必填。
```
{
  "query": "{viewer{name}}",
  "operationName": "",
  "variables": { "name": "value", ... }
}
```
#3. 响应 
无论用 GET 还是 POST 方法查询，响应一般都会以 JSON 格式在 POST 的报文体返回。[响应的规范参考这里](http://blog.csdn.net/kikajack/article/details/79109444#t6)，正常响应的数据放在 data 属性中，异常时记录在 errors 属性中：
```
{
  "data": { ... },
  "errors": [ ... ]
}
```