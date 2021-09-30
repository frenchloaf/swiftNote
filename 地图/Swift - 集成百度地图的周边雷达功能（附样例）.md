# Swift - 集成百度地图的周边雷达功能（附样例）

2016-09-09发布：hangge阅读：1810

**一、周边雷达功能介绍**

**1，什么是周边雷达**

（1）周边雷达功能，是面向移动端开发者的一套 **SDK** 功能接口（同步支持 **Android** 和 **iOS** 端）。

（2）周边雷达本质是一个连接百度 **LBS** 开放平台前端 **SDK** 产品和后端 **LBS** 云的中间服务。

[![原文:Swift - 集成百度地图的周边雷达功能（附样例）](https://www.hangge.com/blog_uploads/201609/2016090717142865905.jpg)](https://www.hangge.com/blog/cache/detail_1279.html#)

**2，使用场景**

利用周边雷达功能，我们可以便捷的在自己的应用内，帮助用户实现查找周边跟“我”使用同一款 **App** 的人。比如：

（1）查看周边都有谁跟“我”使用同一个 **App**，分布在哪里？

（2）查看周边用户在听什么歌、看什么文章、有什么新动态？

（3）查看周边有什么最新发生的新闻、资讯？


**
**

**二、准备工作** 

**1，申请API Key**
首先我们要到百度地图开放平台上申请个 **API Key**，供我们程序使用。具体参考我前面一篇文章：[Swift - 百度地图SDK的配置和使用（附样例） ](https://www.hangge.com/blog/cache/detail_1278.html)


**2，注册周边雷达**
要使用周边雷达功能，我们光申请个 **API Key** 还不够。还需要对这个Key做相应的注册操作。
注册周边雷达是使用其相应功能的基础前提。通过注册可实现一个或多个应用之间的关系绑定，实现相互之间的位置信息查看。 

注册地址：http://developer.baidu.com/map/index.php?title=radar

[![原文:Swift - 集成百度地图的周边雷达功能（附样例）](https://www.hangge.com/blog_uploads/201609/2016090720112495378.png)](https://www.hangge.com/blog/cache/detail_1279.html#)


**3，集成SDK**
周边雷达是百度地图**SDK**产品的一个功能模块。我们除了要将百度地图**SDK**的基础包集成进来外，还要集成周边雷达相关库（**BaiduMapAPI_Radar.framework**）。具体集成方式同样参考[前文](https://www.hangge.com/blog/cache/detail_1278.html)。

[![原文:Swift - 集成百度地图的周边雷达功能（附样例）](https://www.hangge.com/blog_uploads/201609/2016090720040996662.png)](https://www.hangge.com/blog/cache/detail_1279.html#)

同时桥接头文件中也要将雷达库文件 **import** 进来。

```
#``import` `<``BaiduMapAPI_Base``/``BMKBaseComponent``.h> ``//引入base相关所有的头文件``#``import` `<``BaiduMapAPI_Map``/``BMKMapComponent``.h> ``//引入地图功能所有的头文件``#``import` `<``BaiduMapAPI_Map``/``BMKMapView``.h> ``//只引入所需的单个头文件X``#``import` `<``BaiduMapAPI_Radar``/``BMKRadarComponent``.h> ``//引入周边雷达功能所有的头文件``/****** 下面的几个暂时不需要 ******/``//#import <BaiduMapAPI_Search/BMKSearchComponent.h> //引入检索功能所有的头文件``//#import <BaiduMapAPI_Cloud/BMKCloudSearchComponent.h> //引入云检索功能所有的头文件``//#import <BaiduMapAPI_Location/BMKLocationComponent.h> //引入定位功能所有的头文件``//#import <BaiduMapAPI_Utils/BMKUtilsComponent.h> //引入计算工具所有的头文件
```


**三、样例介绍**
下面通过“附近的人”这个小样例演示周边雷达的使用。
（1）程序启动后，应用会自动定时上传用户位置信息（每隔5秒）
（2）上传位置的时候我们同时还附带上传用户名（随机生成的）
（3）主界面有个“附近的人”列表，显示500米范围内其他的用户，以及离你的距离（列表也是5秒钟刷新一次）。
（4）只用一台手机不太好看出效果。拿两台或以上数量的手机测试的话，就可以看到其他人信息，以及他们离你的距离。比如我这里用3部手机测试，其中1部手机的运行效果如下：

[![原文:Swift - 集成百度地图的周边雷达功能（附样例）](https://www.hangge.com/blog_uploads/201609/2016090817222485689.png)](https://www.hangge.com/blog/cache/detail_1279.html#)


**
**

**四、样例代码** 

**1，info.plist**

由于我们要使用 **CoreLocation** 获取实时位置数据，所以在 **info.plist** 里加入定位描述（**Value** 值为空也可以）：

Privacy - Location When In Use Usage Description ：允许在前台获取GPS的描述 

Privacy - Location Always Usage Description ：允许在后台获取GPS的描述 

[![原文:Swift - 集成百度地图的周边雷达功能（附样例）](https://www.hangge.com/blog_uploads/201704/201704261726382845.png)](https://www.hangge.com/blog/cache/detail_1279.html#)

**2，AppDelegate.swift**
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
3，ViewController.swift**
位置信息上传、周边位置检索并显示等功能都在这里实现。这里要注意的是在上传和获取位置信息前，我们需要设置 **userid**。

如果我们不设置的话，这个也会自动生成的。自动生成的**userid** 是唯一的，且与当前设备有关。也就是说对于同一个设备每次启动程序后，生成的 **userid** 都是一样的。（所以多次关闭启动程序不会造成不断新增新用户的问题）

```swift
import UIKit
import CoreLocation
 
class ViewController: UIViewController, CLLocationManagerDelegate, BMKRadarManagerDelegate,
UITableViewDelegate, UITableViewDataSource {
     
    //定位管理器
    let locationManager:CLLocationManager = CLLocationManager()
     
    //当前位置坐标
    var currCoordinate:CLLocationCoordinate2D?
     
    //周边雷达管理类
    var _radarManager:BMKRadarManager?
     
    //定时器，用于定时请求周边信息数据
    var timer:Timer!
     
    //周边用户集合
    var infoList:[BMKRadarNearbyInfo] = [BMKRadarNearbyInfo]()
     
    //显示周边用户的表格
    var tableView:UITableView?
     
    //当前用户名称
    var userName:String!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //随机生成一个用户名
        userName = "陌生人\(String(format: "%03i", Int(arc4random()%1000)))"
        self.title = "\(userName!)，你好！下面是你周围的人。"
         
        //设置定位服务管理器代理
        locationManager.delegate = self
        //设置定位进度
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        //更新距离
        locationManager.distanceFilter = 100
        ////发送授权申请
        locationManager.requestAlwaysAuthorization()
        if (CLLocationManager.locationServicesEnabled()){
            //允许使用定位服务的话，开启定位服务更新
            locationManager.startUpdatingLocation()
            print("定位开始")
        }
         
        //获取周边雷达管理类实例
        _radarManager =  BMKRadarManager.getInstance()
        //在上传和拉取位置信息前，需要设置userid，否则会自动生成
        //_radarManager?.userId = "u007"
        //通过添加radar delegate获取自动上传时的位置信息，以及获得雷达操作结果
        _radarManager?.add(self)
         
        //位置信息连续自动上传（每隔5秒上传一次）
        _radarManager?.startAutoUpload(5)
        //启用计时器，控制每5秒执行一次requestNearbyData方法，请求周边信息
        timer = Timer.scheduledTimer(timeInterval: 5, target:self,
                                     selector:#selector(ViewController.requestNearbyData),
                                     userInfo:nil, repeats:true)
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:UITableViewStyle.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(UITableViewCell.self,
                                 forCellReuseIdentifier: "SwiftCell")
        self.view.addSubview(self.tableView!)
    }
     
    //定位改变执行，可以得到新位置、旧位置
    func locationManager(_ manager: CLLocationManager,
                         didUpdateLocations locations: [CLLocation]) {
        //获取最新的坐标
        currCoordinate = locations.last!.coordinate
    }
     
    //获取我的位置信息（自动上传调用）
    func getRadarAutoUploadInfo() -> BMKRadarUploadInfo! {
        if let pt = currCoordinate{
            let myinfo = BMKRadarUploadInfo()
            myinfo.extInfo = userName
            //myinfo.pt = CLLocationCoordinate2DMake(39.916, 116.404)
            myinfo.pt = pt
            return myinfo
        }else{
            return nil
        }
    }
     
    //返回雷达 上传结果
    func onGetRadarUploadResult(_ error: BMKRadarErrorCode) {
        if error == BMK_RADAR_NO_ERROR {
            print("位置上传成功")
        }else{
            print("位置上传失败")
        }
    }
     
    //发起检索请求，获取周边信息
    func requestNearbyData() {
        if let centerPt = currCoordinate {
            let option =  BMKRadarNearbySearchOption()
            option.radius = 500 //检索半径
            option.sortType = BMK_RADAR_SORT_TYPE_DISTANCE_FROM_NEAR_TO_FAR //排序方式
            option.centerPt = centerPt //检索中心点
            //发起检索
             
            let res = _radarManager?.getRadarNearbySearchRequest(option)
            if res! {
                print("周边信息获取成功")
            } else {
                print("周边信息获取失败")
            }
        }
    }
     
    //获取到周边位置信息回调
    func onGetRadarNearbySearch(_ result: BMKRadarNearbyResult!, error: BMKRadarErrorCode)
    {
        if error == BMK_RADAR_NO_ERROR {
            print("周边用户数量：",result.totalNum)
            self.infoList = result.infoList as! [BMKRadarNearbyInfo]
            self.tableView?.reloadData()
        }
    }
     
    //在本例中，只有一个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.infoList.count
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
            let cell =  UITableViewCell(style: UITableViewCellStyle.value1,
                                        reuseIdentifier: "SwiftCell")
            cell.textLabel?.text = self.infoList[indexPath.row].extInfo
            cell.detailTextLabel?.text = "\(self.infoList[indexPath.row].distance)m"
            return cell
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1279.html