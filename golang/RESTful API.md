---
author: "李昌"
title: "RESTfulAPI入门"
date: "2021-02-25"
tags: ["GoLang"]
categories: ["GoLang"]
ShowToc: true
TocOpen: true
---

# RESTful API 入门

## 1. 简介

表现层状态转换（英语：Representational State Transfer，缩写：REST）是Roy Thomas Fielding博士于2000年在他的[博士论文](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)中提出来的一种万维网软件架构风格，目的是便于不同软件/程序在网络（例如互联网）中互相传递信息。表现层状态转换是根基于超文本传输协议（HTTP）之上而确定的一组约束和属性，是一种设计提供万维网络服务的软件构建风格。符合或兼容于这种架构风格（简称为 REST 或 RESTful）的网络服务，允许客户端发出以统一资源标识符访问和操作网络资源的请求，而与预先定义好的无状态操作集一致化。因此表现层状态转换提供了在互联网络的计算系统之间，彼此资源可交互使用的协作性质（interoperability）。相对于其它种类的网络服务，例如SOAP服务，则是以本身所定义的操作集，来访问网络上的资源。

## 2. REST 架构约束

1. **客户端－服务器**      
    从本质上讲，这意味着客户端应用程序和服务器应用程序必须能够独立发展而彼此之间没有任何依赖关系。客户端应该只知道资源URI，仅此而已。今天，这是Web开发中的常规做法，因此您不需要任何花哨。把事情简单化。
    
    > 服务器和客户端也可以独立替换和开发，只要它们之间的接口没有更改即可。

2. **无状态**   
    Roy fielding的灵感来自HTTP，因此它反映了这一约束。使所有客户端-服务器交互都变为无状态。服务器将不存储有关客户端发出的最新HTTP请求的任何内容。它将每个请求视为新请求。没有会议，没有历史。

    如果客户端应用程序需要是最终用户的有状态应用程序，则用户必须登录一次并在此之后执行其他授权操作，则来自客户端的每个请求都应包含服务于该请求的所有必要信息，包括身份验证和授权细节。

    > 请求之间不得在服务器上存储任何客户端上下文。客户端负责管理应用程序的状态。
 
3. **统一的接口**     
    在约束名称本身适用的情况下，您必须为系统内部暴露给API使用者并认真遵循的资源确定API接口。系统中的资源应仅具有一个逻辑URI，并且应提供一种获取相关或附加数据的方式。最好将资源与网页同义。  
     任何单个资源都不应太大，并在其表示中包含所有内容。只要相关，资源应包含指向相对URI的链接（HATEOAS），以获取相关信息。

    此外，整个系统上的资源表示应遵循特定的准则，例如命名约定，链接格式或数据格式（XML或/和JSON）。

    所有资源都应通过通用方法（例如HTTP GET）进行访问，并使用一致的方法进行类似的修改。

    > 一旦开发人员熟悉您的一个API，他就应该能够对其他API遵循类似的方法。

4. **分层系统**   
    REST允许您使用分层的系统架构，在该架构中，您可以在服务器A上部署API，并在服务器B上存储数据并在服务器C中对请求进行身份验证。客户端通常无法确定它是直接连接到最终服务器还是中​​间连接。

5. **可缓存的**  
    在当今世界中，缓存数据和响应在任何适用/可能的地方都至关重要。我们阅读的网页也是HTML页面的缓存版本。缓存可以提高客户端的性能，并为服务器提供更好的可伸缩性。

    在REST中，缓存应在适用时应用于资源，然后这些资源必须声明自己可缓存。可以在服务器或客户端上实现缓存。
    
    > 管理良好的缓存部分或完全消除了某些客户端-服务器交互，从而进一步提高了可伸缩性和性能。

6. **按需代码（可选）**   
    好吧，这个约束是可选的。大多数时候，您将以XML或JSON的形式发送资源的静态表示。但是，如果需要，您可以自由地return executable code支持应用程序的一部分，例如，客户端可以调用您的API来获取UI小部件呈现代码。这是允许的。

    > 以上所有约束条件都可以帮助您构建真正的RESTful API，并且应该遵循它们。不过，有时您可能会发现自己违反了一两个约束。别担心; 您仍在制作RESTful API，但不是“真正的RESTful”。



