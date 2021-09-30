# Swift - 百度地图SDK的配置和使用（附样例）

2016-09-07发布：hangge阅读：2968

在之前我写过一篇关于百度地图的文章：[Swift - 百度静态地图的使用样例](https://www.hangge.com/blog/cache/detail_1183.html)。当时使用的是百度提供的静态图 **API**，即通过 **url** 地址（附带有经纬度、缩放级别等参数）来获取地图图片，并显示。

但如果要实现更多的用户交互效果，或要在地图上面添加标记、绘制图形等操作，这种方式就不大适合了。这时我们可以使用百度地图 **SDK**，在应用中直接嵌入一个地图功能。下面通过一个小样例演示如何使用百度地图 **SDK**。

**一、申请API Key**

首先我们要到百度地图开放平台上申请个 **API Key**，供我们程序使用。地址：http://lbsyun.baidu.com/

[![原文:Swift - 百度地图SDK的配置和使用（附样例）](https://www.hangge.com/blog_uploads/201609/201609062013064625.png)](https://www.hangge.com/blog/cache/detail_1278.html#)




二、项目配置

**1，修改Info.plist**

由于 **iOS9** 改用更安全的 **https**，为了能够在 **iOS9** 中正常使用地图 **SDK**。需要在"**Info.plist**"中添加如下配置：

```
<``key``>NSAppTransportSecurity</``key``>``<``dict``>``  ``<``key``>NSAllowsArbitraryLoads</``key``>``  ``<``true``/>``</``dict``>
```


**2，下载iOS SDK**
百度地图 **SDK** 是 **framework** 形式，地址：[点击此处下载](http://lbsyun.baidu.com/index.php?title=iossdk/sdkiosdev-download)。

[![原文:Swift - 百度地图SDK的配置和使用（附样例）](https://www.hangge.com/blog_uploads/201609/2016090521194436539.png)](https://www.hangge.com/blog/cache/detail_1278.html#)

**3，根据需要导入 .framework包**
其中 **BaiduMapAPI_Base.framework** 为基础包，必须要导入。其他分包可按需导入。

这里我们只导入 **BaiduMapAPI_Base.framework**、**BaiduMapAPI_Map.framework**。

[![原文:Swift - 百度地图SDK的配置和使用（附样例）](https://www.hangge.com/blog_uploads/201609/2016090620032783952.png)](https://www.hangge.com/blog/cache/detail_1278.html#)

**4，添加所需的第三方openssl库**

我们添加支持HTTPS所需的 **openssl** 静态库：**libssl.a** 和 **libcrypto.a**（这两个在SDK包的 **thirdlibs** 文件夹中）

[![原文:Swift - 百度地图SDK的配置和使用（附样例）](https://www.hangge.com/blog_uploads/201704/2017042616332625955.png)](https://www.hangge.com/blog/cache/detail_1278.html#)

**5，添加相关依赖库**

由于百度地图**SDK** 中提供了定位功能和动画效果，使用的是 **OpenGL** 渲染。所以我们要在 **Project** -> **Build Phases** -> **Link Binary With Libraries**，添加如下几个系统库：

**CoreLocation.framework**、**QuartzCore.framework**、**OpenGLES.framework**、**SystemConfiguration.framework**、**CoreGraphics.framework**、**Security.framework**、**libsqlite3.0.tbd**、**CoreTelephony.framework**、**libstdc++.6.0.9.tbd**

[![原文:Swift - 百度地图SDK的配置和使用（附样例）](https://www.hangge.com/blog_uploads/201609/2016090620050835315.png)](https://www.hangge.com/blog/cache/detail_1278.html#)

**6，引入mapapi.bundle资源文件**

**mapapi.bundle** 中存储了定位、默认大头针标注 **View** 及路线关键点的资源图片，还存储了矢量地图绘制必需的资源文件。
如果我们不需要使用内置的图片显示功能，则可以删除 **bundle** 文件中的 **image** 文件夹。
也可以根据具体需求任意替换或删除该 **bundle** 中 **image** 文件夹的图片文件。

（1）选中工程名，在右键菜单中选择 **Add Files to “工程名”...**

[![原文:Swift - 百度地图SDK的配置和使用（附样例）](https://www.hangge.com/blog_uploads/201609/2016090619060178957.png)](https://www.hangge.com/blog/cache/detail_1278.html#)



（2）选择 **BaiduMapAPI_Map.framework** -> **Resources** -> **mapapi.bundle** 文件，并勾选“**Copy items if needed**”复选框，单击“**Add**”按钮，即可将资源文件添加到工程中。

[![原文:Swift - 百度地图SDK的配置和使用（附样例）](https://www.hangge.com/blog_uploads/201609/2016090619171525967.png)](https://www.hangge.com/blog/cache/detail_1278.html#)

**7，新建桥接头文件**

我们需要在这个头文件中 **import** 需要的库文件，具体内容如下。（关于桥接头文件的配置这里就不多说了）

```objective-c
#import <BaiduMapAPI_Base/BMKBaseComponent.h> //引入base相关所有的头文件
#import <BaiduMapAPI_Map/BMKMapComponent.h> //引入地图功能所有的头文件
#import <BaiduMapAPI_Map/BMKMapView.h> //只引入所需的单个头文件
/****** 下面的几个暂时不需要 ******/
//#import <BaiduMapAPI_Search/BMKSearchComponent.h> //引入检索功能所有的头文件
//#import <BaiduMapAPI_Cloud/BMKCloudSearchComponent.h> //引入云检索功能所有的头文件
//#import <BaiduMapAPI_Location/BMKLocationComponent.h> //引入定位功能所有的头文件
//#import <BaiduMapAPI_Utils/BMKUtilsComponent.h> //引入计算工具所有的头文件
//#import <BaiduMapAPI_Radar/BMKRadarComponent.h> //引入周边雷达功能所有的头文件
```

**8，info.plist中添加Bundle display name**
如果没有添加的话，程序启动会报“info.plist 中必须添加 Bundle display name”错误。

[![原文:Swift - 百度地图SDK的配置和使用（附样例）](https://www.hangge.com/blog_uploads/201704/2017042617235084933.png)](https://www.hangge.com/blog/cache/detail_1278.html#)



**三、样例效果图**
1，当程序启动后自动显示一个全屏的百度地图。地图中心点坐标设为上海东方明珠塔。
2，同时在地图上还添加了一个标注（自定义坐标）。

[![原文:Swift - 百度地图SDK的配置和使用（附样例）](https://www.hangge.com/blog_uploads/201609/2016090621034765422.png)](https://www.hangge.com/blog/cache/detail_1278.html#)


**
**

**四、样例代码**
**1，AppDelegate.swift**
在这里使用我们之前申请的授权 **Key**，来对 **BMKMapManager** 进行声明和初始化。

```swift
import UIKit
 
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate, BMKGeneralDelegate {
 
    var window: UIWindow?
     
    var _mapManager: BMKMapManager?
 
    func application(application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
         
        _mapManager = BMKMapManager()
        // 如果要关注网络及授权验证事件，请设定generalDelegate参数
        let ret = _mapManager?.start("mRYTrHzAZUlT5Aojf5tMSxxxxxxxxxxx",
                                     generalDelegate: self)
        if ret == false {
            NSLog("manager start failed!")
        }
        return true
    }
    //...............
```

**
2，ViewController.swift**
在这里我们创建添加百度地图 **View**，并设置显示区域以及标记点。

```swift
import UIKit
 
class ViewController: UIViewController {
 
    //百度地图View
    var _mapView: BMKMapView?
     
    override func viewDidLoad() {
        super.viewDidLoad()
        //添加百度地图，并全屏显示
        _mapView = BMKMapView(frame: CGRect(x: 0, y: 0,
            width: self.view.frame.width,
            height: self.view.frame.height))
        self.view.addSubview(_mapView!)
    }
     
    override func viewDidAppear(_ animated: Bool) {
        //地图中心点坐标
        let center = CLLocationCoordinate2D(latitude: 31.245087, longitude: 121.506656)
        //设置地图的显示范围（越小越精确）
        let span = BMKCoordinateSpan(latitudeDelta: 0.05, longitudeDelta: 0.05)
        //设置地图最终显示区域
        let region = BMKCoordinateRegion(center: center, span: span)
        _mapView?.region = region
         
        // 添加一个标记点(PointAnnotation）
        let annotation =  BMKPointAnnotation()
        var coor = CLLocationCoordinate2D()
        coor.latitude = 31.254087
        coor.longitude = 121.512656
        annotation.coordinate = coor
        annotation.title = "这里有只野生皮卡丘"
        _mapView!.addAnnotation(annotation)
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1278.html