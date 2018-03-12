[参考中文文档](http://graphql.cn/learn/schema/)
[参考博客资料](http://www.59m59s.com/blog/li-jie-graphqlgai-nian-schema/)
#1. 概念和示例
GraphQL 服务端的库和应用可以用各种语言实现，所以这里的示例脱离语言。
GraphQL 服务端的应用代码的基本实现流程（需要提前安装好 GraphQL 库，[各种语言的库参考这里](http://graphql.cn/code/)）：

1. 定义用户自定义类型。类型的每个字段都必须是已定义的，且最终都是 GraphQL 中定义的类型。
2. 定义根类型。每种根类型中包含了准备暴露给服务调用方的用户自定义类型。
3. 定义 Schema。每一个 Schema 中允许出现三种根类型：query，mutation，subscription，其中至少要有 query。

每次调用 GraphQL 服务，需要明确指定调用 Schema 中的哪个根类型（默认是 query），然后指定这个根类型下的哪几个字段（每个字段对应一个用户自定义类型），然后指定这些字段中的那些子字段的哪几个。一直到所有的字段都没有子字段为止。
Schema 明确了服务端有哪些字段（用户自定义类型）可以用，每个字段的类型和子字段。每次查询时，服务器就会根据 Schema 验证并执行查询。

定义用户自定义类型：
```
interface Human {
 id: ID!
 name: String
 age: Int
}
scalar Url  
type User implements Human { 
 id: ID!
 name: String
 age: Int
 is_active: Boolean
 friends: [User]!
 website: Url
 lastStoryPosted: Story
}
type Story {
  id: ID!
  author: user
  comments(after: ID, limit: Int = 5): [Comment]
}
```
定义根类型（名字随意，和 Schema 中一致即可）：
```
type Query {
  user(id: ID): User
  viewer: User
  stories(after: ID, limit: Int = 10): [Story]!
}
type Mutation {
  ...// 暂时省略
}
type Subscription{
  ...// 暂时省略
}
```
定义 Schema（模型），Schema 文件里有4个关键字：

- schema--表示这是一个GraphQL schema定义； 
- query--定义查询操作，必须有。
- mutation--定义变更操作，可以省略。
- subscription--定义订阅操作，可以省略。
```
schema {  
 query: Query
 mutation: Mutation
 subscription: Subscription
}
```
#2. Type 类型
##2.1 Scalar（标量类型）
Scalar 是解析到单个标量对象的类型，无法再进行次级选择（不含子字段，不可拆分）。
Scalar 是 GraphQL 查询的**叶子节点**。下面例子中的 id 字段，就是标量类型。
```
type Story {
  id: ID!
  author: user
  comments(after: ID, limit: Int = 5): [Comment]
}
```
### 默认标量类型
GraphQL 自带一组默认标量类型：

- Int：有符号 32 位整数。
- Float：有符号双精度浮点值。
- String：UTF‐8 字符序列。
- Boolean：布尔值。
- ID：唯一标识符，用以获取对象或者缓存中的键。ID 类型使用和 String 一样的方式序列化，不易读，例如 `"id": "MDU6SXNzdWUyMTc5NTQ0OTc="`。
### 自定义标量类型
大部分的 GraphQL 服务实现中，都可以**自定义标量类型**。例如，定义一个 Url 类型：`scalar Url`，然后在实现中定义如何将其序列化、反序列化和验证。例如，可以指定 Date 类型应该总是被序列化成整型时间戳，而客户端应该知道任何获取的 date 字段都是这个格式。
##2.2 Object（对象类型）
Schema 中的**最基本的组件**是对象，Schema 中的大多数类型都是对象类型。**对象由子字段组成**。
下面例子中的 Story，就是对象类型。
```
type Story {
  id: ID!
  author: user
  comments(after: ID, limit: Int = 5): [Comment]
}
```
详细解释：

- id、author和 comments 是 Story 对象的字段。这意味着在操作 Story 对象的操作中，最多只能出现这三个字段。
- ID! 表示这个字段是**非空的**，GraphQL 服务会保证只要你查询这个字段，就一定会收到值。
- [Comment] 表示这个字段是一个由 Comment 组成的列表。
##2.3 Interfaces（接口）
接口是一个**抽象类型**，它包含了某些字段，对象必须包含这些字段，才能实现这个接口。
```
```
interface Human {
 id: ID!
 name: String
 age: Int
}
type User implements Human { 
 id: ID!
 name: String
 age: Int
 is_active: Boolean
 friends: [User]!
 website: Url
 lastStoryPosted: Story
}
```
上面例子中，User 对象含有 Human 接口的所有字段，所以正确实现了 Human 接口。
同一个接口，可以实现出不同的对象，比如 `User` 和 `Viewer` 都可以实现 `Human` 接口。
##2.4 Union（联合类型）
联合类型必须由对象类型组成，而不能是接口或者其他联合类型。
服务端返回联合类型的字段时，实际上返回的是联合类型中的某一种对象类型，客户端需要用内联片段处理。
和接口十分相似，但是它并不指定类型之间的任何共同字段。
### 语法
定义联合类型： `union SearchResult = Human | Droid | Starship`
在我们的schema中，任何返回一个 SearchResult 类型的地方，都可能得到一个 Human、Droid 或者 Starship。注意，联合类型的成员需要是具体对象类型；你不能使用接口或者其他联合类型来创造一个联合类型。
### 客户端需要用内联片段处理服务端返回的联合类型字段
如果查询一个返回联合类型的字段，得使用内联片段才能查询任意字段。
```
// 请求，通过内联片段取不同对象的字段
query{
  search(type:ISSUE,query:"Starkast",first:30) {
    edges {
      node {
        ... on Issue {
          title
        }
        ... on PullRequest {
          number
        }
      }
    }
  }
}
```
##2.5 Input（输入类型）
在变更（mutation）中会经常用到 Input 输入类型，因为有时候需要传递一整个对象作为新建对象。输入对象定义的关键字是 input。
输入对象类型上的字段本身也可以指代输入对象类型。输入对象类型的字段当然也不能拥有参数。

服务端定义：
```
input AddReactionInput {
  subjectId: ID!
  content: String
}
type addReaction {
  ...
}
type Mutation {
  addReaction(params: AddReactionInput): AddReactionPayload
}
```
客户端请求：
```
// 请求，字段 addReaction 带了整个 AddReactionInput 类型的对象作为参数
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
```
##2.6 Enumeration（枚举类型）
也叫枚举（enum）。枚举类型是一种特殊的标量，为可选值限定了一个集合。
```
enum Human {
  USER
  VIEWER
}
```
上面的例子中，无论在哪里使用了 Human，都可以肯定它返回的是 USER 和 VIEWER 之一。

注意，各种语言实现的 GraphQL 服务，枚举处理方式可能有所差异。
##2.7 List and Non-Null（列表和非空）
List 列表对应数组，由方括号 `[]` 组成，中间放类型或参数。
非空 可以在类型后面加上感叹号 `!` 表示。例如：`id: ID!`。不能为空就是不能为 null。
非空和列表修饰符可以组合使用。
```
myField: [String!]
// 表示数组本身可以为空，但是其不能有任何空值（null）成员。用 JSON 举例如下：

myField: null // 有效
myField: [] // 有效
myField: ['a', 'b'] // 有效
myField: ['a', null, 'b'] // 错误
```
```
myField: [String]!
// 表示数组本身不能为空，但是可以包含空值成员：

myField: null // 错误
myField: [] // 有效
myField: ['a', 'b'] // 有效
myField: ['a', null, 'b'] // 有效
```
Non-Null 非空用法：

- String ：允许 null 或字符串类型。 
- String! ：不允许 null 或字符串类型。 
- [String] ：允许 null 或字符串列表类型。
- [string]! ：列表内的值允许为 null，但列表本身不允许为 null 的字符串列表类型。 例如 `[]` 和 `[null]` 合法，但 `null` 非法。
- [String!]! ：列表内不允许有 null 值，且列表本身也不允许为 null 的字符串列表类型。

#3. Field（字段），Argument（参数）
**对象由子字段组成**。对象的每一个字段都可能有参数。所有参数都必须命名。
```
type Story {
  id: ID!
  author: user
  comments(after: ID, limit: Int = 5): [Comment]
}
```
上面的例子中，Story 对象有三个字段：id，author，comments。其中 comments 字段有两个参数：after，limit。limit 参数指定了默认值，所以用户可以不传，但是 after 参数必须传。
可选的参数必须定义默认值。
#4. Query（查询）和 Mutation（变更）
Schema 中的这两个特殊类型和常规对象类型唯一的区别：定义了每一个 GraphQL 查询的入口。
所有的查询，都要定义好后放在 query 对象中。所有的变更，都要定义好后放在 mutation 对象中。
Query 是必不可少的，Mutation 可以不定义。
```
schema {  
 query: Query
 mutation: Mutation
 subscription: Subscription
}
```