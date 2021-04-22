---
author: "李昌"
title: "Graphql基本概念"
date: "2021-04-22"
tags: ["graphql", "数据库描述", "language"]
categories: ["Tem"]
ShowToc: true
TocOpen: true
---

## 1. 什么是Graphql
GraphQL 既是一种用于 API 的查询语言也是一个满足你数据查询的runtime。 GraphQL对你的API中的数据提供了一套易于理解的完整描述，使得客户端能够准确地获得它需要的数据，而且没有任何冗余，也让API更容易地随着时间推移而演进，还能用于构建强大的开发者工具。   
一个 GraphQL 服务是通过定义类型和类型上的字段来创建的，然后给每个类型上的每个字段提供解析函数。   
**简单的说，GraphQL为我们定义数据库提供了更为便捷的方式，你不需要写任何SQL语句，即可完成数据库的创建及迁移等工作。**

## 2. 概览
例如，一个 GraphQL 服务告诉我们当前登录用户是 me，这个用户的名称可能像这样：
```graphql
type Query {
  me: User
}

type User {
  id: ID
  name: String
}
```

一并的还有每个类型上字段的解析函数：  
```graphql
function Query_me(request) {
  return request.auth.user;
}

function User_name(user) {
  return user.getName();
}
```
一旦一个 GraphQL 服务运行起来（通常在 web 服务的一个 URL 上），它就能接收 GraphQL 查询，并验证和执行。接收到的查询首先会被检查确保它只引用了已定义的类型和字段，然后运行指定的解析函数来生成结果。   

例如这个查询：
```
{
  me {
    name
  }
}
```
会产生这样的JSON结果：
```json
{
  "me": {
    "name": "Luke Skywalker"
  }
}
```

## 3. Schema 和类型
GraphQL 服务可以用任何语言编写，但并不依赖于任何特定语言的句法句式（譬如 JavaScript）来与 GraphQL schema 沟通，Graphql定义了自己的简单语言，称之为 “GraphQL schema language”。

### 3.1 对象类型和字段
一个 GraphQL schema 中的最基本的组件是对象类型，它就表示你可以从服务上获取到什么类型的对象，以及这个对象有什么字段。使用 GraphQL schema language，我们可以这样表示它：
```graphql
type Character {
  name: String!
  appearsIn: [Episode!]!
}
```
> 说明

- Character 是一个 GraphQL 对象类型，表示其是一个拥有一些字段的类型。你的 schema 中的大多数类型都会是对象类型。
- name 和 appearsIn 是 Character 类型上的字段。这意味着在一个操作 Character 类型的 GraphQL 查询中的任何部分，都只能出现 name 和 appearsIn 字段。
- String 是内置的标量类型之一 —— 标量类型是解析到单个标量对象的类型，无法在查询中对它进行次级选择。后面我们将细述标量类型。
- String! 表示这个字段是非空的，GraphQL 服务保证当你查询这个字段后总会给你返回一个值。在类型语言里面，我们用一个感叹号来表示这个特性。
- [Episode!]! 表示一个 Episode 数组。因为它也是非空的，所以当你查询 appearsIn 字段的时候，你也总能得到一个数组（零个或者多个元素）。且由于 Episode! 也是非空的，你总是可以预期到数组中的每个项目都是一个 Episode 对象

### 3.1.5 查询和变更类型（The Query and Mutation Types）
你的 schema 中大部分的类型都是普通对象类型，但是一个 schema 内有两个特殊类型：
```graphql
schema {
  query: Query
  mutation: Mutation
}
```
每一个 GraphQL 服务都有一个 query 类型，可能有一个 mutation 类型。这两个类型和常规对象类型无差，但是它们之所以特殊，是因为它们定义了每一个 GraphQL 查询的入口。因此如果你看到一个像这样的查询：
```graphql
# Request
query {
  hero {
    name
  }
  droid(id: "2000") {
    name
  }
}

# Response
{
  "data": {
    "hero": {
      "name": "R2-D2"
    },
    "droid": {
      "name": "C-3PO"
    }
  }
}
```
那表示这个 GraphQL 服务需要一个 Query 类型，且其上有 hero 和 droid 字段：
```graphql
type Query {
  hero(episode: Episode): Character
  droid(id: ID!): Droid
}
```
Mutation也是类似的工作方式 —— 你在 Mutation 类型上定义一些字段，然后这些字段将作为 mutation 根字段使用，接着你就能在你的查询中调用.   
有必要记住的是，除了作为 schema 的入口，Query 和 Mutation 类型与其它 GraphQL 对象类型别无二致，它们的字段也是一样的工作方式。

### 3.2 参数
GraphQL 对象类型上的每一个字段都可能有零个或者多个参数，例如下面的 length 字段:
```graphql
type Starship {
  id: ID!
  name: String!
  length(unit: LengthUnit = METER): Float
}
```
所有参数都是具名的。在本例中，length字段定义了一个参数：unit。参数可能是必选或者可选的，当一个参数是可选的，我们可以定义一个默认值 —— 如果 unit 参数没有传递，那么它将会被默认设置为 METER。

