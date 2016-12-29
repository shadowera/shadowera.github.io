---
layout:     post
title:      "Firebase-part1-Realtime Database"
subtitle:   " A new way to build apps"
date:       2016-12-28 12:00:00
author:     "shadowera"
header-img: "img/about-bg.jpg"
catalog: true
tags:
    - android 
    - Firebase
---
# Before

既然Firebase是做后端数据库起家的，那就自然少不了实时数据库这一块了 

实时数据库，顾名思义，就是不需要客户端自己去轮询，而是每当监听的区域发生变动时，数据库都会通知客户端去更新，设备将会在毫秒级的时间内接收到数据更新 

除此之外，考虑到写数据时遇到的无网络连接问题，Firebase的数据库API使用了本地缓存，使得在离线状态下也能保持读写不失败，并且会在网络恢复连接时和服务器进行同步 
和常见的数据库服务器不同，Firebase使用JSON存储数据，这使得数据结构非常可观

# 数据结构

所有 Firebase Realtime Database 数据都被存储为 JSON 对象。您可将该数据库视为云托管 JSON 树。 

该数据库与 SQL 数据库不同，没有任何表格或记录。 当您将数据添加至 JSON 树时，它变为现有 JSON 结构中的一个节点。

* 避免嵌套数据
* 平展数据结构

# Rules
Firebase Realtime Database Rules 确定谁有您的数据库的读取和写入权限、您的数据结构如何，以及存在什么索引。

这些规则存在于 Firebase 服务器上，并且始终自动执行。 每一个读取和写入请求只有当您的规则允许时才能完成。

|  |   |
|-----|---|
|.read	|描述是否允许用户读取数据以及何时读取。|
|.write	|描述是否允许写入数据以及何时写入。|
|.validate	|定义值的正确格式、值是否包含子属性以及值的数据类型。|
|.indexOn	|指定要编入索引的子项以支持排序和查询。|



# Android Firebase Database api
## 读写授权

数据库的默认rule是：

```JSON
{
  "rules": {
    ".read": "auth != null",
    ".write": "auth != null"
  }
}
```

这里表示根节点读写的条件是认证不为空，说明需要身份验证才能获得权限（auth是规则的预定义变量，后面会讲到） 
对应字符串中的逻辑计算结果决定了此节点下的读写权限 

.read和.write是级联的，也就是说，父节点的读写权限直接决定子节点的读写权限，子节点的设置将被忽略

```JSON
{
  "rules": {
      "father": {
          ".read": false
          "child": {
              ".read": true
          }
      }
  }
}
```

由于父节点禁止了读取，即使子节点允许读取，也是没有用的

## 数据验证
validate将会对新写入的数据进行验证，验证通过的数据方可写入。
因此.validate发挥作用需要.write为true，另外，因为.validate是检查新数据，
因此常需要和newData变量搭配使用（newData是预定义常量，表示写入操作后应有的数据，包含新数据和未改动数据）

```JSON
{
    rules: {
        age: {
            \\值为数字并且大于0
            .validate: "newData.isNumber() && newData.val() >= 0"
        }
    }
}
```

```JSON
{
  "rules": {
    "messages": {
      "$message": {
        // only messages from the last ten minutes can be read
        ".read": "data.child('timestamp').val() > (now - 600000)",

        // new messages must have a string content and a number timestamp
        ".validate": "newData.hasChildren(['content', 'timestamp']) && newData.child('content').isString() && newData.child('timestamp').isNumber()"
      }
    }
  }
}
```

## 预定义变量

|  |   |
--------|-----
now	|当前时间，linux毫秒数
root	|RuleDataSnapshot，表示尝试操作之前在 Firebase 数据库中存在的根路径。
newData	|RuleDataSnapshot， 表示尝试操作后应存在的数据。该数据包含写入的新数据和现有数据。
data	|RuleDataSnapshot， 表示尝试操作之前存在的数据。
$variables	|用于表示 ID 和动态子键的通配符路径。
auth	|表示通过身份验证的用户的令牌负载。

## 索引

使用.indexOn来规划索引，索引可以帮助提高检索性能：

```JSON
{
    "rules": {
        "students": {
            "indexOn": ["age"]
        }
    }
}
```

服务器设置了.indexOn规则后，客户端使用orderByValue()向服务器检索学生信息，就能获得更好的性能

