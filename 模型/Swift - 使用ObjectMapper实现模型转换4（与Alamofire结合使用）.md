# Swift - 使用ObjectMapper实现模型转换4（与Alamofire结合使用）

2017-05-24发布：hangge阅读：5161

**相关文章系列：**
[Swift - 使用ObjectMapper实现模型转换1（JSON与Model的相互转换）](https://www.hangge.com/blog/cache/detail_1673.html)
[Swift - 使用ObjectMapper实现模型转换2（StaticMappable协议）](https://www.hangge.com/blog/cache/detail_1674.html)
[Swift - 使用ObjectMapper实现模型转换3（高级用法）](https://www.hangge.com/blog/cache/detail_1675.html)
[当前文章] Swift - 使用ObjectMapper实现模型转换4（与Alamofire结合使用）


**Alamofire** 这个优秀的第三方网络操作库大家想必都用过，我之前也在网站上写过许多相关的文章。通常情况下，如果加载的是 **JSON** 格式的数据，**Alamofire** 可以自动将其转换为字典（**Dictionary**）。

但有时我们想将响应结果转换成 **Swift** 对象，那么借助 **AlamofireObjectMapper**，并配合 **ObjectMapper** 就可以实现。

## 一、AlamofireObjectMapper 的安装与介绍

### 1，基本介绍

**AlamofireObjectMapper** 是一个简单的 **Alamofire** 扩展，它通过使用 **ObjectMapper** 来实现自动将 **JSON** 响应数据映射到 **Swift** 对象。

### 2，安装配置

（1）要使用 **AlamofireObjectMapper** 之前，我们项目中肯定要先配置好 **Alamofire** 和 **ObjectMapper**。这两个库的配置参考我之前的文章：

- [Swift - HTTP网络操作库Alamofire使用详解1（配置，以及数据请求）](https://www.hangge.com/blog/cache/detail_970.html)
- [Swift - 使用ObjectMapper实现模型转换1（JSON与Model的相互转换）](https://www.hangge.com/blog/cache/detail_1673.html)

（2）从 **GitHub** 上下载 **AlamofireObjectMapper** 最新的代码：https://github.com/tristanhimmelman/AlamofireObjectMapper

（3）将下载下来的源码包中 **AlamofireObjectMapper.xcodeproj** 拖拽至你的工程中

[![原文:Swift - 使用ObjectMapper实现模型转换4（与Alamofire结合使用）](https://www.hangge.com/blog_uploads/201705/2017051215440116925.png)](https://www.hangge.com/blog/cache/detail_1679.html#)

（4）工程 -> **General** -> **Embedded Binaries** 项，把 **iOS** 版的 **framework** 添加进来：**AlamofireObjectMapper.framework**

[![原文:Swift - 使用ObjectMapper实现模型转换4（与Alamofire结合使用）](https://www.hangge.com/blog_uploads/201705/2017051215444715384.png)](https://www.hangge.com/blog/cache/detail_1679.html#)

（5）最后，在需要使用 **AlamofireObjectMapper** 的地方 **import** 进来就可以了

```swift
import AlamofireObjectMapper
```



## 二、使用说明

### 1，获取 JSON 数据并转换为模型对象

（1）假设我们请求的 **JSON** 数据如下：

```json
{
    "location": "北京",   
    "three_day_forecast": [
        {
            "conditions": "多云",
            "day" : "星期一",
            "temperature": 28
        },
        {
            "conditions": "小雨",
            "day" : "星期二",
            "temperature": 22
        },
        {
            "conditions": "大雨",
            "day" : "星期三",
            "temperature": 20
        }
    ]
}
```


（2）我们在项目中定义如下模型：

```swift
import ObjectMapper
 
//天气预报
class WeatherResponse: Mappable {
    //位置
    var location: String?
     
    //未来3天的天气
    var threeDayForecast: [Forecast]?
     
    required init?(map: Map){
    }
     
    //映射
    func mapping(map: Map) {
        location <- map["location"]
        threeDayForecast <- map["three_day_forecast"]
    }
}
 
//气候信息
class Forecast: Mappable {
    //日期
    var day: String?
    //温度
    var temperature: Int?
    //天气状况
    var conditions: String?
     
    required init?(map: Map){
    }
     
    //映射
    func mapping(map: Map) {
        day <- map["day"]
        temperature <- map["temperature"]
        conditions <- map["conditions"]
    }
}
```


（3）下面样例使用 **Alamofire** 加载数据，并自动完成模型的映射。

```swift
import UIKit
import Alamofire
import AlamofireObjectMapper
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let url = "http://www.hangge.com/weather.php"
        Alamofire.request(url).responseObject { (response: DataResponse<WeatherResponse>) in
             
            //得到天气模型对象
            let weatherResponse = response.result.value
            print("\(weatherResponse?.location ?? "")，未来三天天气：")
             
            //遍历对象中的每日天气数组
            if let threeDayForecast = weatherResponse?.threeDayForecast {
                for forecast in threeDayForecast {
                    print(forecast.day ?? "",
                          forecast.conditions ?? "",
                          forecast.temperature ?? "")
                }
            }
        }
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

运行结果如下：

[![原文:Swift - 使用ObjectMapper实现模型转换4（与Alamofire结合使用）](https://www.hangge.com/blog_uploads/201705/2017051215533852183.png)](https://www.hangge.com/blog/cache/detail_1679.html#)



### 2，获取 JSON 数据并转换为数组（Array）

（1）假设我们请求的 **JSON** 数据如下：

```json
[
        {
            "conditions": "多云",
            "day" : "星期一",
            "temperature": 28
        },
        {
            "conditions": "小雨",
            "day" : "星期二",
            "temperature": 22
        },
        {
            "conditions": "大雨",
            "day" : "星期三",
            "temperature": 20
        }
]
```


（2）下面样例使用 **Alamofire** 加载数据，并自动完成模型的映射。（**Forecast** 模型定义见上方代码）

```swift
import UIKit
import Alamofire
import AlamofireObjectMapper
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let url = "http://www.hangge.com/weather.php"
        Alamofire.request(url).responseArray { (response: DataResponse<[Forecast]>) in
             
            //获取天气数组
            let forecastArray = response.result.value
             
            //遍历数组
            if let forecastArray = forecastArray {
                for forecast in forecastArray {
                    print(forecast.day ?? "",
                    forecast.conditions ?? "",
                    forecast.temperature ?? "")
                }
            }
        }
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

运行结果如下：

[![原文:Swift - 使用ObjectMapper实现模型转换4（与Alamofire结合使用）](https://www.hangge.com/blog_uploads/201705/2017051216014247624.png)](https://www.hangge.com/blog/cache/detail_1679.html#)



## 三、补充说明

### 1，嵌套对象的映射

（1）**AlamofireObjectMapper** 同样支持使用点符号（**.**）映射嵌套对象。比如有如下 **JSON** 字符串：

```json
"distance" : {
     "text" : "102 ft",
     "value" : 31
}
```


（2）我们可以使用如下方式访问嵌套对象

```swift
func mapping(map: Map) {
    distance <- map["distance.value"]
}
```



### 2，使用路径（KeyPath）映射指定部分数据

（1）假设我们有如下 **JSON** 数据，其中真正要映射的数据在 **data** 这个键下面：

```json
{
    "data": {
        "location": "Toronto, Canada",
        "three_day_forecast": [
            {
                "conditions": "多云",
                "day": "星期一",
                "temperature": 28
            },
            {
                "conditions": "小雨",
                "day": "星期二",
                "temperature": 22
            },
            {
                "conditions": "大雨",
                "day": "星期三",
                "temperature": 20
            }
        ]
    }
}
```


（2）我们可以使用 **keyPath** 参数深入到指定层次再进行数据映射。**keyPath** 同样支持深入到多个层级下，比如：**data.three_day_forecast**

```swift
let url = "http://www.hangge.com/weather.php"
Alamofire.request(url).responseObject(keyPath: "data") {
    (response: DataResponse<WeatherResponse>) in
     
    //得到天气模型对象
    let weatherResponse = response.result.value
    print("\(weatherResponse?.location ?? "")，未来三天天气：")
     
    //遍历对象中的每日天气数组
    if let threeDayForecast = weatherResponse?.threeDayForecast {
        for forecast in threeDayForecast {
            print(forecast.day ?? "",
                  forecast.conditions ?? "",
                  forecast.temperature ?? "")
        }
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1679.html