### 3.3 标量类型（Scalar Types）
一个对象类型有自己的名字和字段，而某些时候，这些字段必然会解析到具体数据。这就是标量类型的来源：它们表示对应 GraphQL 查询的叶子节点。  
下列查询中，name 和 appearsIn 字段将解析到标量类型：
```
{
  hero {
    name
    appearsIn
  }
}
```

```json
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ]
    }
  }
}
```

标量类型没有任何次级字段，它们是一次查询的叶子节点。
Graphql中预定义的标量类型有：
- Int：有符号 32 位整数。
- Float：有符号双精度浮点值。
- String：UTF‐8 字符序列。
- Boolean：true 或者 false。
- ID：ID 标量类型表示一个唯一标识符，通常用以重新获取对象或者作为缓存中的键。ID 类型使用和 String 一样的方式序列化；然而将其定义为 ID 意味着并不需要人类可读型。

还可以自定义标量类型：
```graphql
scalar Date
```
在自己的实现中定义如何将其序列化、反序列化和验证。例如，你可以指定 Date 类型应该总是被序列化成整型时间戳，而客户端应该知道去要求任何 date 字段都是这个格式。

### 3.4 枚举类型（Enumseration Types）
枚举类型是一种特殊的标量，它限制在一个特殊的可选值集合内。这让你能够：
1. 验证这个类型的任何参数是可选值的的某一个
2. 与类型系统沟通，一个字段总是一个有限值集合的其中一个值。

下面是一个用 GraphQL schema 语言表示的 enum 定义：
```graphql
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}
```
这表示无论我们在 schema 的哪处使用了 Episode，都可以肯定它返回的是 NEWHOPE、EMPIRE 和 JEDI 之一。

### 3.5 列表和非空
对象类型、标量以及枚举是 GraphQL 中你唯一可以定义的类型种类。但是当你在 schema 的其他部分使用这些类型时，或者在你的查询变量声明处使用时，你可以给它们应用额外的类型修饰符来影响这些值的验证。我们先来看一个例子：
```graphql
type Character {
  name: String!
  appearsIn: [Episode]!
}
```
String类型后的感叹号！表示此类型非空。服务器在返回这个字段时，总是会返回一个非空值，如果结果得到了一个空值，那么事实上将会触发一个 GraphQL 执行错误，以让客户端知道发生了错误。  

在 GraphQL schema 语言中，我们通过将类型包在方括号（[ 和 ]）中的方式来标记列表。列表对于参数也是一样的运作方式，验证的步骤会要求对应值为数组。   
非空和列表修饰符可以组合使用。
```graphql
myField: [String!] # 这表示数组本身可以为空,但是其不能有任何空值成员.
# myField: null // 有效
# myField: [] // 有效
# myField: ['a', 'b'] // 有效
# myField: ['a', null, 'b'] // 错误

myField: [String]! # 数组本身不能为空，但是其可以包含空值成员
# myField: null // 错误
# myField: [] // 有效
# myField: ['a', 'b'] // 有效
# myField: ['a', null, 'b'] // 有效
```
你可以根据需求嵌套任意层非空和列表修饰符。

### 3.6 接口（interface）
跟许多类型系统一样，GraphQL 支持接口。一个接口是一个抽象类型，它包含某些字段，而对象类型必须包含这些字段，才能算实现了这个接口。   
例如，你可以用一个 Character 接口用以表示《星球大战》三部曲中的任何角色：
```graphql
interface Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
}
```
这意味着任何实现 Character 的类型都要具有这些字段，并有对应参数和返回类型。  
例如，这里有一些可能实现了 Character 的类型：
```graphql
type Human implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  starships: [Starship]
  totalCredits: Int
}

type Droid implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  primaryFunction: String
}
```
可见这两个类型都具备 Character 接口的所有字段，但也引入了其他的字段 totalCredits、starships 和 primaryFunction，这都属于特定的类型的角色。

### 3.7 联合类型
联合类型和接口十分相似，但是它并不指定类型之间的任何共同字段。
```graphql
union SearchResult = Human | Droid | Starship
```
在我们的schema中，任何返回一个 SearchResult 类型的地方，都可能得到一个 Human、Droid 或者 Starship。注意，联合类型的成员需要是具体对象类型；你不能使用接口或者其他联合类型来创造一个联合类型。

### 3.7 输入类型
目前为止，我们只讨论过将例如枚举和字符串等标量值作为参数传递给字段，但是你也能很容易地传递复杂对象。这在变更（mutation）中特别有用，因为有时候你需要传递一整个对象作为新建对象。在 GraphQL schema language 中，输入对象看上去和常规对象一模一样，除了关键字是 input 而不是 type：
```graphql
input ReviewInput {
  stars: Int!
  commentary: String
}
```