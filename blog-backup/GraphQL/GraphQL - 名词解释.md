客户使用 GraphQL 查询语言向 GraphQL 服务发出请求。我们将**这些请求源称为文档**。一个文档可能包含 operation 操作（query 查询和 mutation 变更都是操作）以及 fragment（片段）。
#1. Source Text 源文本
GraphQL 支持的 Unicode 字符序列范围：`/ [\ u0009 \ u000A \ u000D \ u0020- \ uFFFF] /`
大多数 GraphQL 仅在原始的非控制 ASCII 范围内表达，以便与尽可能多的现有工具，语言和序列化格式广泛兼容，并避免文本编辑器和源代码控制中的显示问题。
##1.1 Unicode
“字节顺序标记”（Byte Order Mark，`U + FEFF`）是一个特殊的 Unicode 字符，包含 Unicode 的文件以这个字符开始，程序可能使用该字符来确定文本流是否为 Unicode，文本流的字节顺序以及多个 Unicode 中的哪一个编码解释。
非 ASCII 的 Unicode 字符可以在 GraphQL 的 String 和 Comment 部分中自由出现。
##1.2 WhiteSpace 空白符
Horizontal Tab 水平制表符(U+0009)
Space 空格(U+0020)
只是用于提高可读性，无意义。
##1.3 LineTerminator 行结束符
换行：(U+000A)
回车换行：(U+000D) (U+000A)
只是用于提高可读性，无意义。
##1.4 Comments 注释
GraphQL 源文件只可以包含单行注释，以＃标记开始。# 标记之后，行结束符之前的都是注释，对 GraphQL 查询文档没有意义。
##1.5 逗号
类似于空格和行结束符，逗号（，）在 GraphQL 查询文档中是不重要的，可有可无。
##1.6 GraphQL 中的符号
```
! $ ( ) ... : = @ [ ] { | }
```
GraphQL 是一种数据描述语言，而不是编程语言，因此GraphQL缺乏常用于数学表达式的标点符号。
##1.7 GraphQL 名称区分大小写
GraphQL 查询文档充满命名的东西：操作，字段，参数，指令，片段和变量。所有的名字都必须遵循相同的语法形式。
GraphQL 中的名称尽量只用 ASCII 字符的常用子集，以提高兼容性。
#2. Query Document 查询文档
GraphQL 查询文档描述了由 GraphQL 服务接收的完整文件或请求字符串。一个文档包含多个操作和片段的定义。GraphQL 查询文档只有在包含操作的情况下才能由服务器执行。然而，不包含操作的文档仍然可以被解析和验证，以允许客户跨多个文档表示单个请求。

如果一个文档只包含一个操作，则该操作可以不写操作名称。否则，如果一个GraphQL 查询文档包含多个操作，则每个操作都必须命名。将具有多个操作的查询文档提交给 GraphQL 服务时，还必须提供要执行的操作的名称（提交的查询文档可以有多个操作，但是每次只能指定一个操作）。
#3. Operation 操作
GraphQL 模型有两种类型的操作：

- query：查询，只读数据。
- mutation：变更，先写，然后再读取。

操作由操作名称（查询文档中只有一个操作时操作名称可选）和选择集表示。
如果文档只包含一个查询操作，并且该查询没有定义变量并且不包含任何指令，则该操作可以省略查询关键字和查询名称。
```
{
  viewer {
  name
  }
}
```
#4. Selection Sets 选择集
操作选择它所需要的一组信息，并且只会收到这些信息，而不会再收到任何信息，从而避免了数据的过度读取和低速读取。这组信息，就是选择集。
```
// 此查询的 id，firstName 和 lastName 字段构成选择集。选择集可以包含片段引用。
{
  id
  firstName
  lastName
}
```
#5. Fields 字段
选择集主要由字段组成。一个字段描述了在一个选择集内可用于请求的一条独立信息。字段本身可以包含一个选择集，构成嵌套请求。所有 GraphQL 操作都必须最终返回都是标量值的字段。
操作的顶级选择集中的字段通常代表您的应用程序及其当前查看器可全局访问的一些信息，对应函数。
```
// me 是顶级选择集中的字段。me 不是标量所以需要嵌套找内部的标量字段。birthday 和 friends 同理。
{
  me {
    id
    firstName
    lastName
    birthday {
      month
      day
    }
    friends {
      name
    }
  }
}
```
#6. Arguments 参数
对应 GraphQL 服务端的实现是函数的字段，有的可以接受参数。这些字段参数通常直接映射到 GraphQL 服务器实现的函数中的参数。