## 3. REST资源命名指南

在REST中，主要数据表示称为Resource。从长远来看，拥有一个强大且一致的REST资源命名策略–无疑将证明是最佳的设计决策之一。  

> REST中信息的关键抽象是一种资源。可以命名的任何信息都可以是资源：文档或图像，临时服务（例如“洛杉矶今天的天气”），其他资源的集合，非虚拟对象（例如人）等上。换句话说，任何可能成为作者超文本引用目标的概念都必须符合资源的定义。资源是到一组实体的概念映射，而不是在任何特定时间点对应于该映射的实体。

**一个资源可以是单个的或一个集合**。例如，在一个银行应用中，```customers```代表一个资源集合，```customer```代表单个资源。我们可以使用URL"```/customers```"来标记```customers```，可以使用URL"```/customers/{customerId}```"来标记```customer```

**一个资源可能包括多个子资源**。例如，一个```customer```的子资源```account```可以使用URL"```/customers/{customerId}/accounts```"来标记；类似的，```accounts```的子资源```account```可以标记如下：URL"```/customers/{customerId}/accounts/{accountId}

## 4. REST资源命名实践

### 4.1 使用名词来表示资源

RESTful URI应该引用作为事物（名词）的资源，而不是引用动作（动词），因为名词具有动词所没有的属性–类似于资源具有属性。资源的一些示例是：  
* Users of the system
* User Accounts
* Network Devices etc.  

他们的资源URL应该如下：   
```
* http://api.example.com/device-management/managed-devices    
* http://api.example.com/device-management/managed-devices/{device-id}    
* http://api.example.com/user-management/users/   
* http://api.example.com/user-management/users/{id}
```   

为了更加清晰，让我们将资源原型分为四类**（document, collection, store and controller）**，然后您始终应该以将资源放入一个原型为目标，然后一致地使用其命名约定。为了统一起见，请抵制设计资源的诱惑，这些资源是多个原型的混合体。
* **Document**  
    文档资源是单个概念，类似于对象实例或数据库记录。在REST中，您可以将其视为资源集合中的单个资源。文档的状态表示形式通常包括带有值的字段以及指向其他相关资源的链接。

    使用“单数”名称表示文档资源原型。
    ```
    http://api.example.com/device-management/managed-devices/{device-id}
    http://api.example.com/user-management/users/{id}
    http://api.example.com/user-management/users/admin
    ```  

* **Collection**  
    Collection资源是服务器管理的资源目录。客户可以建议将新资源添加到Collection中。但是，由Collection决定是否创建新资源。Collection资源选择要包含的内容，并确定每个包含的资源的URI。

    使用复数名称表示Collection资源原型。
    ```
    http://api.example.com/device-management/managed-devices
    http://api.example.com/user-management/users
    http://api.example.com/user-management/users/{id}/admin
    ```

* **Store**  
    Store是客户端管理的资源存储库。Store资源可让API客户端放入资源，将其撤回并决定何时删除它们。Store永远不会生成新的URI。取而代之的是，每个存储的资源都有一个URI，该URI是客户端在最初将其放入Store时选择的。

    使用复数名称表示Store资源原型。
    ```
    http://api.example.com/cart-management/users/{id}/carts
    http://api.example.com/song-management/users/{id}/accounts
    ```  
    
* **Controller**  
    Controller  资源为过程概念建模。Controller  资源就像可执行函数一样，带有参数和返回值。输入和输出。

    使用动词表示控制器原型。
    ```
    http://api.example.com/cart-management/users/{id}/cart/checkout
    http://api.example.com/song-management/users/{id}/playlist/play
    ```

### 4.2. 一致性是关键

