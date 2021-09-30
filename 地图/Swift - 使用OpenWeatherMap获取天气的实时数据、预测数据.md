# Swift - 使用OpenWeatherMap获取天气的实时数据、预测数据

2016-06-21发布：hangge阅读：6583

我们做一些Web应用或者是手机应用，有时会需要展示出实时的天气数据或者天气预报数据。这里推荐一个好用的天气API服务：**OpenWeatherMap**。

**1，OpenWeatherMap介绍**

（1）**OpenWeatherMap** 提供广泛的气象数据，如包含当前天气的地图，一周预报，降水，风，云，来自气象站的其他数据。
（2）数据结果提供 **JSON**、**XML**,、以及 **HTML** 等多种格式。
（3）免费用户可以使用绝大部分功能。同收费用户比起来，只是对每分钟查询次数有限制（60次/分钟）。但这个也足够我们使用了。

**2，OpenWeatherMap注册** 

要使用 **OpenWeatherMap** 的服务，首先我们要到它的官网上注册一个帐号，地址：[http://openweathermap.org](http://openweathermap.org/)
然后再登录帐号，注册个 **apikey**。这个在后面获取气候信息时参数中需要带上。（注册这些都是免费的。）

[![原文:Swift - 使用OpenWeatherMap获取天气的实时数据、预测数据](https://www.hangge.com/blog_uploads/201606/2016062014334460452.png)](https://www.hangge.com/blog/cache/detail_1157.html#)

**3，API提供的功能介绍**
（1）实时天气数据
（2）最近5天的天气预报数据（时间精度：每3小时一条数据）
（3）最近16天的天气预报数据（时间精度：每天一条数据）
（4）历史数据查询
（5）紫外线指数查询
（6）提供天气图层（Weather map layers）
（7）气象站数据提供
（8）批量数据下载
（9）空气污染查询
官方API说明地址：[http://openweathermap.org/api ](http://openweathermap.org/api)。下面我通过两个简单的样例，演示使用方法。

**4，实时天气查询**
API支持根据城市名称、城市ID、城市邮编、经纬度坐标这4种方式查询对应城市的天气信息。
下面样例使用城市名称来查询，查询出北京当前的实时天气，效果图如下： 



[![原文:Swift - 使用OpenWeatherMap获取天气的实时数据、预测数据](https://www.hangge.com/blog_uploads/201606/2016062016012866298.png)](https://www.hangge.com/blog/cache/detail_1157.html#)


（注意：这里对返回的 **JSON** 数据我使用 **SwiftyJSON** 来解析，具体用法可以参考我原来写的这篇文章：[Swift - SwiftyJSON的使用祥解](https://www.hangge.com/blog/cache/detail_968.html)）

```swift
import UIKit
 
class ViewController: UIViewController {
     
    let apiId = "12b2817fbec86915a6e9b4dbbd3d9036"
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        getCurrentWeatherData();
    }
     
    //获取当前天气数据（北京）
    func getCurrentWeatherData(){
        let urlStr = "http://api.openweathermap.org/data/2.5/weather?q=beijing&units=metric&appid=\(apiId)"
        let url = NSURL(string: urlStr)!
        guard let weatherData = NSData(contentsOfURL: url) else { return }
         
        //将获取到的数据转为json对象
        let jsonData = JSON(data: weatherData)
         
        //日期格式化输出
        let dformatter = NSDateFormatter()
        dformatter.dateFormat = "yyyy年MM月dd日 HH:mm:ss"
         
        print("城市：\(jsonData["name"].string!)")
         
        let weather = jsonData["weather"][0]["main"].string!
        print("天气：\(weather)")
        let weatherDes = jsonData["weather"][0]["description"].string!
        print("详细天气：\(weatherDes)")
         
        let temp = jsonData["main"]["temp"].number!
        print("温度：\(temp)°C")
         
        let humidity = jsonData["main"]["humidity"].number!
        print("湿度：\(humidity)%")
         
        let pressure = jsonData["main"]["pressure"].number!
        print("气压：\(pressure)hpa")
         
        let windSpeed = jsonData["wind"]["speed"].number!
        print("风速：\(windSpeed)m/s")
         
        let lon = jsonData["coord"]["lon"].number!
        let lat = jsonData["coord"]["lat"].number!
        print("坐标：[\(lon),\(lat)]")
         
        let timeInterval1 = NSTimeInterval(jsonData["sys"]["sunrise"].number!)
        let date1 = NSDate(timeIntervalSince1970: timeInterval1)
        print("日出时间：\(dformatter.stringFromDate(date1))")
         
        let timeInterval2 = NSTimeInterval(jsonData["sys"]["sunset"].number!)
        let date2 = NSDate(timeIntervalSince1970: timeInterval2)
        print("日落时间：\(dformatter.stringFromDate(date2))")
         
        let timeInterval3 = NSTimeInterval(jsonData["dt"].number!)
        let date3 = NSDate(timeIntervalSince1970: timeInterval3)
        print("数据时间：\(dformatter.stringFromDate(date3))")
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


**5，未来天气预测**
下面样例查询出未来5天内，北京的天气预报数据。效果图如下：



[![原文:Swift - 使用OpenWeatherMap获取天气的实时数据、预测数据](https://www.hangge.com/blog_uploads/201606/2016062016260232076.png)](https://www.hangge.com/blog/cache/detail_1157.html#)



```swift
import UIKit
 
class ViewController: UIViewController {
     
    let apiId = "12b2817fbec86915a6e9b4dbbd3d9036"
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        getForcastWeatherData();
    }
     
    //获取未来天气数据（北京）
    func getForcastWeatherData(){
        let urlStr = "http://api.openweathermap.org/data/2.5/forecast?q=beijing&units=metric&appid=\(apiId)"
        let url = NSURL(string: urlStr)!
        guard let weatherData = NSData(contentsOfURL: url) else { return }
         
        //将获取到的数据转为json对象
        let weatherJson = JSON(data: weatherData)
         
        print("城市：\(weatherJson["city"]["name"].string!)")
         
        let lon = weatherJson["city"]["coord"]["lon"].number!
        let lat = weatherJson["city"]["coord"]["lat"].number!
        print("坐标：[\(lon),\(lat)]")
         
        //日期格式化输出
        let dformatter = NSDateFormatter()
        dformatter.dateFormat = "yyyy年MM月dd日 HH:mm:ss"
         
        //遍历数据
        for (_,jsonData):(String, JSON) in weatherJson["list"] {
            let timeInterval = NSTimeInterval(jsonData["dt"].number!)
            let date = NSDate(timeIntervalSince1970: timeInterval)
            print("--- 时间：\(dformatter.stringFromDate(date)) ---")
             
            let weather = jsonData["weather"][0]["main"].string!
            print("天气：\(weather)")
            let weatherDes = jsonData["weather"][0]["description"].string!
            print("详细天气：\(weatherDes)")
             
            let temp = jsonData["main"]["temp"].number!
            print("温度：\(temp)°C")
             
            let humidity = jsonData["main"]["humidity"].number!
            print("湿度：\(humidity)%")
             
            let pressure = jsonData["main"]["pressure"].number!
            print("气压：\(pressure)hpa")
             
            let windSpeed = jsonData["wind"]["speed"].number!
            print("风速：\(windSpeed)m/s")
        }
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1157.html