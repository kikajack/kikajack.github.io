[官方参考文档](http://facebook.github.io/graphql/October2016/#sec-Response)

GraphQL服务器收到一个请求时，它必须返回一个格式正确的响应。服务器的响应描述了操作的结果或遇到的错误。
#1. Serialization Format 序列格式
##1.1 概述
GraphQL 不需要特定的序列格式。但是，客户端应该使用支持 GraphQL 响应中主要原语的序列化格式。
序列化格式必须支持以下四个类型的表示：

- Map
- List
- String
- Null

其中，String 可以用这些类型替代：

- Boolean
- Int
- Float
- Enum Value
##1.2 JSON 序列化
###首选序列化格式
**GraphQL 中首选 JSON 作为序列化格式。**返回的 JSON 对象与查询所定义的那些字段有相同的顺序。
###概念的映射关系
GraphQL 概念和 JSON 中的概念的映射关系：
|GraphQL Value| JSON Value|
|-|-|
|Map| Object|
|List|  Array
|Null|  null
|String|  String
|Boolean| true or false
|Int| Number
|Float| Number
|Enum Value|  String

###对象属性排序
JSON 对象是键值对的无序集合，但以有序方式表示。换句话说，虽然JSON字符串 `{ "name": "Mark", "age": 30 }` 和 `{ "age": 30, "name": "Mark" }` 有相同的值，但是排序不同。

GraphQL 返回的序列化的 JSON 对象与查询所定义的那些字段有相同的顺序。

例如，如果查询是{ name, age }，以JSON响应的GraphQL服务器应该响应{ "name": "Mark", "age": 30 }并且不应该响应{ "age": 30, "name": "Mark" }。
#2. Response Format 响应格式
对GraphQL操作的响应必须是一个 map。
响应有三个顶级属性：data，errors，extensions。
##2.1 data
响应的第一个属性是 data，用来填充所有响应的数据。
data 响应中的数据是执行操作的结果。对于查询操作，此输出是模式的查询根类型的一个对象；对于变更操作，这个输出是该模式的变更根类型的一个对象。

- 如果在执行之前由于语法错误、缺少信息或验证错误导致操作失败，此条目不得存在。
- 如果在执行期间遇到阻止有效响应的错误，则响应中的data条目应该是null。
```
{
  "data": {
    "viewer": {
      "name": null,
      "id": "MDQ6VXNlcjE4NTMyODU5"
    }
  }
}
```
##2.2 errors
只要遇到错误，响应就必须包含属性 errors。该条目的值在“错误”一节中进行了描述。
errors 响应是一个非空的错误列表 List，其中每个错误都是一个 map。

- 如果操作完成而没有遇到任何错误，则 errors 不得存在。
- 每个错误都必须包含属性 message，message 是用于提示开发人员错误的描述信息。
- 如果错误与请求的 GraphQL 文档中的特定点相关联，则这个错误就应该包含 locations 属性。 locations 属性是错误行列位置信息的列表，其中用 line 和 column 表示错误发生在第几行第几列。

如果 data 响应不存在或响应中的值是 null，则 errors 响应不能为空，必须包含至少一个错误，这个错误应该指出为什么没有数据能够被返回。
如果 data 响应不是 null，errors 响应可以包含执行期间发生的任何错误。
```
{
  "data": null,
  "errors": [
    {
      "message": "Field 'namea' doesn't exist on type 'User'",
      "locations": [
        {
          "line": 3,
          "column": 5
        }
      ]
    },
    {
      "message": "Field 'ida' doesn't exist on type 'User'",
      "locations": [
        {
          "line": 6,
          "column": 5
        }
      ]
    }
  ]
}
```
##2.3 extensions
响应也可能包含属性 extensions。此属性的值是 map。这个属性是为接口定义者保留的，可以扩展协议，对内容没限制。
