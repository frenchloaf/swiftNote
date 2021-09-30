# Swift - 使用ObjectMapper实现模型转换2（StaticMappable协议）

2017-05-19发布：hangge阅读：3896

**相关文章系列：**
[Swift - 使用ObjectMapper实现模型转换1（JSON与Model的相互转换）](https://www.hangge.com/blog/cache/detail_1673.html)
[当前文章] Swift - 使用ObjectMapper实现模型转换2（StaticMappable协议）
[Swift - 使用ObjectMapper实现模型转换3（高级用法）](https://www.hangge.com/blog/cache/detail_1675.html)
[Swift - 使用ObjectMapper实现模型转换4（与Alamofire结合使用）](https://www.hangge.com/blog/cache/detail_1679.html)


在前面的文章中我们使用的都是 **ObjectMapper** 的 **Mappable** 协议，本文接着介绍另一个协议：**StaticMappable**



### 1，StaticMappable协议介绍

（1）同 **Mappable** 协议一样，**StaticMappable** 也 **BaseMappable** 的子协议。
（2）**ObjectMapper** 通过该协议如下方法获取相应的映射对象（这个对象当然也要符合 **BaseMappable** 协议）。同时，我们也可以在使用这个方法在对象序列化之前验证 **JSON**。

```swift
class func objectForMapping(map: Map) -> BaseMappable?
```



### 2，使用样例

（1） 这里我定义了 **3** 个模型类。

- 基类 **Vehicle** 实现 **StaticMappable** 协议，而 **Car** 和 **Bus** 是 **Vehicle** 的两个子类。
- **Vehicle** 中有个 **type** 属性，用来表示汽车类型。**objectForMapping** 方法便是根据该属性获取相应的映射对象。
- **Car** 和 **Bus** 又有自己单独的属性，分别是：**name** 和 **fee**。

```swift
//交通工具
class Vehicle: StaticMappable {
    //类型
    var type: String?
     
    //获取映射对象
    class func objectForMapping(map: Map) -> BaseMappable? {
        if let type: String = map["type"].value() {
            switch type {
            case "car":
                return Car()
            case "bus":
                return Bus()
            default:
                return Vehicle()
            }
        }
        return nil
    }
     
    init(){
    }
     
    func mapping(map: Map) {
        type <- map["type"]
    }
}
 
//小汽车
class Car: Vehicle {
    //名字
    var name: String?
     
    override func mapping(map: Map) {
        super.mapping(map: map)
        name <- map["name"]
    }
}
 
//公交车
class Bus: Vehicle {
    //费用
    var fee: Int?
     
    override func mapping(map: Map) {
        super.mapping(map: map)
        fee <- map["fee"]
    }
}
```


（2）下面我们将一个 **JSON** 字符串转为模型数组。注意 **JSON** 数据中包含多种类型的模型对象，通过 **type** 字段进行区分。

```swift
let JSON = [["type": "car", "name": "奇瑞QQ"],
            ["type": "bus", "fee": 2],
            ["type": "vehicle"]]
 
if let vehicles = Mapper<Vehicle>().mapArray(JSONArray: JSON){
    print("交通工具数量：\(vehicles.count)")
    if let car = vehicles[0] as? Car {
        print("小汽车名字：\(car.name!)")
    }
    if let bus = vehicles[1] as? Bus {
        print("公交车费用：\(bus.fee!) 元")
    }
}
```


（3）运行结果如下：

[![原文:Swift - 使用ObjectMapper实现模型转换2（StaticMappable协议）](https://www.hangge.com/blog_uploads/201705/2017050915590780519.png)](https://www.hangge.com/blog/cache/detail_1674.html#)