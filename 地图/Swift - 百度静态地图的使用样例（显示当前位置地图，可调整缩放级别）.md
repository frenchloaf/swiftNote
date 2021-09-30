# Swift - 百度静态地图的使用样例（显示当前位置地图，可调整缩放级别）

2016-05-16发布：hangge阅读：2444

通过系统提供的设备定位的框架（**CoreLocation**），我们可以很方便地获取到设备当前的各种GPS位置信息。

**具体用法可以参考我原来写的文章：**
[Swift - 使用CoreLocation实现定位（经纬度、海拔、速度、距离等）](https://www.hangge.com/blog/cache/detail_783.html)
[Swift - 经纬度位置坐标与真实地理位置相互转化](https://www.hangge.com/blog/cache/detail_785.html)

但有时如果只显示个经纬度坐标值，或者真实地理位置信息还是不够直观。如果能显示个地图就好了，这时借助系统提供的MapKit框架即可。

[Swift - 使用MapKit显示地图，并在地图上做标记](https://www.hangge.com/blog/cache/detail_787.html)

本文介绍另一种显示地图的方式：使用百度提供的静态图API。



**1，百度地图静态图API介绍**
（1）通过url地址即可获取地图的静态图片，然后显示出来。 

（2）请求时可以带上各种参数，比如：坐位位置、图片尺寸、缩放级别等等。具体可参考：[百度在线API文档](http://developer.baidu.com/map/static-1.htm)。

**2，样例说明**

（1）下面样例首先通过 **CoreLocation** 获取到当前的经纬度坐标。
（2）通过静态图API获取到该坐标位置的地图图片，并显示。
（3）通过页面上的滑块，我们可以调整地图级别（默认11）
[![原文:Swift - 百度静态地图的使用样例（显示当前位置地图，可调整缩放级别）](https://www.hangge.com/blog_uploads/201605/2016051416410084592.png)](https://www.hangge.com/blog/cache/detail_1183.html#)[![原文:Swift - 百度静态地图的使用样例（显示当前位置地图，可调整缩放级别）](https://www.hangge.com/blog_uploads/201605/2016051416410865232.png)](https://www.hangge.com/blog/cache/detail_1183.html#)[![原文:Swift - 百度静态地图的使用样例（显示当前位置地图，可调整缩放级别）](https://www.hangge.com/blog_uploads/201605/201605141641181347.png)](https://www.hangge.com/blog/cache/detail_1183.html#)

**3，代码如下**

```swift
import UIKit
import CoreLocation
 
class ViewController: UIViewController, CLLocationManagerDelegate {
     
    //定位管理器
    let locationManager:CLLocationManager = CLLocationManager()
     
    //记录经纬度
    var longitude:CLLocationDegrees!
    var latitude:CLLocationDegrees!
     
    @IBOutlet weak var label1: UILabel!
    @IBOutlet weak var imageView: UIImageView!
    @IBOutlet weak var slider: UISlider!
    @IBOutlet weak var zoomLabel: UILabel!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //设置定位服务管理器代理
        locationManager.delegate = self
        //设置定位进度
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        //更新距离
        locationManager.distanceFilter = 100
        ////发送授权申请
        locationManager.requestAlwaysAuthorization()
        if (CLLocationManager.locationServicesEnabled())
        {
            //允许使用定位服务的话，开启定位服务更新
            locationManager.startUpdatingLocation()
            print("定位开始")
        }
    }
     
    //滑块值改变
    @IBAction func sliderDidchange(sender: AnyObject) {
        //显示地图
        showBaiduMap()
    }
     
    //定位改变执行，可以得到新位置、旧位置
    func locationManager(manager: CLLocationManager,
                         didUpdateLocations locations: [CLLocation]) {
        //获取最新的坐标
        let currLocation:CLLocation = locations.last!
        //获取经纬度
        longitude = currLocation.coordinate.longitude
        latitude = currLocation.coordinate.latitude
        //显示经纬度
        label1.text = "经度：\(longitude)\n纬度：\(latitude)"
         
        //显示地图
        showBaiduMap()
    }
     
    //加载地图
    func showBaiduMap() {
        //地图图片宽度,取值范围：(0, 1024]。
        let width = min(1024, imageView.frame.width*UIScreen.mainScreen().scale)
        //地图图片高度,取值范围：(0, 1024]。
        let height = min(1024, imageView.frame.height*UIScreen.mainScreen().scale)
        //地图缩放级别,取值范围：[3, 19]。
        let zoom = Int(slider.value)
        zoomLabel.text = "地图级别：\(Int(slider.value))"
         
        //定义NSURL对象
        let url = NSURL(string: "http://api.map.baidu.com/staticimage?" +
            "center=\(longitude),\(latitude)&markers=\(longitude),\(latitude)" +
            "&zoom=\(zoom)&width=\(width)&height=\(height)")
        //从网络获取数据流
        let data = NSData(contentsOfURL: url!)
        //通过数据流初始化图片
        let newImage = UIImage(data: data!)
        self.imageView.image = newImage
    }
}
```

源码下载：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1183.zip](https://www.hangge.com/blog_uploads/201605/201605141649418468.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1183.html