登录 GitHub 后，[可以在这个地址在线验证](https://developer.github.com/v4/explorer/)。
[参考资料](http://graphql.cn/learn/queries/)
![GitHub GraphQL 接口在线验证地址](http://img.blog.csdn.net/20180117110034883?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#1. Fields（字段）
```
// 请求
{
  viewer {
    id
    email
    repositories {
      totalCount
    }
  }
}

// 返回
{
  "data": {
    "viewer": {
      "id": "MDQ6VXNlcjE4NTMyODU5",
      "email": "",
      "repositories": {
        "totalCount": 18
      }
    }
  }
}
```
查询和其结果拥有几乎一样的结构。
这个例子中，查询的是 viewer 中的 name，email，repositories 三个字段。其中，repositories 字段是对象类型（Object），可以对这个对象的字段进行次级选择。GraphQL 查询能够遍历相关对象及其字段，使得客户端可以一次请求查询大量相关数据，而不像传统 REST 架构中那样需要多次往返查询。
#2. Arguments（参数）
```
// 请求
{
  viewer {
    repository(name: "kikajack.github.io") {
      id
      name
      diskUsage
    }
  }
}

// 返回
{
  "data": {
    "viewer": {
      "repository": {
        "id": "MDEwOlJlcG9zaXRvcnkxMDg4NTg3ODE=",
        "name": "kikajack.github.io",
        "diskUsage": 3205
      }
    }
  }
}
```
Arguments 的识别，需要服务端支持。
在 GraphQL 中，每一个字段和嵌套对象都能有自己的一组参数。也可以给 标量（scalar）字段传递参数（比如指定单位），服务端的可以根据参数返回指定格式的标量数据。
#3. Aliases（别名）
```
// 请求
{
  viewer {
    myid: id
    email
    repositories {
      totalCount
    }
  }
}

// 返回
{
  "data": {
    "viewer": {
      "myid": "MDQ6VXNlcjE4NTMyODU5",
      "email": "",
      "repositories": {
        "totalCount": 18
      }
    }
  }
}
```
发请求给 GraphQL 服务器时，可以给字段起别名，返回的数据中的对应字段就用这个别名作为属性名。
#4. Fragments（片段）
```
// 请求
{
  viewer {
    ...myFragement1
  }
  user(login:"kikajack") {
    ...myFragement1
  }
}

fragment myFragement1 on User{
  login
  id
}
// 返回
{
  "data": {
    "viewer": {
      "login": "kikajack",
      "id": "MDQ6VXNlcjE4NTMyODU5"
    },
    "user": {
      "login": "kikajack",
      "id": "MDQ6VXNlcjE4NTMyODU5"
    }
  }
}
```
将多次用到的字段，组织为片段（可复用单元）。在需要的地方引入即可。
片段的 on 后面的字段是接口（Interface）或类（Object）。请求中，只有在模式继承这个接口或实现这个类时，内部字段位置才可以使用这个片段。例如把片段中的 User 改为其他无关类或接口，会提示错误信息。
#5. Operation type & name（操作类型和操作名称）
```
// 请求
query myQuery{
  viewer {
    ...myFragement1
  }
  user(login:"kikajack") {
    ...myFragement1
  }
}

fragment myFragement1 on User {
  login
  id
}
```
**操作类型**可以是 query、mutation 或 subscription，只有 query 可以省略。

**操作名称**是你的操作的有意义和明确的名称。对于调试和日志有用。 GraphQL 的查询、变更以及片段名称，都可以成为服务端侧用来识别不同 GraphQL 请求的有效调试工具。
#6. Variables（变量）
```
// 请求，变量类型（AddReactionInput 的位置）必须是标量、枚举型或输入对象类型（由 Schema 指定），变量 $myVar 必须实现该类型，且变量中的属性只能从该类型中选。叹号表示必传，也是由 Schema 指定的
query myQuery($login:String!){
  user(login:$login) {
    login
    id
  }
}
// 查询变量 query variables
{
  "login":"kikajack"
}
// 返回
{
  "data": {
    "user": {
      "login": "kikajack",
      "id": "MDQ6VXNlcjE4NTMyODU5"
    }
  }
}
```
```
// 传输的 POST 数据（Content-Type:application/json），有 query 和 variables 两个属性：
{"query":"mutation($myVar:AddReactionInput!) {\n  addReaction(input:$myVar) {\n    reaction {\n      content\n    }\n    subject {\n      id\n    }\n  }\n}\n","variables":"{\n  \"myVar\": {\n    \"subjectId\": \"MDU6SXNzdWUyMTc5NTQ0OTc=\",\n    \"content\": \"HOORAY\"\n  }\n}"}
```
![带参数的 schema](http://img.blog.csdn.net/20180117111537537?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
对于简单参数或固定参数，可以写在查询字符串内（`user(login:"kikajack"){...}`）。
对于复杂或**动态参数**，可以在外部定义符合对应 Input Type 类型的 JSON，查询的时候和 query 或 mutation 一起发送即可。这样查询语句只需要写一次，每次重新组织变量即可。例如：`{"query":"mutation($myVar:AddReactionInput!) {//固定}","variables":"{//每次重新组织}"}`
### 默认变量（Default variables） 
在查询中的类型定义后面可以附带默认值。
```
query myQuery($login:String = "kikajack"){
  user(login:$login) {
    login
    id
  }
}

```
有默认值的时候，可以不传变量直接调用查询。
#7. Directives（指令）
```
// 请求
query myQuery($login:String = "kikajack",$showID:Boolean!){
  user(login:$login) {
    login
    id @include(if: $showID)
  }
}
// 参数
{
  "login":"kikajack",
  "showID": false
}
// 返回
{
  "data": {
    "user": {
      "login": "kikajack"
    }
  }
}
// POST 数据
{"query":"query myQuery($login:String = \"kikajack\",$showID:Boolean!){\n  user(login:$login) {\n    login\n    id @include(if: $showID)\n  }\n}\n","variables":"{\n  \"login\":\"kikajack\",\n  \"showID\": false\n}","operationName":"myQuery"}
```
```
// 请求
query myQuery($login:String = "kikajack",$showID:Boolean!){
  user(login:$login) {
    login
    id @include(if: $showID)
  }
}
// 参数
{
  "login":"kikajack",
  "showID": true
}
// 返回
{
  "data": {
    "user": {
      "login": "kikajack",
      "id": "MDQ6VXNlcjE4NTMyODU5"
    }
  }
}
// POST 数据
{"query":"query myQuery($login:String = \"kikajack\",$showID:Boolean!){\n  user(login:$login) {\n    login\n    id @include(if: $showID)\n  }\n}\n","variables":"{\n  \"login\":\"kikajack\",\n  \"showID\": true\n}","operationName":"myQuery"}
```
GraphQL 支持的指令有两个：

- @include(if: Boolean) 仅在参数为 true 时，包含此字段。
- @skip(if: Boolean) 如果参数为 true，跳过此字段。

指令可以附着在字段或者片段包含的字段上，然后以任何服务端期待的方式来增删查询字段。
服务端实现也可以定义新的指令来添加新的特性。
#8. Inline Fragments（内联片段）
```
// 请求，通过内联片段取不同类型数据的不同字段
query{
  search(type:ISSUE,query:"Starkast",first:30) {
    edges {
      node {
        ... on Issue {
          title
        }
        ... on Repository {
          reposname: name
        }
        ... on User {
          name
        }
        ... on PullRequest {
          number
        }
      }
    }
  }
}
// 返回
{
  "data": {
    "search": {
      "edges": [
        {
          "node": {
            "title": "Search doesn't match part of words"
          }
        },
        ...// 这里删除了部分 node
        {
          "node": {
            "title": "Be able to supply port to Dnsruby::DNS"
          }
        },
        {
          "node": {
            "number": 63
          }
        },
        {
          "node": {
            "number": 62
          }
        },
        {
          "node": {
            "title": "Remove oauth2 from Gemfile"
          }
        },
        {
          "node": {
            "title": "Replace github-markdown with github-markup"
          }
        },
        ...// 这里删除了部分 node
      ]
    }
  }
}
```
如果查询的**结果类型不能提前确定**，且返回的字段是接口（Interface）或者联合（Union）类型，就需要使用内联片段来根据不同的返回类型取出下层具体类型的数据。
在需要判断类型的字段上，通过 `... on 类型名 { 字段名 }` 即可自动对服务端查询出的数据类型进行判断。
#9. Mutations（变更）
```
// 请求，其中变量类型（AddReactionInput 的位置）必须是标量、枚举型或输入对象类型（由 Schema 指定），变量 $myVar 必须实现该类型，且变量中的属性只能从该类型中选
mutation($myVar:AddReactionInput!) {
  addReaction(input:$myVar) {
    reaction {
      content
    }
    subject {
      id
    }
  }
}
// 查询变量 query variables
{
  "myVar": {
    "subjectId": "MDU6SXNzdWUyMTc5NTQ0OTc=",
    "content": "HOORAY"
  }
}
// 返回
{
  "data": {
    "addReaction": {
      "reaction": {
        "content": "HOORAY"
      },
      "subject": {
        "id": "MDU6SXNzdWUyMTc5NTQ0OTc="
      }
    }
  }
}
```
查询字段时，是并行执行，而变更字段时，是顺序执行，一个接着一个。一个请求中发送了多个变更时，前面的变更会首先执行。