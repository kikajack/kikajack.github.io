[官方文档](http://facebook.github.io/graphql/October2016/#sec-Introspection)

#1. 概述
通过内省系统（Introspection System，也叫自检系统），我们可以知道服务端支持哪些查询，有哪些字段及字段类型等信息。
`__Schema`, `__Type`, `__TypeKind`, `__Field`, `__InputValue`, `__EnumValue`, `__Directive` 这些用两个下划线开头的类型是内省系统的一部分。
```
// 查询 GitHub 中的 queryType 类型（查询入口）的字段类型、字段名和描述信息
query{
  __schema {
    queryType {
      kind
      name
      description
    }
  }
}
// 返回
{
  "data": {
    "__schema": {
      "queryType": {
        "kind": "OBJECT",
        "name": "Query",
        "description": "The query root of GitHub's GraphQL interface."
      }
    }
  }
}
```
```
// 查询 GitHub 中的 User 类型中的所有的字段名和字段类型
query {
  __type(name:"User") {
    name
    fields {
      name
      type {
        name
      }
    }
  }
}

// 返回
{
  "data": {
    "__type": {
      "name": "User",
      "fields": [
        {
          "name": "avatarUrl",
          "type": {
            "name": null
          }
        },
        // 省略了部分数据
        {
          "name": "watching",
          "type": {
            "name": null
          }
        },
        {
          "name": "websiteUrl",
          "type": {
            "name": "URI"
          }
        }
      ]
    }
  }
}
```
#2. 原则
#### 约定
内省系统所需的类型和字段以两个下划线作为前缀，避免了与用户定义的 GraphQL 类型的冲突。
内省系统中的所有类型都提供了一个 `description` 字段，类型是 String，可以对字段做简单的描述。

#### 弃用
为了支持向后兼容性的管理，GraphQL 字段和枚举值可以指示它们是否被弃用（isDeprecated: Boolean），并说明它为什么被弃用（deprecationReason: String）。

#### 类型名称自省
在查询任何对象，接口或联合时，GraphQL 支持在元字段的查询中的任何点处进行类型名称自省。它返回当前被查询的对象类型的名称。
类型名称自省，就是在查询对象中可以使用查询对象类型的名称，经常用在查询 Interface 或 Union 类型以确定返回哪些可能类型的实际类型。参考下面例子：
```
// 请求，查询时指定 __typename，返回数据会把这个字段填充为本次查询的对象名称。
{
  __type(name: "User") {
    __typename
    name
  }
}
// 响应
{
  "data": {
    "__type": {
      "__typename": "__Type",
      "name": "User"
    }
  }
}
```
#3 服务端定义内省系统
##3.1 入口字段
内省系统的元字段（内省系统的入口）是隐含的，不会出现在查询操作的根类型的字段列表中。元字段有两个：

- __schema: __Schema!
- __type(name: String!): __Type

##3.2 GraphQL 内省系统定义的模式：
```
type __Schema {
  types: [__Type!]!
  queryType: __Type!
  mutationType: __Type
  directives: [__Directive!]!
}
type __Type {
  kind: __TypeKind!
  name: String
  description: String

  # OBJECT and INTERFACE only
  fields(includeDeprecated: Boolean = false): [__Field!]
  # OBJECT only
  interfaces: [__Type!]
  # INTERFACE and UNION only
  possibleTypes: [__Type!]
  # ENUM only
  enumValues(includeDeprecated: Boolean = false): [__EnumValue!]
  # INPUT_OBJECT only
  inputFields: [__InputValue!]
  # NON_NULL and LIST only
  ofType: __Type
}
type __Field {
  name: String!
  description: String
  args: [__InputValue!]!
  type: __Type!
  isDeprecated: Boolean!
  deprecationReason: String
}
type __InputValue {
  name: String!
  description: String
  type: __Type!
  defaultValue: String
}
type __EnumValue {
  name: String!
  description: String
  isDeprecated: Boolean!
  deprecationReason: String
}
enum __TypeKind {
  SCALAR
  OBJECT
  INTERFACE
  UNION
  ENUM
  INPUT_OBJECT
  LIST
  NON_NULL
}
type __Directive {
  name: String!
  description: String
  locations: [__DirectiveLocation!]!
  args: [__InputValue!]!
}
enum __DirectiveLocation {
  QUERY
  MUTATION
  FIELD
  FRAGMENT_DEFINITION
  FRAGMENT_SPREAD
  INLINE_FRAGMENT
}
```
##3.3 `__Type`
`__Type` 是内省系统的核心，代表系统中的标量，接口，对象类型，联合，枚举。可以查询一个具体的对象的详细信息。
`__Type` 也代表类型修饰符，用于修改它引用的类型（ofType: __Type）。这就是我们如何表示列表，不可为空的类型及其组合。
##3.4 `__Field`
`__Field` 类型表示对象或接口类型中的每个字段。
```
type __Field {
  name: String!
  description: String
  args: [__InputValue!]!
  type: __Type!
  isDeprecated: Boolean!
  deprecationReason: String
}
```
name：必须返回一个字符串
description：可能返回一个 String 或 null
args：返回 `__InputValue` 表示该字段接受的参数的列表。
type：必须返回一个 `__Type` 代表该字段返回值的类型。
isDeprecated：如果不再使用此字段，则返回 true，否则返回 false。
deprecationReason：可选地提供了为什么这个字段被弃用的原因。
##3.5 `__InputValue`
`__InputValue` 类型表示字段和指令参数以及inputFields输入对象。
```
type __InputValue {
  name: String!
  description: String
  type: __Type!
  defaultValue: String
}
```
name：必须返回一个 String
description：可能返回一个 String 或 null
type：必须返回一个 `__Type` 表示此输入值所期望的类型。
defaultValue：可能会在运行时未提供值的情况下返回此输入值使用的默认值的字符串编码（使用GraphQL语言）。如果此输入值没有默认值，则返回 null。
##3.6 `__Directive`
`__Directive` 类型表示服务器支持的指令。
```
type __Directive {
  name: String!
  description: String
  locations: [__DirectiveLocation!]!
  args: [__InputValue!]!
}
```
name：必须返回一个 String
description：返回一个 String 或 null
locations：返回一个 `__DirectiveLocation`，表示这个指令可能被放置的有效位置的列表。
args：返回 `__InputValue` 表示此指令接受的参数的列表。
##3.7 `__TypeKind`
GraphQL 支持多种类型，每种类型所支持的字段是不一样的。`__TypeKind`枚举中列出了这些类型。
```
enum __TypeKind {
  SCALAR
  OBJECT
  INTERFACE
  UNION
  ENUM
  INPUT_OBJECT
  LIST
  NON_NULL
}
```
###1. 标量 Scalar
标量类型包括：Int，String，Boolean，Float，ID。标量不能有字段。

GraphQL 类型的设计者应该在任何标量的描述字段中描述数据格式和标量强制规则。

kind：必须返回 `__TypeKind.SCALAR`。
name：必须返回一个字符串。
description：可能返回一个 String 或 null。
所有其他字段必须返回 null。
###2. Object
对象类型代表了由一组字段组成的类型。内省类型（例如 `__Type`，`__Field` 等）都是对象。

kind：必须返回 `__TypeKind.OBJECT`。
name：必须返回一个字符串。
description：可能返回一个 String 或 null。
fields：在这个类型上可以查询的字段集合。
接受 includeDeprecated 默认为 false 的参数。如果为 true，则也会返回已弃用的字段。
interfaces：对象实现的一组接口。
所有其他字段必须返回 null。
###3. Union
联合类型是一种抽象类型，没有声明公共的字段。在possibleTypes 明确列出联合类型的可能类型。类型可以直接作为联合的一部分。

kind：必须返回__TypeKind.UNION。
name：必须返回一个字符串。
description：可能返回一个 String 或 null。
possibleTypes：返回此联合中可以表示的类型的列表。它们必须是对象类型。
所有其他字段必须返回 null。
###4. Interface
接口是一个抽象类型，其中声明了通用字段。任何实现接口的类型都必须实现接口中定义的字段且类型一致。接口的所有实现需要在 possibleTypes 明确列出。

kind：必须返回__TypeKind.INTERFACE。
name：必须返回一个字符串。
description：可能返回一个 String 或 null。
fields：这个接口所需的一组字段。
接受 includeDeprecated 默认为false的参数。如果为 true，则也会返回已弃用的字段。
possibleTypes：返回实现此接口的类型的列表。它们必须是对象类型。
所有其他字段必须返回 null。
###5. Enum
枚举是特殊的标量，只能有一组定义的值。

kind必须返回__TypeKind.ENUM。
name 必须返回一个字符串。
description可能返回一个String或null。
enumValues：的清单EnumValue。必须至少有一个，他们必须有独特的名字。
接受includeDeprecated默认为false的参数。如果为true，则也会返回不推荐使用的枚举值。
所有其他字段必须返回null。
###6. Input Object
输入对象是用作查询输入的复合类型，定义为指定输入值的列表。

例如，输入对象Point可以被定义为：
```
type Point {
  x: Int
  y: Int
}
```
kind：必须返回__TypeKind.INPUT_OBJECT。
name：必须返回一个字符串。
description：可能返回一个String或null。
inputFields：清单InputValue。
所有其他字段必须返回null。
###7. List
列表表示 GraphQL 中值的序列。List类型是一个类型修饰符：它在 ofType 字段中包含另一个类型实例，该实例定义了列表中每个项目的类型。

kind必须返回__TypeKind.LIST。
ofType： 任何类型。
所有其他字段必须返回null。
###8. Non-Null
非 null 类型是一个类型修饰符：它在该ofType字段中包装另一个类型实例。非null类型不允许null作为响应，并指示参数和输入对象字段所需的输入。

kind：必须返回__TypeKind.NON_NULL。
ofType：除非空之外的任何类型。
所有其他字段必须返回null。
#4. 客户端使用内省系统
##4.1 入口字段
内省系统的元字段有两个：

- __schema: __Schema!
- __type(name: String!): __Type
##4.2 查询服务端的入口（查询，变更，订阅）
```
// 查询
query{
  __schema {
    queryType {
      name
    }
    mutationType {
      name
    }
    subscriptionType {
      name
    }
  }
}

// 响应
{
  "data": {
    "__schema": {
      "queryType": {
        "name": "Query"
      },
      "mutationType": {
        "name": "Mutation"
      },
      "subscriptionType": null
    }
  }
}
```
##4.3 查询所有可用的对象
```
// 请求
query{
  __schema {
    types {
      name
      description
      kind
    }
  }
}
// 响应
{
  "data": {
    "__schema": {
      "types": [
        {
          "name": "Boolean",
          "description": "Represents `true` or `false` values.",
          "kind": "SCALAR"
        },
        {
          "name": "String",
          "description": "Represents textual data as UTF-8 character sequences. This type is most often used by GraphQL to represent free-form human-readable text.",
          "kind": "SCALAR"
        },
        {
          "name": "Query",
          "description": "The query root of GitHub's GraphQL interface.",
          "kind": "OBJECT"
        },
        {
          "name": "Node",
          "description": "An object with an ID.",
          "kind": "INTERFACE"
        },
        // 后面的省略了
      ]
    }
  }
}
```
##4.4 查询某个对象的所有字段
###每个字段的名字，类型，描述，是否废弃。
```
// 请求
query{
  __type(name:"Topic") {
    name
    fields {
      name
      type {
        name
        kind
      }
      description
      isDeprecated
    }
  }
}
// 响应
{
  "data": {
    "__type": {
      "name": "Topic",
      "fields": [
        {
          "name": "id",
          "type": {
            "name": null,
            "kind": "NON_NULL"
          },
          "description": null,
          "isDeprecated": false
        },
        {
          "name": "name",
          "type": {
            "name": null,
            "kind": "NON_NULL"
          },
          "description": "The topic's name.",
          "isDeprecated": false
        },
        {
          "name": "relatedTopics",
          "type": {
            "name": null,
            "kind": "NON_NULL"
          },
          "description": "A list of related topics, including aliases of this topic, sorted with the most relevant\nfirst.\n",
          "isDeprecated": false
        }
      ]
    }
  }
}
```
### 对于 LIST 等包装类型，需要用 `ofType` 查看具体支持的类型
```
// 请求
{
  __type(name: "User") {
    name
    fields {
      name
      type {
        name
        kind
        ofType {
          name
          kind
        }
      }
    }
  }
}
// 响应
{
  "data": {
    "__type": {
      "name": "User",
      "fields": [
        {
          "name": "avatarUrl",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": "URI",
              "kind": "SCALAR"
            }
          }
        },
        {
          "name": "bio",
          "type": {
            "name": "String",
            "kind": "SCALAR",
            "ofType": null
          }
        },
        {"name": "bioHTML",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": "HTML",
              "kind": "SCALAR"
            }
          }
        },
        {
          "name": "commitComments",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": "CommitCommentConnection",
              "kind": "OBJECT"
            }
          }
        },
        // 下面省略
      ]
    }
  }
}
```
