[REST 和 GraphQL 的对比](https://www.tuicool.com/articles/N3uqyyB)
[APIJSON 和 GraphQL 的对比](https://ruby-china.org/topics/31997)

#1. REST
核心理念是资源。**服务端定义资源形式**。资源的类型和获取资源的方法是紧密相关的。
REST 是多入口的，每个资源对应一个 URL，例如：`http://api.test.com/books/`，`http://api.test.com/users/`。
每个资源由后台定义好后，通过指定的一个 URL 访问（每个 URL 访问到不同的控制器）。通过向指定 URL发送 GET 请求来获取资源，或发送 POST 请求来创建或操作资源。
大部分 API 会返回 JSON 响应。例如：
```
// 请求
GET /books/1
GET /users/250

// 响应
{
  "title": "Black Hole Blues",
  "author": {
    "firstName": "Janna",
    "lastName": "Levin"
  }
  // ... more fields here
}
```
#2. GraphQL
GraphQL 中需要定义所有资源的类型，但是不会指定对应的方法。**只需要改变查询内容，前端就能定制服务器返回的响应内容。**
GraphQL 是单入口的，所有的请求通过同一个 URL 进入服务器，例如：`http://api.test.com/graphql`。
在服务端，必须定义 Schema（模式）作为 GraphQL 请求的入口，用户的 GraphQL 请求在服务端解析后，会对应到具体的 Schema。
GraphQL 请求是这样的：`query getHightScore { score }` 可以获取 getHightScore 的 score 值。
也可以加上查询条件，例如：`query getHightScore(limit: 10) { score }`。

客户端示例：
```
// 查询，不加任何前缀时，默认是 query 查询。
{
  user(id: 3500401) {
    name,
    profilePicture(size: 50)  {
      uri,
      width,
      height
    }
  }
}

// 响应，返回 JSON 格式的数据
{
  "user" : {
    "name": "Jing Chen",
    "profilePicture": {
      "uri": "http: //someurl.cdn/pic.jpg",
      "width": 50,
      "height": 50
    }
  }
}
```
服务端示例（协议实现起来挺复杂，最好直接安装现成的库，[各种语言版本的服务端库在这里](http://graphql.cn/code/)。）：
```
// 定义资源
type Book {
  id: ID
  title: String
  published: Date
  price: String
  author: Author
}
type Author {
  id: ID
  firstName: String
  lastName: String
  books: [Book]
}
// 定义查询
type Query {
  book(id: ID!): Book
  author(id: ID!): Author
}
```
使用 GraphQL：
```
GET /graphql?query={ book(id: "1") { title, author { firstName } } }

{
  "title": "Black Hole Blues",
  "author": {
    "firstName": "Janna",
  }
}
```
#3. APIJSON
[APIJSON 的 GitHub 地址](https://github.com/TommyLemon/APIJSON/blob/master/Document.md)
暂时还没研究，后面再说。