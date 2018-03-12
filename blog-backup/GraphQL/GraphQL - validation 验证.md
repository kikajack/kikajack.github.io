[官方文档](http://facebook.github.io/graphql/October2016/#sec-Validation)

#1. 概述
GraphQL 不只验证请求在**语法**上是否正确，还验证在**当前服务器的 GraphQL 模式上下文**中是否正确。预判查询是否有效，从而可以让服务器和客户端在无效查询创建时就有效地通知开发者，而不用等运行时才知道。
###类型系统演变
GraphQL 类型系统模式随着时间的推移而增加新的类型和新的字段，以前有效的请求有可能在以后变得无效。任何可能导致先前有效的请求变为无效的更改都被视为重大更改。

下面的类型系统是后面所有示例所参考的：
```
enum DogCommand { SIT, DOWN, HEEL }

type Dog implements Pet {
  name: String!
  nickname: String
  barkVolume: Int
  doesKnowCommand(dogCommand: DogCommand!): Boolean!
  isHousetrained(atOtherHomes: Boolean): Boolean!
  owner: Human
}

interface Sentient {
  name: String!
}

interface Pet {
  name: String!
}

type Alien implements Sentient {
  name: String!
  homePlanet: String
}

type Human implements Sentient {
  name: String!
}

enum CatCommand { JUMP }

type Cat implements Pet {
  name: String!
  nickname: String
  doesKnowCommand(catCommand: CatCommand!): Boolean!
  meowVolume: Int
}

union CatOrDog = Cat | Dog
union DogOrHuman = Dog | Human
union HumanOrAlien = Human | Alien

type QueryRoot {
  dog: Dog
}
```
#2. 详情
##2.1 operations 操作规范
###2.1.1 操作不可重名，即使每个操作的类型不同也不行：
```
// 无效操作
query dogOperation {
  dog {
    name
  }
}
mutation dogOperation {
  mutateDog {
    id
  }
}
```
###2.1.2 如果用匿名操作，则只能有一个操作，不能有其他操作，即使是已经命名的操作也不行：
```
// 无效操作
query dogOperation {
  dog {
    name
  }
}
{
  mutateDog {
    id
  }
}
```
##2.2 Field 字段
###2.2.1 对象，接口和联合类型的字段存在时才可以选择
由于联合不定义字段，字段不能直接从联合类型选择集中选择，除了元字段__typename。只能通过片段间接查询来自联合类型选择集的字段。
```
// 有效的:
fragment inDirectFieldSelectionOnUnion on CatOrDog {
  __typename
  ... on Pet {
    name
  }
  ... on Dog {
    barkVolume
  }
}
// 无效的：
fragment directFieldSelectionOnUnion on CatOrDog {
  name
  barkVolume
}
```
###2.2.2 字段选择合并
如果在执行期间**客户端发送具有相同名称的多个字段**，则要执行的字段和参数以及结果值应该是明确的。因此，对于同一对象可能遇到的任何两个字段选择只有在等同的情况下才是有效的。
```
// 正确合并：
fragment mergeIdenticalFields on Dog {
  name
  name
}
fragment mergeIdenticalFieldsWithIdenticalArgs on Dog {
  doesKnowCommand(dogCommand: SIT)
  doesKnowCommand(dogCommand: SIT)
}
// 无法合并：
fragment conflictingBecauseAlias on Dog {
  name: nickname
  name
}
```
###2.2.3 叶子字段
叶子字段作为标量，其上不允许进行字段的选择。相反，GraphQL 查询的**叶子字段必须是标量**。
```
// 无效的：
fragment scalarSelectionsNotAllowedOnBoolean on Dog {
  barkVolume {
    sinceWhen
  }
}
```
##2.3 Arguments 参数
参数可以用在字段和指令中。
###2.3.1 服务端定义了字段或指令的可能参数后，客户端才可以用
```
// 有效的:
fragment argOnOptional on Dog {
  isHousetrained(atOtherHomes: true) @include(if: true)
}
// 无效的，因为字段 doesKnowCommand 没有定义 command 类型的参数。
fragment invalidArgName on Dog {
  doesKnowCommand(command: CLEAN_UP_HOUSE)
}
// 也是无效的，因为 @include 没有定义 unless。
fragment invalidArgName on Dog {
  isHousetrained(atOtherHomes: true) @include(unless: false)
}
```
###2.3.2 参数值类型的正确性
- 参数值的类型满足自动转换规则即合法。比如 String 参数可以接受 Int 类型。Int 类型可以被强制转换为 Float 类型，但是 Float 类型不可以转换为 Int 类型。
```
// 有效的：
fragment goodBooleanArg on Arguments {
  booleanArgField(booleanArg: true)
}
fragment coercedIntIntoFloatArg on Arguments {
  floatArgField(floatArg: 1)
}
// 无效的，Float 类型无法转换为 Int 类型：
fragment stringIntoInt on Arguments {
  intArgField(intArg: "3")
}
```
- 非空参数必传，不传或传 null 都是无效的。
```
// 无效的：
fragment missingRequiredArg on Arguments {
  notNullBooleanArgField(nonNullBooleanArg: null)
}
```
##2.4 Fragments 片段
片段不可重名，且定义后就必须使用，否则查询无效。用 `...片段名` 来使用片段，比如 `...dogFragment`。
###2.4.1 片段声明
片段声明不可重名，且必须基于 Schema 中已有的类型 `on SomeType`。
```
// 有效的：
fragment inlineFragment on Dog {
  ... on Dog {
    name
  }
}
fragment inlineFragment2 on Dog {
  ... @include(if: true) {
    name
  }
}
// 无效的：
fragment notOnExistingType on NotInSchema {
  name
}
fragment inlineNotExistingType on Dog {
  ... on NotInSchema {
    name
  }
}
```
###2.4.2 复合类型的片段
不管是内联片段还是普通片段，都必须必须基于以下类型： UNION，INTERFACE 或 OBJECT。在标量上或叶子上是无效的。
```
// 无效的：
fragment fragOnScalar on Int {
  something
}

fragment inlineFragOnScalar on Dog {
  ... on Boolean {
    somethingElse
  }
}
```

###2.4.3 片段不可构成循环
```
// 构成循环的片段，无效：
{
  dog {
    ...nameFragment
  }
}

fragment nameFragment on Dog {
  name
  ...barkVolumeFragment
}

fragment barkVolumeFragment on Dog {
  barkVolume
  ...nameFragment
}
```
###2.4.4 片段可以嵌套
下面的抽象范围指的是接口 Interface 或联合 Union 的作用域，对象范围指的是对象 Object 的作用域。
####1. 对象范围中嵌套对象片段
在对象类型的范围内，唯一有效的对象类型片段就是这个片段自身。
```
// 有效的查询：
query you{
  viewer {
    ...myFragment
  }
}
fragment myFragment on User{
  name
  ... on User { // 这里只能基于 User，因为在 User 对象的作用域中。
    login
  }
}
// 有效的查询的返回数据：
{
  "data": {
    "viewer": {
      "name": null,
      "login": "kikajack"
    }
  }
}
// 无效的：
fragment myFragment on User{
  name
  ... on Repositories{ // 这里只能基于 User，因为在 User 对象的作用域中。
    id
  }
}
```
####2. 对象范围中嵌套抽象片段
在对象类型的范围内，如果对象类型实现接口或者是联合的成员，则可以使用联合或接口传播。
```
// 有效，因为 Dog 实现了 Pet。
fragment petNameFragment on Pet {
  name
}
fragment interfaceWithinObjectFragment on Dog {
  ...petNameFragment
}
// 有效，因为 Dog 是 CatOrDog Union 类型的成员
fragment catOrDogNameFragment on CatOrDog {
  ... on Cat {
    meowVolume
  }
}
fragment unionWithObjectFragment on Dog {
  ...catOrDogNameFragment
}
```
####3. 抽象范围中嵌套对象片段
只有当对象类型是该接口或联合的可能类型之一时，才可以在对象类型片段的上下文中使用联合或接口扩展。
```
//有效的：
fragment petFragment on Pet {
  name
  ... on Dog {
    barkVolume
  }
}
fragment catOrDogFragment on CatOrDog {
  ... on Cat {
    meowVolume
  }
}
```
####4. 抽象范围中嵌套抽象片段
Union 和 Interface 片段可以相互嵌套。只要在作用域和扩展的可能类型的交集中存在至少一个对象类型，就是有效的。
```
// 有效的，因为狗实现接口宠物，是DogOrHuman的成员。
fragment unionWithInterface on Pet {
  ...dogOrHumanFragment
}
fragment dogOrHumanFragment on DogOrHuman {
  ... on Dog {
    barkVolume
  }
}
// 无效的，因为没有实现Pet和Sentient的类型。
fragment nonIntersectingInterfaces on Pet {
  ...sentientFragment
}
fragment sentientFragment on Sentient {
  name
}
```
##2.5 Values 值
Values 具有唯一性，下面的查询将不会通过验证：
```
{
  field(arg: { field: true, field: false })
}
```
##2.6 Directives 指令
###2.6.1 指令在有效位置才可用
下面的查询不会通过验证，因为 QUERY 不认可 @skip 指令，这不是 @skip 的有效的位置：
```
query @skip(if: $foo) {
  field
}
```
###2.6.2 
每个位置不允许重复使用指令。下面的查询不会通过验证：
```
query ($foo: Boolean = true, $bar: Boolean = false) {
  field @skip(if: $foo) @skip(if: $bar)
}
```
##2.7 Variables 变量
###2.7.1 变量默认值
####如果变量不是非空，则允许变量定义默认值。如果变量是非空，则默认值不可访问。
```
// 成功
query houseTrainedQuery($atOtherHomes: Boolean = true) {
  dog {
    isHousetrained(atOtherHomes: $atOtherHomes)
  }
}
// 下面的查询失败验证：
query houseTrainedQuery($atOtherHomes: Boolean! = true) {
  dog {
    isHousetrained(atOtherHomes: $atOtherHomes)
  }
}
```
####默认值必须与变量的类型兼容。类型必须匹配或者必须是强制类型的。
```
// 不匹配的类型会失败：
query houseTrainedQuery($atOtherHomes: Boolean = "true") {
  dog {
    isHousetrained(atOtherHomes: $atOtherHomes)
  }
}
// 如果变量类型在强制转换后满足要求，查询将通过验证。
query intToFloatQuery($floatVar: Float = 1) {
  arguments {
    floatArgField(floatArg: $floatVar)
  }
}
```
###2.7.2 输入类型的变量
变量是输入类型时，只能是标量、枚举、输入对象、列表和这些类型的非空变体。不能是对象、联合和接口。

对于这些示例，添加以下类型：
```
input ComplexInput { name: String, owner: String }
extend type QueryRoot {
  findDog(complex: ComplexInput): Dog
  booleanList(booleanListArg: [Boolean!]): Boolean
}
```

```
// 有效的：
query takesBoolean($atOtherHomes: Boolean) {
  dog {
    isHousetrained(atOtherHomes: $atOtherHomes)
  }
}
query takesComplexInput($complexInput: ComplexInput) {
  findDog(complex: $complexInput) {
    name
  }
}
query TakesListOfBooleanBang($booleans: [Boolean!]) {
  booleanList(booleanListArg: $booleans)
}
// 无效：
query takesCat($cat: Cat) {
  # ...
}
query takesDogBang($dog: Dog!) {
  # ...
}
query takesListOfPet($pets: [Pet]) {
  # ...
}
query takesCatOrDog($catOrDog: CatOrDog) {
  # ...
}
```
###2.7.3 变量要先在操作的顶层定义后才能使用
####变量范围是基于操作的，在操作的上下文中使用的变量必须在在操作的顶层定义。
```
// 有效的：
query variableIsDefined($atOtherHomes: Boolean) {
  dog {
    isHousetrained(atOtherHomes: $atOtherHomes)
  }
}
// 无效的：
query variableIsNotDefined {
  dog {
    isHousetrained(atOtherHomes: $atOtherHomes)
  }
}
```
####碎片 Fragment 可以直接使用操作顶层定义的变量，Fragment 中用的的变量需要在所有操作中定义：
```
// 有效的：
query variableIsDefinedUsedInSingleFragment($atOtherHomes: Boolean) {
  dog {
    ...isHousetrainedFragment
  }
}
fragment isHousetrainedFragment on Dog {
  isHousetrained(atOtherHomes: $atOtherHomes)
}
// 无效的：
query housetrainedQueryOne($atOtherHomes: Boolean) {
  dog {
    ...isHousetrainedFragment
  }
}
query housetrainedQueryTwoNotDefined {
  dog {
    ...isHousetrainedFragment
  }
}
fragment isHousetrainedFragment on Dog {
  isHousetrained(atOtherHomes: $atOtherHomes)
}
```
###2.7.4 变量定义后必须使用
所有由操作定义的变量必须在该操作或在该操作中传递的片段中使用，或者在该操作中传递的片段中使用。未使用的变量会导致验证错误。
```
// 有效的，因为 $atOtherHomes 在 isHousetrainedFragment 中使用，其中包含 variableUsedInFragment。
query variableUsedInFragment($atOtherHomes: Boolean) {
  dog {
    ...isHousetrainedFragment
  }
}
fragment isHousetrainedFragment on Dog {
  isHousetrained(atOtherHomes: $atOtherHomes)
}
// 无效的，这个片段没有引用$ atOtherHomes：
query variableNotUsedWithinFragment($atOtherHomes: Boolean) {
  ...isHousetrainedWithoutVariableFragment
}
fragment isHousetrainedWithoutVariableFragment on Dog {
  isHousetrained
}
```
###2.7.5 变量必须和传入的参数兼容
#### 类型匹配，下面的例子会失败，因为类型不匹配：
```
query intCannotGoIntoBoolean($intArg: Int) {
  arguments {
    booleanArgField(booleanArg: $intArg)
  }
}
```
#### 可空变量不能被传递给一个非空的参数（当提供默认参数时）
```
query booleanArgQuery($booleanArg: Boolean) {
  arguments {
    nonNullBooleanArgField(nonNullBooleanArg: $booleanArg)
  }
}
```
#### 对于列表类型，可以为空的列表不能传递给非空列表，并且可以为 null 的值的列表不能传递给非空值的列表
```
// 有效的：
query nonNullListToList($nonNullBooleanList: [Boolean]!) {
  arguments {
    booleanListArgField(booleanListArg: $nonNullBooleanList)
  }
}
// 无效的，可以清空的列表不能传递给非空列表：
query listToNonNullList($booleanList: [Boolean]) {
  arguments {
    nonNullBooleanListField(nonNullBooleanListArg: $booleanList)
  }
}
```