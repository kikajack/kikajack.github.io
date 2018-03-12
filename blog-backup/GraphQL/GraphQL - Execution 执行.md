#1. 概述
GraphQL 请求（查询，变更或订阅）在验证通过后，GraphQL 服务器会执行这个请求，并返回与请求的结构相对应的结果，通常是 JSON 格式。GraphQL 服务端**处理请求时依赖类型系统**。

请求由几条信息组成：

- 要使用的模式，必须是 GraphQL 服务端有效的模式。
- 包含 GraphQL 操作和片段的文档（对于变更 mutation 和订阅 subscription，操作类型必填）。
- 可选：要执行的文档中的操作的名称。
- 可选：操作定义的任何变量的值。
- 与正在执行的根类型相对应的初始值。从概念上讲，初始值代表了通过 GraphQL 服务可用的数据的“范围”。GraphQL 服务通常对每个请求都使用相同的初始值。

`ExecuteRequest（schema，document，operationName，variableValues，initialValue）`，执行请求的五要素：模式，请求文档，操作名称，变量，初始值。
#2. Executing Requests 执行请求
要想执行请求，GraphQL 服务端必须要有一个解析过的请求文档，如果请求文档定义了多个操作还需要执行的操作的名称 operationName。
##2.1 示例
```
// 这是 GitHub Explorer 模拟时发送的数据。定义了2个查询请求 you 和 my，所以必须指定要执行的操作名称 operationName。
{
"query":"
query you{
  viewer {
    name
  }
}
query my {
  viewer {
    name
  }
}",
"variables":"",
"operationName":"my"
}
```
##2.2 执行请求的两个函数
###2.2.1 GetOperation 获取操作名称
GetOperation（document，operationName）
operationName 为 null 时，如果文档只包含一个操作，返回文档中包含的操作。否则产生一个查询错误。
operationName 不为 null 时，返回文档中名称为 operationName 的操作。如果没有找到这个操作，则产生查询错误。
###2.2.1 CoerceVariableValues 获取变量
CoerceVariableValues（schema，operation，variableValues）
###2.2.2 ExecuteRequest 执行请求
ExecuteRequest（schema，document，operationName，variableValues，initialValue）