使用一致的资源命名约定和URI格式，以最大程度地减少歧义并最大程度地提高可读性和可维护性。您可以实现以下设计提示以实现一致性：  
1. **使用正斜杠（/）表示层次关系**  
    URI的路径部分使用正斜杠（/）字符表示资源之间的层次关系。例如
    ```
    http://api.example.com/device-management
    http://api.example.com/device-management/managed-devices
    http://api.example.com/device-management/managed-devices/{id}
    http://api.example.com/device-management/managed-devices/{id}/scripts
    http://api.example.com/device-management/managed-devices/{id}/scripts/{id}
    ````
    
    
2. **不要在URI中使用结尾的正斜杠(/)**  
    作为URI路径中的最后一个字符，正斜杠（/）不添加语义值，并且可能引起混淆。最好将它们完全删除。
    ````
    http://api.example.com/device-management/managed-devices/
    http://api.example.com/device-management/managed-devices     # 这是更好的版本
    ```
    
    
3. **使用连字符（-）来提高URI的可读性**  
    为了使你的URI易于人们扫描和解释，请使用连字符（-）来提高长路径段中名称的可读性。
    ```
    http://api.example.com/inventory-management/managed-entities/{id}/install-script-location     # 可读性强
    http://api.example.com/inventory-management/managedEntities/{id}/installScriptLocation    # 可读性较低
    ```  
    
    
4. **请勿使用下划线（_）**  
    可以使用下划线代替连字符作为分隔符–但是，根据应用程序的字体，下划线（_）字符可能会在某些浏览器或屏幕中被部分遮盖或完全隐藏。

    为避免这种混淆，请使用连字符（-）代替下划线（_）。
    ```
    http://api.example.com/inventory-management/managed-entities/{id}/install-script-location    # More readable
    http://api.example.com/inventory_management/managed_entities/{id}/install_script_location  # More error prone
    ```  
    
    
5. **在URI中使用小写字母**  
    方便时，在URI路径中应始终首选小写字母。

    RFC 3986将URI定义为区分大小写，但方案和主机组件除外。例如
    ```
    http://api.example.org/my-folder/my-doc // 1
    HTTP://API.EXAMPLE.ORG/my-folder/my-doc // 2
    http://api.example.org/My-Folder/my-doc // 3
    ```
    在上面的示例中，1和2相同，但3不是，因为它以大写字母使用My-Folder。
    
    
6. **不要使用文件扩展名**  
    文件扩展名看起来很糟糕，并且没有任何优势。删除它们也会减少URI的长度。没有理由保留它们。

    除上述原因外，如果要使用文件扩展名突出显示API的媒体类型，则应依靠通过Content-Type标头传达的媒体类型来确定如何处理正文内容。
    ```
    http://api.example.com/device-management/managed-devices.xml   # 请勿使用
    http://api.example.com/device-management/managed-devices    # 这是正确的URI
    ```

## 5. 不要在URL中使用CRUD函数名称

URI不应用于指示执行CRUD功能。URI应该用于唯一标识资源，而不是对资源进行任何操作。应该**使用HTTP请求方法**来指示执行了哪个CRUD功能。
```
HTTP GET http://api.example.com/device-management/managed-devices //获取所有设备
HTTP POST http://api.example.com/device-management/managed-devices //创建新设备

HTTP GET http://api.example.com/device-management/managed-devices/{id} //获取给定ID的设备
HTTP PUT http://api.example.com/device-management/managed-devices/{id} //更新给定ID的设备
HTTP DELETE http://api.example.com/device-management/managed-devices/{id} //删除给定ID的设备
```

## 6. 使用查询组件过滤URL集合

很多时候，您会遇到需求，在这些需求中，您将需要根据某些资源属性对资源进行排序，过滤或限制的集合。为此，请勿创建新的API，而应在资源收集API中启用排序，过滤和分页功能，并将输入参数作为查询参数传递。例如
```
http://api.example.com/device-management/managed-devices
http://api.example.com/device-management/managed-devices?region=USA
http://api.example.com/device-management/managed-devices?region=USA&brand=XYZ
http://api.example.com/device-management/managed-devices?region=USA&brand=XYZ&sort=installation-date
```

## 7. 使用正确的HTTP Status Code表示访问状态

## 8. 错误返回使用明确易懂的文本