## android api

Firebase的SDK当然是必不可少的了，为了使用数据库相关的API，需要在应用的依赖项中添加：

```java
    compile 'com.google.firebase:firebase-database:...'
```

为了和数据库交互，需要得到数据库的一个实例，

而且具体的交互对象是数据库中的某个节点，这时就需要获得这个节点的一个引用：

```java
FirebaseDatabase database = FirebaseDatabase.getInstance();
DatabaseReference mRef = database.getReference("students");
```

### 写
写操作包括了增和删，但是SDK并没有对这两个操作进行明显的区分，而是将其视为写操作，SDK中有四种写操作：
| 方法 | 说明 |
-----|----
setValue()	|在指定的路径，将数据写入或替换
push()	|生成唯一ID，路径进入到此ID中
updateChildren()	|更新此路径中的部分键值，而不是所有数据
runTransaction()	|进行并发更新

### setValue(Object)是最基本的写操作。得到某个节点的引用后，调用setValue()即可写入数据，
如果该节点原本就有数据，则会覆盖该节点下的所有数据 
由于数据库使用了JSON作为数据格式，setValue()可接受的参数类型有：

1. 
    * Boolean
    * Long
    * Double
    * Map< String, Object >
    * List< Object >

2. 传递自定义 Java 对象（如果定义该对象的类的默认构造函数不接受参数，且为要指定的属性提供了公用 getter）。 
如果使用 Java 对象，则对象的内容将自动以嵌套方式映射到子位置。

### 离线写入数据

如果客户端失去网络连接，您的应用将继续正常运行。

连接到 Firebase 数据库的每个客户端均维护任何活动数据各自的内部版本。 写入数据时，首先将其写入此本地版本。 然后，Firebase 客户端尽最大程度将这些数据与远程数据库服务器以及其他客户端同步。

因此，在将任何数据写入服务器之前，对数据库执行的所有写入会立即触发本地事件。 这意味着应用将保持随时响应，无论网络延迟或连接情况如何均是如此。

重新建立连接之后，您的应用将收到一组适当的事件，以便客户端与当前服务器状态同步，而不必编写任何自定义代码。

### 检索数据
Firebase 数据可通过将异步侦听器附加到 FirebaseDatabase 引用来检索。 
侦听器由数据的初始状态触发一次，并在数据发生任何更改时再次触发。
您可以侦听以下类型的数据检索事件：

|侦听器	|事件回调	|典型用法|
|--------|---------|-------|
|ValueEventListener	| onDataChange()	| 读取和侦听对路径的全部内容所做的更改。|
|ChildEventListener	| onChildAdded()	| 检索项目列表，或侦听项目列表中是否添加了新项目。 建议与 onChildChanged() 和 onChildRemoved() 配合使用，从而监控对列表所做的更改|
| |onChildChanged()| 侦听对列表中的项目所做的更改。与 onChildAdded() 和 onChildRemoved() 结合使用，从而监控对列表所做的更改。|
||onChildRemoved()	| 侦听正从列表中删除的项目。与 onChildAdded() 和 onChildChanged() 结合使用， 从而监控对列表所做的更改。|
||onChildMoved()	| 与经过排序的数据配合使用，从而侦听对项目优先级所做的更改。|

### 排序数据

要检索排序数据，请先指定排序依据方法之一，确定如何对结果排序

方法	|用法
------|-----
orderByChild()	|按指定子键的值对结果排序。
orderByKey()	|按子键对结果排序。
orderByValue()	|按子值对结果排序。
orderByPriority()	|按节点的指定优先级对结果排序。

### 过滤数据
要过滤数据，您可以结合使用限制或范围方法之一与排序依据方法来构造查询。


方法	|用法
------|------
limitToFirst()	|设置要从排序结果列表开头返回的最大项目数。
limitToLast()	|设置要从排序结果列表结尾返回的最大项目数。
startAt()	|返回大于或等于指定键、值或优先级的项目，具体取决于所选的排序依据方法。
endAt()	|返回小于或等于指定键、值或优先级的项目，具体取决于所选的排序依据方法。
equalTo()	|返回等于指定键、值或优先级的项目，具体取决于所选的排序依据方法。

# 服务器

* 创建帐户并配置项目
* 操作数据库