如果操作是查询操作：
返回 ExecuteQuery（operation，schema，coercedVariableValues，initialValue）。
否则如果操作是一个变异操作：
返回 ExecuteMutation（operation，schema，coercedVariableValues，initialValue）。
##2.3 Validating Requests 验证请求
[验证规则](http://blog.csdn.net/kikajack/article/details/79100332)
只有通过所有验证规则的请求才能被执行。如果验证错误，则应将其报告在响应的“错误”列表中，并且请求必须失败不能执行。

通常在执行请求之前先在请求的上下文中验证。然而，如果已经验证过完全相同的请求，则 GraphQL 服务可以在不验证的情况下执行请求。一个 GraphQL 服务应该**只执行已经知道没有任何验证错误的请求**。

例如：请求可以在开发过程中进行验证，前提是它稍后不会更改，或者服务可以验证请求一次并记录结果以避免将来再次验证相同的请求。
##2.4 Coercing Variable Values 强制转换变量值
如果操作定义了变量，则这些变量的值需要用变量声明的类型按照转换规则进行强制转换。如果在变量值的强制转换过程中遇到查询错误，则操作失败而不执行。

`CoerceVariableValues（schema, operation, variableValues）`
#3. Executing Operations 执行操作
类型系统必须提供一个**查询根对象类型**（通常是 Query）。如果支持变更或订阅，则还必须提供变更根对象类型（Mutation）或订阅根对象类型（Subscription）。
##3.1 查询
`ExecuteQuery(query, schema, variableValues, initialValue)`
查询操作使用查询根对象类型执行查询的顶级选择集（top level selection set）。执行查询时可能会提供一个初始值。
```
// queryType 必须是模式 Schema 中的根查询类型。用下面的查询操作可以获得 queryType 的名字
{
  __schema {
    queryType {
      name
    }
  }
}
```
##3.2 变更
`ExecuteMutation（突变，模式，variableValues，initialValue）`
变更操作使用变更根对象类型执行变更的顶层选择集。多个变更会**顺序执行**而不是并发执行。要注意可能的相互影响，防止竞争。
```
// mutationType 必须是模式 Schema 中的根查询类型。用下面的查询操作可以获得 mutationType 的名字
{
  __schema {
    mutationType {
      name
    }
  }
}
```
#4. Executing Selection Sets 执行选择集
要执行一个选择集，需要知道正在被评估的对象值和对象类型，以及是否必须顺序执行还是可以并行执行。
选择集合首先要转换成分组的字段集合。然后，分组的字段集合中的每个字段都会生成一个对应响应的条目。
`ExecuteSelectionSet(selectionSet, objectType, objectValue, variableValues)`
##4.1 Normal and Serial Execution 正常和串行执行
###4.1.1 查询操作可以并行
通常，服务器可以按照它选择的顺序（默认并行）执行分组字段中的条目。因为查询操作始终是无副作用和幂等的，所以执行顺序不影响结果。
```
// GraphQL 执行器可以按照它的顺序来解析这四个字段（birthday 必须在 month 前解析，address 必须在 street 前解析）。：
{
  birthday {
    month
  }
  address {
    street
  }
}
```
###4.1.2 变更操作必须串行执行
服务器会按照收到的顺序串行执行变更操作。
##4.2 Field Collection 字段收集
在执行之前，需要调用 `CollectFields()` 将选择集转换为分组字段。分组字段集中的每个条目都是一个共享响应键（response key (alias or field name) ）的字段列表。这确保了通过引用片段包含的相同响应键（response key (alias or field name) ）中的所有字段可以同时执行。

作为一个例子，收集这个选择集的字段将收集该字段的两个实例a和一个字段b：
```
{
  a {
    subfield1
  }
  ...ExampleFragment
}

fragment ExampleFragment on Query {
  a {
    subfield2
  }
  b
}
```
`CollectFields(objectType, selectionSet, variableValues, visitedFragments)` ：产生分组字段集。
执行过程中，`CollectFields()` 方法采用深度优先策略，以确保响应中的字段顺序是稳定和可预测的。

`DoesFragmentTypeApply(objectType, fragmentType)`：判断两个参数的类型是否一致或具有继承关系。

#5. Executing Fields 执行字段
分组字段集中所请求的每个字段，都对应响应数据中的一个条目。执行字段时，首先强制转换任何提供的参数值，然后解析字段值，如果字段值是标量则返回标量值，否则递归继续执行这个分组字段选择集。

`ExecuteField(objectType, objectValue, fieldType, fields, variableValues)` ：执行字段
##5.1 Coercing Field Arguments 强制转换字段参数类型
字段可能包含参数。这些参数是由类型系统定义的，只能是这几种输入类型：Scalar 标量，Enum 枚举，Input Object 输入对象或 List 列表或 Non-Null 非空包装这三种变体。参数可以是常量或变量。
`CoerceArgumentValues(objectType, field, variableValues)`：转换字段参数类型
##5.2 Value Resolution 为字段提供值
最终暴露 GraphQL 接口的内部系统必须为所有字段提供值。

`ResolveFieldValue(objectType, objectValue, fieldName, argumentValues)`：确定 fieldName 字段的解析值。
由于解析器依靠读取底层数据库或联网服务来产生一个值，所以通常是异步的。需要 GraphQL 执行程序来处理异步执行流程。
##5.3 Value Completion 计算值，直到满足类型要求
确定了一个字段的值后，如果确实是期望的返回类型，则结束计算。如果返回类型是另一个对象 Object 类型，则字段计算过程将递归执行。
`CompleteValue(fieldType, fields, result, variableValues)`：计算字段的值。
### 解决抽象类型
当计算完成的一个字段的返回类型是抽象类型（接口或联合）时，抽象类型必须解析为相关的对象类型。

`ResolveAbstractType(abstractType, objectValue)`：调用由类型系统提供的内部方法，以确定通过 objectValue 给定的抽象类型的具体的对象的类型。
###合并选择集
当并行执行多个相同名称的字段时，它们的选择集在完成该值时合并在一起以便继续执行子选择集。
```
// GitHub 查询示例，选择集同名时响应会合并。
// 请求：
{
  viewer {
    name
  }
  viewer {
    id
  }
}
// 返回：
{
  "data": {
    "viewer": {
      "name": null,
      "id": "MDQ6VXNlcjE4NTMyODU5"
    }
  }
}
```
`MergeSelectionSets(fields)`
##5.4 Errors and Non-Nullability 错误和不可空性
- 如果在解析字段时引发错误，则应将其视为该字段返回 null，并且必须将错误添加到响应的错误列表中。
- 如果解析字段的结果为空（或者因为解析字段的函数返回 null 或者发生了错误）且该字段是 Non-Null 类型，则会引发字段错误。必须将错误添加到响应的错误列表中。
- 每个字段只有一个错误应该添加到错误列表中。
- 由于 Non-Null 类型字段不能为 null，因此字段错误将由父字段处理。如果父字段也是 Non-Null 类型，字段错误进一步传播到它的父字段，否则解析为 null。
- 如果请求触发的错误源的所有字段都是 Non-Null 类型，则响应中的 `data` 条目应为空。