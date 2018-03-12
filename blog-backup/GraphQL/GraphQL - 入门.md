[完整的教程](http://howtographql.cn/basics/0-introduction/)
[完整标准查看这里](http://facebook.github.io/graphql/October2016/)
[完整标准的中文版](http://graphql.cn/learn/)
[淘宝前端团队的 GraphQL 入门教程](http://taobaofed.org/blog/2015/11/26/graphql-basics-server-implementation/)
[GitHub 的基于 GraphQL 的 API 接口示例](https://developer.github.com/v4/explorer/)
[各种语言版本（Java，PHP，Node 等）的 GraphQL 服务端库](http://graphql.cn/code/)

#1. GraphQL 概述
#### GraphQL 是完整的 API 标准
REST 只是一种 API 的风格，而 GraphQL 则是完整的 API 标准。[完整标准查看这里](http://facebook.github.io/graphql/October2016/)
GraphQL 由 Facebook 开发并开源，并且由一个来自全世界的公司和个人组成的社区维护。
GraphQL 仅仅暴露一个端点就能准确的返回客户端需要的数据结构。
REST 最常见的问题之一是过度或者欠缺获取。GraphQL 避免了这个问题，刚刚好。
#### 模式和类型系统
GraphQL 使用强类型系统来定义 API 的功能。API 中所有类型都需要使用 GraphQL 模式定义语言（SDL）在 模式 中定义好。此模式将成为客户端和服务器之间的合同，以确定客户端获取数据的方式。
#2. GraphQL 核心概念
##2.1 基本查询
基本查询可以分两种：带参数的查询，不带参数的查询。
示例：
```
// 不带参数的查询
{
  allPersons {
    name
  }
}

// 返回
{
  "allPersons": [
    { "name": "Johnny" },
    { "name": "Sarah" },
    { "name": "Alice" }
  ]
}
```
```
// 带参数的查询，可以指定查询类型 query （默认就是 query），可以为查询指定名称 myQuery
query myQuery{
  allPersons(last:2) {
    name
  }
}

// 返回
{
  "allPersons": [
    { "name": "Sarah" },
    { "name": "Alice" }
  ]
}
```
查询中的 allPersons 字段称为查询的 `根字段`。 根字段必须要在服务器端定义好对应的模式。根字段之后的所有内容都称为查询的 `有效载荷`。
##2.2 类型系统（Type）和模式（Schema）
###2.2.1 类型系统
类型系统，包含系统支持的类型和指令。指令和类型可以重名。
####1. 类型
GraphQL 中有八种类型：

1. Scalar 标量表示原始值，如字符串或整数。通常情况下，标量字段的可能的响应是可枚举的。Enum在这种情况下提供了一种类型，其中类型指定了有效响应的空间。
2. Object 对象
3. Interface 接口，是抽象类型
4. Union 联合，是抽象类型
5. Enum 枚举
6. Input Object 输入对象
7. List 列表，是包装类型
8. Non-Null 非空，是包装类型

标量和枚举在响应树中形成叶子；中间级别是 Object 定义一组字段的类型，其中每个字段是系统中的另一种类型，允许定义任意类型的层次结构。
Interface 定义了一个字段列表，Object实现该接口的类型保证实现这些字段。每当类型系统声明它将返回一个接口，它将返回一个 Object。
Union 定义了一个可能类型的列表，类似于接口，只要类型系统声明返回 Union，会返回 Union 中一个的类型。
List 返回其他类型的列表，并包装其他类型。
Non-Null 类型包装另一种类型，并表示结果将永远不会为空。
非包装类型被称为“基本类型”。包装类型由基本类型组成。
Input Object类型允许模式在这些查询中准确定义来自客户端的数据。
####2. 指令
GraphQL 支持的指令有两个：

@include(if: Boolean)：仅在参数为 true 时，包含此字段。
@skip(if: Boolean)：如果参数为 true，跳过此字段。
具体可以[参考这里](http://blog.csdn.net/kikajack/article/details/79085377#t7)
###2.2.2 模式
**服务器端定义好了模式，客户端才可以发起查询或变更。**

模式（Schema）是 GraphQL 请求的入口，描述了 GraphQL 服务器的功能，并用于确定查询是否有效。Schema 还描述查询变量的输入类型，以确定在运行时提供的值是否有效。用户的 GraphQL 请求会对应到具体的 Schema。编写模式的语法称为模式定义语言 (SDL，Schema Definition Language)。
模式由每种操作的根类型表示：查询和变更。根类型决定了每种操作开始的类型系统中的位置。

模式是根据类型系统所支持的类型和指令来定义的，就是一些**基本类型的集合**。
模式分为两种：

- 系统模式：划分类型。包括 Query 查询（相当于数据库操作的查），Mutation 改变（相当于数据库操作的增删改），Subscription 订阅。系统模式是客户端发送的请求的 入口点，只有这三个模式内的用户自定义模式才可以通过 API 访问到。
- 用户自定义模式：执行具体的操作。所有的用户自定义模式，需要添加到某个类型的系统模式才能生效。

下面的例子中，有四个用户自定义的模式可以使用：查询（Query）可以用 viewer 或 allPersons，变更（Mutation）可以用 createPerson，订阅（Subscription）可以用 newPerson。
```
type Query {
  viewer: Person!
  allPersons(last: Int): [Person!]!
}
type Mutation {
  createPerson(name: String!, age: Int!): Person!
}
type Subscription {
  newPerson: Person!
}
type Person {
  name: String!
  age: Int!
  posts: [Post!]!
}
type Post {
  title: String!
  author: Person!
}
```
Person 类型由两个 字段 组成，类型为 String 的 name 和类型为 Int 的 age。! 表示该 字段 为 必选。