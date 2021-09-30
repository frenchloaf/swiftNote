# Swift - JPush极光推送的使用6（定时推送通知）

2016-07-17发布：hangge阅读：3660

前面的文章中推送通知我都是立即发送，其实使用 **JPush** 我们还可以通过创建定时任务来实现消息的定时发送。

**1，创建具体时间点的定时任务**

（1）样例说明

服务端页面上我们除了填写通知内容外，还可以指定发送通知的时间。点击“发送”后，客户端不会立刻收到通知。

[![原文:Swift - JPush极光推送的使用6（定时推送通知）](https://www.hangge.com/blog_uploads/201607/201607151005409175.png)](https://www.hangge.com/blog/cache/detail_1276.html#)

[![原文:Swift - JPush极光推送的使用6（定时推送通知）](https://www.hangge.com/blog_uploads/201607/2016071510054979933.png)](https://www.hangge.com/blog/cache/detail_1276.html#)



只有到了指定的的时间，客户端才会收到通知。

[![原文:Swift - JPush极光推送的使用6（定时推送通知）](https://www.hangge.com/blog_uploads/201607/2016071510113621704.png)](https://www.hangge.com/blog/cache/detail_1276.html#)

（2）**index.php**（服务端代码）

```php+HTML
<?
//引入代码
require 'JPush/autoload.php';
 
use JPush\Client as JPush;
 
if(isset($_POST["message"])){
  //初始化
  $app_key = "7b528331738ec719195798fd";
  $master_secret = "32da4e2c06dc7b25da2c9828";
  $client = new JPush($app_key, $master_secret);
 
  //简单的推送样例
  $payload = $client->push()
      ->setPlatform('ios', 'android')
      ->addAllAudience()
      ->setNotificationAlert($_POST["message"])
      ->options(array(
        "apns_production" => true  //true表示发送到生产环境(默认值)，false为开发环境
      ))
      ->build();
 
  // 创建在指定时间点触发的定时任务
  $response = $client->schedule()->createSingleSchedule("指定时间点的定时任务",
      $payload, array("time"=>$_POST["time"]));
  echo 'Result=' . json_encode($response);
}
?>
<html>
  <head>
  </head>
  <body>
    <form action="index.php" method="post">
      通知内容：<input type="text" name="message"/><br>
      发送时间：<input type="text" name="time"/>
      <button type="submit">推送</button>
    </form>
  </body>
</html>
```



（3）**AppDelegate.swift**（客户端代码。这个同之前文章里的一样，没有改变。本文代码已升级至 **Swfit3**）



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
        JPUSHService.setup(withOption: nil, appKey: "7b96331738ea713195698fd",
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



**2，创建循环重复执行的定时任务**
比如下面代码创建一个每天14点发送的定时任务。

```php
// 创建一个每天14点发送的定时任务
$response = $client->schedule()->createPeriodicalSchedule("每天14点发送的定时任务", $payload,
        array(
            "start"=>"2016-01-22 13:45:00",
            "end"=>"2016-12-25 13:45:00",
            "time"=>"14:00:00",
            "time_unit"=>"DAY",
            "frequency"=>1
        ));
echo 'Result=' . json_encode($response);
```


**3，更新指定的定时任务**

```php
$schedule_id = $response["body"]["schedule_id"];
// 更新指定的定时任务
$response = $client->schedule()->updatePeriodicalSchedule($schedule_id, null, true);
echo "Result=" . json_encode($response);
```


**4，获取定时任务列表**

```php
$response = $client->schedule()->getSchedules();
echo "Result=" . json_encode($response);
```


**5，删除定时任务**

```php
$response = $client->schedule()->deleteSchedule($schedule_id);
echo "Result=" . json_encode($response);
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1276.html