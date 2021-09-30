# Swift - JPush极光推送的使用7（发送自定义消息）

2016-07-18发布：hangge阅读：4907

我之前的一系列文章讲的都是发送通知的相关内容。JPush极光推送除了可以推送通知，还可以用来发送自定义消息。本文接着介绍下后者。

**1，发送自定义消息与发送通知的异同**

（1）客户端**App**只有在运行的时候才能收到自定义消息。而通知不同，不管客户端是否在运行都是能够收到推送过来的通知。

（2）发送自定义消息的话不需要通过 **APNs**，但相较于通知，可以发送更多的内容（当然还是有长度限制的）。
（3）虽然**App**退出后就没法收到自定义消息，但 **JPush** 服务器这时会将其保存成离线消息（具体保留时长可以设置）。当**App**启动后，会自动获取到这条离线消息。

（4）由于发送自定义消息不需要通过 **APNs**，且客户端在运行时才能接收。所以比较适合用在在线聊天室、即时通讯等相关应用上。
（5）同发送通知一样，发送自定义消息也可以根据别名、标签发送给指定用户，或者广播的形式发给所有人。 

（6）自定义消息同样可以定时发送。

**2，自定义消息的样例**

（1）服务端页面上填写消息内容，点击“发送”。即可将自定义消息发送到客户端。

[![原文:Swift - JPush极光推送的使用7（发送自定义消息）](https://www.hangge.com/blog_uploads/201607/2016071514144627569.png)](https://www.hangge.com/blog/cache/detail_1277.html#)

[![原文:Swift - JPush极光推送的使用7（发送自定义消息）](https://www.hangge.com/blog_uploads/201607/201607151414585154.png)](https://www.hangge.com/blog/cache/detail_1277.html#)



（2）客户端如果是运行状态，则会接收到并进行下一步处理（这里直接将收到的自定义消息弹出显示。）如果客户端此刻没有在运行，等客户端下次运行时也会收到之前发送的这条信息。

[![原文:Swift - JPush极光推送的使用7（发送自定义消息）](https://www.hangge.com/blog_uploads/201607/2016071514154566020.png)](https://www.hangge.com/blog/cache/detail_1277.html#)


**3，样例代码**
（1）客户端代码：**AppDelegate.swift**

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
         
        //监听自定义消息的接收
        let defaultCenter =  NotificationCenter.default
        defaultCenter.addObserver(self, selector: #selector(networkDidReceiveMessage(notification:)),
                                  name:Notification.Name.jpfNetworkDidReceiveMessage, object: nil)
         
        return true
    }
     
    //收到自定义消息
    func networkDidReceiveMessage(notification:Notification){
        var userInfo =  notification.userInfo!
        //获取推送内容
        let content =  userInfo["content"] as! String
        //获取服务端传递的Extras附加字段，key是自己定义的
        //let extras =  userInfo["extras"]  as! NSDictionary
        //let value1 =  extras["key1"] as! String
         
        //显示获取到的数据
        let alertController = UIAlertController(title: "收到自定义消息",
                                                message: content,
                                                preferredStyle: .alert)
        alertController.addAction(UIAlertAction(title: "确定", style: .cancel, handler: nil))
        self.window?.rootViewController!.present(alertController, animated: true, completion: nil)
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
 
    //...................
```


（2）服务端代码：**index.php**

```php+HTML
<?
//引入代码
require 'JPush/autoload.php';
 
use JPush\Client as JPush;
 
if(isset($_POST["message"])){
  //初始化
  $app_key = "7b528367738ec346395798fd";
  $master_secret = "32da4a2c06dc7b24472c9828";
  $client = new JPush($app_key, $master_secret);
 
  //简单的消息发送样例
  $result = $client->push()
      ->setPlatform('ios', 'android')
      ->addAllAudience()
      ->message($_POST["message"])
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
      消息：<input type="text" name="message"/>
      <button type="submit">发送</button>
    </form>
  </body>
</html>
```


**4，更完整的发送样例**
同推送通知一样，发送自定义消息也可以添加附加字段，以及定时发送。

```php
// 发送自定义消息
$payload = $client->push()
    ->setPlatform('ios', 'android')
    ->addAllAudience()
    ->message($_POST["message"], [
      'title' => 'Hello',
      'content_type' => 'text',
      'extras' => [
       'key1' => 'value1'
      ]
    ])
    ->options(array(
      "apns_production" => true  //true表示发送到生产环境(默认值)，false为开发环境
    ))
    ->build();
 
// 创建在指定时间点触发的定时任务
$response = $client->schedule()->createSingleSchedule("指定时间点的定时任务",
    $payload, array("time"=>"2016-11-01 15:00:00"));
echo 'Result=' . json_encode($response);
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1277.html