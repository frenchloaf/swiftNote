# Swift - 使用ObjectMapper实现模型转换3（高级用法）

2017-05-22发布：hangge阅读：11287

**相关文章系列：**
[Swift - 使用ObjectMapper实现模型转换1（JSON与Model的相互转换）](https://www.hangge.com/blog/cache/detail_1673.html)
[Swift - 使用ObjectMapper实现模型转换2（StaticMappable协议）](https://www.hangge.com/blog/cache/detail_1674.html)
[当前文章] Swift - 使用ObjectMapper实现模型转换3（高级用法）
[Swift - 使用ObjectMapper实现模型转换4（与Alamofire结合使用）](https://www.hangge.com/blog/cache/detail_1679.html)

## 一、嵌套对象的映射

### 1，使用点符号（.）映射嵌套对象

（1）假设我们有如下 **JSON** 字符串

```json
{
    "type" : "S1",
    "distance" : {
        "text" : "102 ft",
        "value" : 31
    }
}
```


（2）要将 **distance** 中 **value** 映射成 **distance** 属性可以使用如下操作：

```swift
func mapping(map: Map) {
    distance <- map["distance.value"]
}
```



### 2，嵌套键值访问同样支持数组

（1）假设我们有如下 **JSON** 字符串

```swift
{
    "type" : "S1",
    "distances" : [
        {
        "text" : "104 ft",
        "value" : 32
        },
        {
        "text" : "102 ft",
        "value" : 31
        }
    ]
}
```


（2）要将 **distances** 里第一个元素的 **value** 值映射成 **distance** 属性可以使用如下操作：



```swift
func mapping(map: Map) {
    distance <- map["distances.0.value"]
}
```



### 3，如果JSON字符串中的键值就包含点（.）则需要关闭这个特性。

（1）假设我们有如下 **JSON** 字符串

```json
{
    "type" : "S1",
    "distance.value" : 31
}
```


（2）要将 **distance.value** 这个键值映射成 **distance** 属性可以使用如下操作：

```swift
func mapping(map: Map) {
    identifier <- map["distance.value", nested: false]
}
```



### 4，如果JSON字符串中的键值就包含点（.）又想要嵌套访问功能

（1）假设我们有如下 **JSON** 字符串

```json
{
    "type" : "S1",
    "com.hangge.info" : {
        "com.hangge.name" : "航歌",
        "com.hangge.version" : 12
    }
}
```


（2）要将 **com.hangge.info** 里的 **com.hangge.name** 键值映射成 **appName** 属性可以使用如下操作（指定分隔符）：



```swift
func mapping(map: Map) {
    appName <- map["com.hangge.info->com.hangge.name", delimiter: "->"]
}
```



## 二、数据自定义变换

**ObjectMapper** 支持在映射过程中实现转换值的自定义变换。要使用变换，只需在 **<-** 操作符的右侧创建一个元组，其中包含 **map["field_name"]** 以及对应的变换对象。



### 1，创建自定义变换类

 （1）首先自定义变换类需要实现 **TransformType** 协议，该协议定义如下：

```swift
public protocol TransformType {
    associatedtype Object
    associatedtype JSON
 
    func transformFromJSON(_ value: Any?) -> Object?
    func transformToJSON(_ value: Object?) -> JSON?
}
```


（2）**ObjectMapper** 自带了一个日期变换类 **DateTransform**，用来实现日期时间与时间戳的相互转换。其实现的就是 **TransformType** 协议。

- 将模型转为 **JSON** 时：**Date** 类型数据转成 **Int** 值
- 将 **JSON** 转为模型时：**Int** 值转成 **Date** 类型数据

```swift
import Foundation
 
open class DateTransform: TransformType {
    public typealias Object = Date
    public typealias JSON = Double
 
    public init() {}
 
    open func transformFromJSON(_ value: Any?) -> Date? {
        if let timeInt = value as? Double {
            return Date(timeIntervalSince1970: TimeInterval(timeInt))
        }
         
        if let timeStr = value as? String {
            return Date(timeIntervalSince1970: TimeInterval(atof(timeStr)))
        }
         
        return nil
    }
 
    open func transformToJSON(_ value: Date?) -> Double? {
        if let date = value {
            return Double(date.timeIntervalSince1970)
        }
        return nil
    }
}
```


（3）使用样例

```swift
class User: Mappable {
    var username: String?
    var birthday: Date?
     
    init(){
    }
     
    required init?(map: Map) {
    }
     
    // Mappable
    func mapping(map: Map) {
        username    <- map["username"]
        birthday    <- (map["birthday"], DateTransform())
    }
}
 
let lilei = User()
lilei.username = "李雷"
lilei.birthday = Date()
 
let json = lilei.toJSONString()!
print(json)
```

[![原文:Swift - 使用ObjectMapper实现模型转换3（高级用法）](https://www.hangge.com/blog_uploads/201705/2017050917183462183.png)](https://www.hangge.com/blog/cache/detail_1675.html#)

### 2，使用 TransformOf 快速创建变换

如果觉得要创建一个变换类比较麻烦，我们还可以使用 **TransformOf**，通过闭包的形式创建变换。

（1）比如下面实现 **JSON String** 值与模型里 **Int** 类型属性的变换。 

```swift
let transform = TransformOf<Int, String>(fromJSON: { (value: String?) -> Int? in
    // 将值从 String? 变换为 Int?
    return Int(value!)
}, toJSON: { (value: Int?) -> String? in
    // 将值从 Int? 变换为 String?
    if let value = value {
        return String(value)
    }
    return nil
})
 
id <- (map["id"], transform)
```


（2）上面代码还可以进一步精简：

```swift
id <- (map["id"], TransformOf<Int, String>(fromJSON: { Int($0!) },
                                           toJSON: { $0.map { String($0) } }))
```

## 三、泛型对象（Generic Objects）

**ObjectMapper** 同样支持泛型类，只要这个泛型类型实现 **Mappable** 协议。下面是一个简单的样例：

```swift
class Result<T: Mappable>: Mappable {
    var result: T?
 
    required init?(map: Map){
 
    }
 
    func mapping(map: Map) {
        result <- map["result"]
    }
}
 
let result = Mapper<Result<User>>().map(JSON)
```



## 四、映射上下文（Mapping Context）

**Map** 对象在映射的时候，可以传递一个可选的 **MapContext** 对象。这个开发人员可以利用这个对象在映射的时候传递一些信息，然后 **Map** 对象内部通过上下文来做出相关的映射的决策，比如：哪些需要映射、如何映射等。
该功能只需创建一个实现 **MapContext** 协议（这是一个空协议）的对象，并在 **Mapper** 初始化期间将其传递即可。



```swift
struct Context: MapContext {
    var importantMappingInfo = "Info that I need during mapping"
}
 
class User: Mappable {
    var name: String?
     
    required init?(map: Map){
     
    }
     
    func mapping(map: Map){
        if let context = map.context as? Context {
            //使用上下文来做相关的映射决策
        }
    }
}
 
let context = Context()
let user = Mapper<User>(context: context).map(JSONString)
```



## 五、与Realm配合使用

**ObjectMapper** 同 **Realm** 是可以一起使用的。我们可以使用 **ObjectMapper** 生成 **Realm** 模型，下面是一个简单的类结构代码：

```swift
class Model: Object, Mappable {
    dynamic var name = ""
 
    required convenience init?(map: Map) {
        self.init()
    }
 
    func mapping(map: Map) {
        name <- map["name"]
    }
}
```

如果我们要序列化关联的 **RealmObjects**，可以使用 [ObjectMapper+Realm](https://github.com/jakenberg/ObjectMapper-Realm)。这是一个简单的 **Realm** 扩展，可以将任意 **JSON** 序列化为 **Realm** 的 **List**。


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1675.html