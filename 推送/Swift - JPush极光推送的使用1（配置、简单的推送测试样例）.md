# Swift - JPush极光推送的使用1（配置、简单的推送测试样例）

2016-07-11发布：hangge阅读：6931

**一，APNs介绍**
（1）**APNs** 是 **Apple Push Notification service** 的简称，中文翻译为：苹果推送通知服务。
（2）**APNs** 允许设备与苹果的推送通知服务器保持常连接状态。当你想发送一个推送通知给某个用户的iPhone上的应用程序时，你可以使用 **APNs** 发送一个推送消息给目标设备上已安装的某个应用程序。
（3）**APNs** 也是 **iOS** 系统里唯一的推送方式。　
**
二， 极光推送（JPush）介绍**
（1）极光推送是一个端到端的推送服务，使得服务器端消息能够及时地推送到终端用户手机上。
（2）推送客户端支持 **Android**, **iOS** 两个平台。（iOS下使用APNs推送）
（3）为 **JPush Server** 上报 **Device Token**，免除开发者管理 **Device Token** 的麻烦。
（4）前台运行时，可接收由 **JPush** 下发的（透传的）自定义消息。
（5）灵活管理接收用户：**Tag**（标签分组）、**Alias**（用户别名）、**RegistrationID**（设备注册ID）

