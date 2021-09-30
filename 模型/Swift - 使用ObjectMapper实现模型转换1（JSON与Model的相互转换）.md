# Swift - 使用ObjectMapper实现模型转换1（JSON与Model的相互转换）

2017-05-17发布：hangge阅读：17205

**相关文章系列：**
[当前文章] Swift - 使用ObjectMapper实现模型转换1（JSON与Model的相互转换）
[Swift - 使用ObjectMapper实现模型转换2（StaticMappable协议）](https://www.hangge.com/blog/cache/detail_1674.html)
[Swift - 使用ObjectMapper实现模型转换3（高级用法）](https://www.hangge.com/blog/cache/detail_1675.html)
[Swift - 使用ObjectMapper实现模型转换4（与Alamofire结合使用）](https://www.hangge.com/blog/cache/detail_1679.html)

## 一、ObjectMapper的安装与介绍

### 1，基本介绍

**ObjectMapper** 是一个使用 **Swift** 语言编写的数据模型转换框架。使用它，我们可以很方便地将模型对象（类和结构体）转换为 **JSON**，或者根据 **JSON** 生成对应的模型对象。

**Github**主页：https://github.com/Hearst-DD/ObjectMapper

### 2，功能特点

- 可以将 **JSON** 映射到对象
- 可以将对象映射到 **JSON**
- 支持嵌套对象（在数组或字典中单独使用）
- 支持映射过程中的自定义转换
- 支持结构体
- 支持 **Immutable**(目前处于测试阶段)

### 3，安装配置

（1）将下载下来的源码包中 **ObjectMapper.xcodeproj** 拖拽至你的工程中

[![原文:Swift - 使用ObjectMapper实现模型转换1（JSON与Model的相互转换）](https://www.hangge.com/blog_uploads/201705/2017050910214328064.png)](https://www.hangge.com/blog/cache/detail_1673.html#)

（2）工程 -> **General** -> **Embedded Binaries** 项，把 **iOS** 版的 **framework** 添加进来： **ObjectMapper.framework**

[![原文:Swift - 使用ObjectMapper实现模型转换1（JSON与Model的相互转换）](https://www.hangge.com/blog_uploads/201705/2017050910230368606.png)](https://www.hangge.com/blog/cache/detail_1673.html#)

（3）最后，在需要使用 **ObjectMapper** 的地方 **import** 进来就可以了

```swift
import ObjectMapper
```



## 二、模型（Model）与字典（Dictionary）相互转换

### 1，模型定义

要实现映射，我们的模型需要实现 **ObjectMapper** 的 **Mappable** 协议，并实现该协议里的如下两个方法。

```swift
init?(map: Map)
mutating func mapping(map: Map)
```


这里我定义一个用户类（**User**）。注意：**ObjectMapper** 定义了一个 **<-** 操作符来表示成员对象与 **JSON** 中属性的相互映射关系。

```swift
class User: Mappable {
    var username: String?
    var age: Int?
    var weight: Double!
    var bestFriend: User?        // User对象
    var friends: [User]?         // Users数组
    var birthday: Date?
    var array: [AnyObject]?
    var dictionary: [String : AnyObject] = [:]
     
    init(){
    }
     
    required init?(map: Map) {
    }
     
    // Mappable
    func mapping(map: Map) {
        username    <- map["username"]
        age         <- map["age"]
        weight      <- map["weight"]
        bestFriend  <- map["best_friend"]
        friends     <- map["friends"]
        birthday    <- (map["birthday"], DateTransform())
        array       <- map["arr"]
        dictionary  <- map["dict"]
    }
}
```



### 2，将模型转为字典

（1）下面代码中我们定义了两个 **User** 对象，其中一个引用另一个。最后将该对象转为字典，并打印出来。

```swift
let lilei = User()
lilei.username = "李雷"
lilei.age = 18
 
let meimei = User()
meimei.username = "梅梅"
meimei.age = 17
meimei.bestFriend = lilei
 
let meimeiDic:[String: Any] = meimei.toJSON()
print(meimeiDic)
```

（2）运行结果如下：

[![原文:Swift - 使用ObjectMapper实现模型转换1（JSON与Model的相互转换）](https://www.hangge.com/blog_uploads/201705/201705091049457175.png)](https://www.hangge.com/blog/cache/detail_1673.html#)

### 3，将模型数组转为字典数组

（1）下面代码将一个包含两个 **User** 对象的数组转换成字典数组，并打印出来。

[?](https://www.hangge.com/blog/cache/detail_1673.html#)

```swift
let lilei = User()
lilei.username = "李雷"
lilei.age = 18
 
let meimei = User()
meimei.username = "梅梅"
meimei.age = 17
 
let users = [lilei, meimei]
let usersArray:[[String: Any]]  = users.toJSON()
print(usersArray)
```

（2）运行结果如下： 

[![原文:Swift - 使用ObjectMapper实现模型转换1（JSON与Model的相互转换）](https://www.hangge.com/blog_uploads/201705/2017050911115236409.png)](https://www.hangge.com/blog/cache/detail_1673.html#)

### 4，将字典转为模型

```swift
let meimeiDic = ["age": 17, "username": "梅梅",
                 "best_friend": ["age": 18, "username": "李雷"]]
 
let meimei = User(JSON: meimeiDic)
```



### 5，字典数组转为模型数组

```swift
let usersArray = [["age": 17, "username": "梅梅"],
                  ["age": 18, "username": "李雷"]]
 
let users:[User] = Mapper<User>().mapArray(JSONArray: usersArray)
```



## 三、模型（Model）与JSON字符串的相互转换

这里我同样使用上面的用户类（**User**）来演示。



### 1，将模型转为JSON字符串

（1）下面代码中我们定义了两个 **User** 对象，其中一个引用另一个。最后将该对象转为 **json** 串，并打印出来。

```swift
let lilei = User()
lilei.username = "李雷"
lilei.age = 18
 
let meimei = User()
meimei.username = "梅梅"
meimei.age = 17
meimei.bestFriend = lilei
 
let meimeiJSON:String = meimei.toJSONString()!
print(meimeiJSON)
```


（2）运行结果如下：

[![原文:Swift - 使用ObjectMapper实现模型转换1（JSON与Model的相互转换）](https://www.hangge.com/blog_uploads/201705/2017050914235695665.png)](https://www.hangge.com/blog/cache/detail_1673.html#)



### 2，将模型数组转为JSON字符串

（1）下面代码将一个包含两个 **User** 对象的数组转换成字典数组，并打印出来。

```swift
let lilei = User()
lilei.username = "李雷"
lilei.age = 18
 
let meimei = User()
meimei.username = "梅梅"
meimei.age = 17
 
let users = [lilei, meimei]
let json:String  = users.toJSONString()!
print(json)
```


（2）运行结果如下：

[![原文:Swift - 使用ObjectMapper实现模型转换1（JSON与Model的相互转换）](https://www.hangge.com/blog_uploads/201705/2017050914290756996.png)](https://www.hangge.com/blog/cache/detail_1673.html#)



### 3，将JSON字符串转为模型

```swift
let meimeiJSON:String = "{\"age\":17,\"username\":\"梅梅\",\"best_friend\":{\"age\":18,\"username\":\"李雷\"}}"
 
let meimei = User(JSONString: meimeiJSON)
```

### 4，将JSON字符串转为模型数组

```swift
let json = "[{\"age\":18,\"username\":\"李雷\"},{\"age\":17,\"username\":\"梅梅\"}]"
 
let users:[User] = Mapper<User>().mapArray(JSONString: json)!
```



## 四、init?(map: Map)使用介绍

**ObjectMapper** 通过 **Mappable** 协议中的 **init?(map: Map)** 方法来初始化创建对象。我们可以利用这个方法，在对象序列化之前验证 **JSON** 合法性。在不符合的条件时，返回 **nil** 阻止映射发生。
（1）这里我们需要检测 **JSON** 数据中是否包含 **username** 属性。

```swift
class User: Mappable {
    var username: String?
    var age: Int?
    var weight: Double!
     
    init(){
    }
     
    required init?(map: Map){
        // 检查JSON中是否有"username"属性
        if map.JSON["username"] == nil {
            return nil
        }
    }
     
    // Mappable
    func mapping(map: Map) {
        username    <- map["username"]
        age         <- map["age"]
        weight      <- map["weight"]
    }
}
```

（2）下面我们将 **JSON** 字符串转为模型数组，并打印出对象个数。

```swift
let json = "[{\"age\":18,\"username\":\"李雷\"},{\"age\":17}]"
let users:[User] = Mapper<User>().mapArray(JSONString: json)!
print(users.count)
```


（3）运行结果如下，可以看到生成的数组中只有 **1** 个对象，这是由于另一个对象的 **username** 为空，因此被过滤掉了。

[![原文:Swift - 使用ObjectMapper实现模型转换1（JSON与Model的相互转换）](https://www.hangge.com/blog_uploads/201705/2017050915092865516.png)](https://www.hangge.com/blog/cache/detail_1673.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1673.html