查询一个特定的用户（通过id参数请求）和他们的特定的个人资料图片size（服务端的这个字段的实现方法接受 id 参数）：
```
{
  user(id: 4) {
    id
    name
    profilePic(size: 100)
  }
}
```
参数是无序的。这两个查询在语义上是相同的：
```
{
  picture(width: 200, height: 100)
}
{
  picture(height: 100, width: 200)
}
```
#7. Field Alias 字段别名
默认情况下，响应对象中的键将使用查询的字段名称。如果提供了别名，则字段的响应键是其别名。
```
{
  zuck: user(id: 4) {
    aliasid: id
    name
  }
}
// 返回结果，其中的 user 字段和 id 字段被别名替换：
{
  "zuck": {
    "aliasid": 4,
    "name": "Mark Zuckerberg"
  }
}
```
#8. Fragments 片段
片段允许重复使用常见的重复字段选择，减少文档中的重复文本。在查询界面或联合时，内联片段可以直接在选择内使用，以便在类型条件下进行调整。
通过 `fragment 片段名 on 字段类型` 定义片段，通过 `...片段名` 使用已经定义好的片段。
片段可以嵌套。
```
query withFragments {
  user(id: 4) {
    friends(first: 10) {
      ...friendFields
    }
    mutualFriends(first: 10) {
      ...friendFields
    }
  }
}

fragment friendFields on User {
  id
  name
  profilePic(size: 50)
}
```
##8.1 Type Conditions 类型条件
- 片段必须用 `on 字段类型` 指定适用的类型。
- 片段不能在任何输入值（标量，枚举或输入对象）上指定。
- 可以在对象类型，接口和联合中指定片段。
- 片段中的选择仅在对象的具体类型与片段的类型匹配时才会返回值。
```
query FragmentTyping {
  profiles(handles: ["zuck", "cocacola"]) {
    handle
    ...userFragment
    ...pageFragment
  }
}

fragment userFragment on User {
  friends {
    count
  }
}

fragment pageFragment on Page {
  likers {
    count
  }
}
// profiles根字段返回一个列表，每个元素可能是Page或User。当profiles结果中的对象是User，friends将出现而likers不会出现。相反当结果是Page，likers会出现，friends不会出现。
{
  "profiles": [
    {
      "handle": "zuck",
      "friends": { "count" : 1234 }
    },
    {
      "handle": "cocacola",
      "likers": { "count" : 90234512 }
    }
  ]
}
```
##8.2 Inline Fragments 内联片段
片段可以在选择集内定义。这是为了**有条件地包含在运行时才能确定类型的字段**。
###标准片段可以用内联片段替换，实现同样目的。
```
query inlineFragmentTyping {
  profiles(handles: ["zuck", "cocacola"]) {
    handle
    ... on User {
      friends {
        count
      }
    }
    ... on Page {
      likers {
        count
      }
    }
  }
}
```
###内联片段也可用于将指令应用于一组字段。如果省略了TypeCondition，则认为内联片段与封闭上下文的类型相同。
```
query inlineFragmentNoType($expandedInfo: Boolean) {
  user(handle: "zuck") {
    id
    name
    ... @include(if: $expandedInfo) {
      firstName
      lastName
      birthday
    }
  }
}
```
#9. Input Values 输入值
字段和指令的参数接受各种输入值：标量，枚举值，列表或输入对象（scalars, enumeration values, lists, or input objects.）。

- Int
- Float
- String
- Boolean
- Null
- Enum
- List 
- Object 

可以将输入值指定为变量。列表和输入对象也可能包含变量（除非定义为常量）。
#10. Variables 变量
GraphQL 查询可以通过变量进行参数化，最大限度地提高查询的重用性。
**变量必须在操作的顶部定义，并且在整个操作的范围内可用。**
```
query getZuckProfile($devicePicSize: Int) {
  user(id: 4) {
    id
    name
    profilePic(size: $devicePicSize)
  }
}
```
#11. Input Types 输入类型
GraphQL 描述了查询中的变量所期望的数据类型。输入类型可以是其他输入类型的列表，也可以是任何其他输入类型的非空变体。
#12. Directives 指令
指令可以改变 GraphQL 的执行行为，比如有条件地包含或跳过一个字段。
GraphQL 中目前有两个指令：

- @include(if: Boolean) 仅在参数为 true 时，包含此字段。
- @skip(if: Boolean) 如果参数为 true，跳过此字段。

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
```