**三，使用极光推送的优点**
虽然我们可以搭建自己应用服务器，将需要被推送的信息发给APNs。接着由APNs推送到指定的iOS设备上，然后再由设备通知到我们的应用程序，最后设备以通知或者声音的形式通知用户有新的消息。
但使用 **JPush SDK** 可以更快捷地为 **iOS App** 增加推送功能，减少集成 **APNs** 需要的工作量、开发复杂度，也让服务器端向 **iOS** 设备推送变得更加简单方便。
**
四，极光推送SDK集成步骤**
**1、在JPush Portal上创建应用
**在 **JPush** 的管理 **Portal** 上创建应用并上传 **APNs** 证书。如果对 **APNs** 证书不太了解 请参考： [iOS 证书设置指南](http://docs.jiguang.cn/jpush/client/iOS/ios_cer_guide/)

[![原文:Swift - JPush极光推送的使用1（配置、简单的推送测试样例）](https://www.hangge.com/blog_uploads/201607/2016071011275432336.jpg)](https://www.hangge.com/blog/cache/detail_1268.html#)

创建成功后自动生成 **AppKey** 用以标识该应用。

[![原文:Swift - JPush极光推送的使用1（配置、简单的推送测试样例）](https://www.hangge.com/blog_uploads/201607/2016071011282436995.jpg)](https://www.hangge.com/blog/cache/detail_1268.html#)



**2，导入API开发包到应用程序项目**
将 **iOS SDK** 包解压。下载地址：http://docs.jiguang.cn/jpush/resources/#ios-sdk
然后把解压后的 **lib** 子文件夹（包含 **JPUSHService.h**、**jpush-ios-x.x.x.a**）添加到你的工程目录中。

[![原文:Swift - JPush极光推送的使用1（配置、简单的推送测试样例）](https://www.hangge.com/blog_uploads/201607/2016071011390371014.png)](https://www.hangge.com/blog/cache/detail_1268.html#)



**3，创建并配置个桥接头文件将JPUSHService.h引入进来**

```
#``import` `"JPUSHService.h"
```



**4，添加必要的框架**

（1）**CFNetwork.framework**

（2）**CoreFoundation.framework**

（3）**CoreTelephony.framework**

（4）**SystemConfiguration.framework**

（5）**CoreGraphics.framework**

（6）**Foundation.framework**

（7）**UIKit.framework**

（8）**Security.framework**

（9）**libz.tbd**

（10）**Adsupport.framework** (获取 **IDFA** 需要；如果不使用 **IDFA**，请不要添加)

（11）**libresolv.tbd**（JPush 2.2.0及以上版本需要）

[![原文:Swift - JPush极光推送的使用1（配置、简单的推送测试样例）](https://www.hangge.com/blog_uploads/201610/201610311529426250.png)](https://www.hangge.com/blog/cache/detail_1268.html#)





**5，允许XCode7支持Http传输方法**
如果用的是 **Xcode7** 或者更新的版本时，需要在App项目 **info.plist** 中添加如下配置以支持 **http** 传输。

```swift
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
```


**6，开启Remote notifications**
需要在 **Xcode** 中修改应用的 **Capabilities** 开启 **Remote notifications**

[![原文:Swift - JPush极光推送的使用1（配置、简单的推送测试样例）](https://www.hangge.com/blog_uploads/201607/2016071013055174259.png)](https://www.hangge.com/blog/cache/detail_1268.html#)



**7，开启Push Notifications**

如使用 **Xcode8** 及以上环境开发，还要开启 **Capabilities** 下的 **Push Notifications** 选项。

[![原文:Swift - JPush极光推送的使用1（配置、简单的推送测试样例）](https://www.hangge.com/blog_uploads/201610/2016103115103777247.png)](https://www.hangge.com/blog/cache/detail_1268.html#)

**8，最后在AppDelegate.swift中添加如下代码，启动JPush的SDK服务。** （本文代码已升级至 **Swift3**）

```swift
import UIKit
 
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
     
    var window: UIWindow?
     
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions
                        launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
       
        //通知类型（这里将声音、消息、提醒角标都给加上）
        let userSettings = UIUserNotificationSettings(types: [.alert, .badge, .sound],
                                                      categories: nil)
        if ((UIDevice.current.systemVersion as NSString).floatValue >= 8.0) {
            //可以添加自定义categories
            JPUSHService.register(forRemoteNotificationTypes: userSettings.types.rawValue,
                                                            categories: nil)
        }
        else {
            //categories 必须为nil
            JPUSHService.register(forRemoteNotificationTypes: userSettings.types.rawValue,
                                                            categories: nil)
        }
         
        // 启动JPushSDK
        JPUSHService.setup(withOption: nil, appKey: "7b528331738ec719195798fd",
                                     channel: "Publish Channel", apsForProduction: false)
 
        return true
    }
  
     
    func application(_ application: UIApplication,
                     didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        //注册 DeviceToken
        JPUSHService.registerDeviceToken(deviceToken)
    }
     
    func application(_ application: UIApplication,
                     didReceiveRemoteNotification userInfo: [AnyHashable : Any],
                     fetchCompletionHandler
                        completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
        //增加IOS 7的支持
        JPUSHService.handleRemoteNotification(userInfo)
        completionHandler(UIBackgroundFetchResult.newData)
    }
     
    func application(_ application: UIApplication,
                     didFailToRegisterForRemoteNotificationsWithError error: Error) {
        //可选
        NSLog("did Fail To Register For Remote Notifications With Error: \(error)")
    }
     
    //..........
}
```



> **JPUSHService.setupWithOption()方法的参数说明：**
>
> **channel**
>
> 指明应用程序包的下载渠道，为方便分渠道统计，具体值由你自行定义，如：App Store。
>
> **appKey**
>
> 填写管理Portal上创建应用后自动生成的AppKey值。请确保应用内配置的 AppKey 与第1步在 Portal 上创建应用后生成的 AppKey 一致。
>
> **apsForProduction**
>
> 1.3.1版本新增，用于标识当前应用所使用的APNs证书环境。
>
> 0 (默认值)表示采用的是开发证书，1 表示采用生产证书发布应用。
>
> 注：此字段的值要与Build Settings的Code Signing配置的证书环境一致。


五，推送测试
1，我们将上面配置好的应用编译发布到手机上（使用模拟器无法测试 **JPush** 推送）
2，第一次运行程序会提示我们是否允许推送通知，选择“好”即可。

[![原文:Swift - JPush极光推送的使用1（配置、简单的推送测试样例）](https://www.hangge.com/blog_uploads/201607/2016071021130659150.png)](https://www.hangge.com/blog/cache/detail_1268.html#)

控制台输出消息如下：

[![原文:Swift - JPush极光推送的使用1（配置、简单的推送测试样例）](https://www.hangge.com/blog_uploads/201607/2016071021151020390.png)](https://www.hangge.com/blog/cache/detail_1268.html#)



3，将程序退出。接着我们到极光推送网站上的控制台模块来进行消息发送的测试。

在“推送”->“发送通知”页面中，输入需要推送的消息及相关配置（这里我们选择广播，即给所有人都发生推送）。点击“立即发送”。

[![原文:Swift - JPush极光推送的使用1（配置、简单的推送测试样例）](https://www.hangge.com/blog_uploads/201607/2016071021360341720.png)](https://www.hangge.com/blog/cache/detail_1268.html#)



4，通知发送后，虽然我们的应用之前已经关闭了，但我们可以看到手机上还是会成功显示接收到的通知消息。

[![原文:Swift - JPush极光推送的使用1（配置、简单的推送测试样例）](https://www.hangge.com/blog_uploads/201607/2016071021421628541.png)](https://www.hangge.com/blog/cache/detail_1268.html#)




原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1268.html