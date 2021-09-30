# Swift - JPush极光推送的使用5（发送通知时附带自定义参数）

2016-07-15发布：hangge阅读：5565

**1，推送消息时添加附加字段**

在之前的几篇文章中介绍了如何发送通知（给所有人发送广播，或者通过 **alias**、**tag** 给指定的设备发送通知）。对于发送的内容，前面的样例都是使用简单的文本消息，然后客户端那边显示出来就好了。其实发送通知时，除了有推送内容，我们还可以附加上一些自定义的扩展字段（自定义的 **Key**/**Value** 信息），以供业务使用。

比如，服务器给客户端推送一条新闻，除了用于通知显示的新闻标题外，还可以附上这条新闻的id或是url（这个用户是看不见的）。这样设备点击通知进入**App** 后，**App** 就可以根据传过来的id或url来自动打开这条新闻。

**2，样例说明**

（1）服务端页面上除了填写新闻标题外（作为推送内容），还可以填写新闻编号和时间（作为附加字段）。

[![原文:Swift - JPush极光推送的使用5（发送通知时附带自定义参数）](https://www.hangge.com/blog_uploads/201607/2016071413584533291.png)](https://www.hangge.com/blog/cache/detail_1274.html#)

[![原文:Swift - JPush极光推送的使用5（发送通知时附带自定义参数）](https://www.hangge.com/blog_uploads/201607/201607141359019260.png)](https://www.hangge.com/blog/cache/detail_1274.html#)



（2）通知推送后，客户端这边收到通知。显示新闻标题。

[![原文:Swift - JPush极光推送的使用5（发送通知时附带自定义参数）](https://www.hangge.com/blog_uploads/201607/2016071414145457530.png)](https://www.hangge.com/blog/cache/detail_1274.html#)

（3）点击通知打开App，App获取附加字段进行下一步操作。（这里就直接弹出显示）

[![原文:Swift - JPush极光推送的使用5（发送通知时附带自定义参数）](https://www.hangge.com/blog_uploads/201607/2016071414152397822.png)](https://www.hangge.com/blog/cache/detail_1274.html#)



**3，完整代码**

（1）客户端：**AppDelegate.swift**（本文代码已升级至 **Swift3**）

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
         
        print(userInfo)
        //获取通知消息和自定义参数数据
        let id = userInfo["id"] as! String
        let time = userInfo["time"] as! String
        let aps = userInfo["aps"] as! NSDictionary
        let alert = aps["alert"] as! String
         
        //显示获取到的数据
        let alertController = UIAlertController(title: alert,
                                                message: "新闻编号：\(id)\n发布时间：\(time)",
                                                preferredStyle: .alert)
        alertController.addAction(UIAlertAction(title: "确定", style: .cancel, handler: nil))
        self.window?.rootViewController!.present(alertController, animated: true, completion: nil)
    }
     
    func application(_ application: UIApplication,
                     didFailToRegisterForRemoteNotificationsWithError error: Error) {
        //可选
        NSLog("did Fail To Register For Remote Notifications With Error: \(error)")
    }
 
    //....................
```

点击通知进入App后，控制台输出如下： 

[![原文:Swift - JPush极光推送的使用5（发送通知时附带自定义参数）](https://www.hangge.com/blog_uploads/201607/2016071414235371839.png)](https://www.hangge.com/blog/cache/detail_1274.html#)


（2）服务端：**index.php**

```php+HTML
<?
//引入代码
require 'JPush/autoload.php';
 
use JPush\Client as JPush;
 
if(isset($_POST["message"])){
  //初始化
  $app_key = "7b528333683ec71919553648fd";
  $master_secret = "32d3be2c06dc734a22c9828";
  $client = new JPush($app_key, $master_secret);
 
  //将附加字段
  $extras = array("id"=>$_POST["id"], "time"=>$_POST["time"]);
 
  //简单的推送样例
  $result = $client->push()
      ->setPlatform('ios', 'android')
      ->addAllAudience()
      ->iosNotification($_POST["message"], [
        'sound' => 'sound',
        'badge' => '+1',
        'extras' => $extras
      ])
      ->androidNotification($_POST["message"], [
        'title' => 'title',
        'extras' => $extras
      ])
      ->options(array(
          "apns_production" => true  //true表示发送到生产环境(默认值)，false为开发环境
        ))
      ->send();
 
  echo 'Result=' . json_encode($result);
}
?>
<html>
  <head>
  </head>
  <body>
    <form action="index.php" method="post">
      消息：<input type="text" name="message"/><br>
      编号：<input type="text" name="id"/><br>
      时间：<input type="text" name="time"/>
      <button type="submit">推送</button>
    </form>
  </body>
</html>
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1274.html