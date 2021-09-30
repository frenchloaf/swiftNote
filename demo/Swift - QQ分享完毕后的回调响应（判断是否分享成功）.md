# Swift - QQ分享完毕后的回调响应（判断是否分享成功）

2016-09-22发布：hangge阅读：1751

我之前写过一篇关于调用QQ分享的文章：[Swift - 腾讯官方SDK的配置及使用（分享到QQ空间、分享到好友）](https://www.hangge.com/blog/cache/detail_1070.html)。通过使用腾讯提供的 **SDK**，我们可以从自己的 **App** 跳转到手机 **QQ**，将消息分享给**QQ** 好友或发布到空间上。

   [![原文:Swift - QQ分享完毕后的回调响应（判断是否分享成功）](https://www.hangge.com/blog_uploads/201603/2016030121394499463.png)](https://www.hangge.com/blog/cache/detail_1364.html#)    [![原文:Swift - QQ分享完毕后的回调响应（判断是否分享成功）](https://www.hangge.com/blog_uploads/201603/2016030121395395535.png)](https://www.hangge.com/blog/cache/detail_1364.html#)
    [![原文:Swift - QQ分享完毕后的回调响应（判断是否分享成功）](https://www.hangge.com/blog_uploads/201603/2016030121400154261.png)](https://www.hangge.com/blog/cache/detail_1364.html#)    [![原文:Swift - QQ分享完毕后的回调响应（判断是否分享成功）](https://www.hangge.com/blog_uploads/201603/2016030121401238333.png)](https://www.hangge.com/blog/cache/detail_1364.html#)

通过 **QQApiInterface.send()** 方法发送消息后，根据返回 **QQApiSendResultCode** 类型的结果。我们可以判断 **QQ** 是否调用成功。没调用成功的话还能得到具体错误原因（**API** 不支持、还是 **QQ** 没安装等等）

除了判断调用是否成功，我们有时还会需要知道发送的结果，这样根据用户分享结果进行不同的处理。本文演示如何通过委托响应，捕获消息分享的结果。

**1，样例效果图**

当分享消息完毕，返回应用时，我们应用中会显示是否分享成功的提示。如果分享失败，还会附带上失败原因。

   [![原文:Swift - QQ分享完毕后的回调响应（判断是否分享成功）](https://www.hangge.com/blog_uploads/201609/201609172102287042.png)](https://www.hangge.com/blog/cache/detail_1364.html#)    [![原文:Swift - QQ分享完毕后的回调响应（判断是否分享成功）](https://www.hangge.com/blog_uploads/201609/2016091721024154265.png)](https://www.hangge.com/blog/cache/detail_1364.html#)

**2，样例代码**

（1）**ViewController.swift**

分享相关的代码与前文一样，这里就不再写了。[点击查看详细代码](https://www.hangge.com/blog/cache/detail_1070.html)

（2）**AppDelegate.swift**

我们在 **AppDelegate** 中除了要重写 **open url** 方法，还要添加 **QQApiInterfaceDelegate** 代理，并实现相关的代理方法。这样当消息发送完毕后，我们可以在回调方法中进行需要的操作。

```swift
import UIKit
 
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate, QQApiInterfaceDelegate  {
 
    var window: UIWindow?
     
    //重写openURL
    func application(_ app: UIApplication, open url: URL,
                     options: [UIApplicationOpenURLOptionsKey : Any] = [:]) -> Bool {
        QQApiInterface.handleOpen(url, delegate: self)
        return TencentOAuth.handleOpen(url)
    }
     
    //处理来至QQ的响应
    func onResp(_ resp: QQBaseResp!) {
        print("--- onResp ---")
        //确保是对我们QQ分享操作的回调
        if resp.isKind(of: SendMessageToQQResp.self) {
            //QQApi应答消息类型判断（手Q -> 第三方应用，手Q应答处理分享消息的结果）
            if uint(resp.type) == ESENDMESSAGETOQQRESPTYPE.rawValue {
                let title = resp.result == "0" ? "分享成功" : "分享失败"
                var message = ""
                switch(resp.result){
                case "-1":
                    message = "参数错误"
                case "-2":
                    message = "该群不在自己的群列表里面"
                case "-3":
                    message = "上传图片失败"
                case "-4":
                    message = "用户放弃当前操作"
                case "-5":
                    message = "客户端内部处理错误"
                default: //0
                    //message = "成功"
                    break
                }
 
                //显示消息
                showAlert(title: title, message: message)
            }
        }
    }
     
    //处理来至QQ的请求
    func onReq(_ req:QQBaseReq!){
        print("--- onReq ---")
    }
     
    //处理QQ在线状态的回调
    func isOnlineResponse(_ response: [AnyHashable : Any]!) {
        print("--- isOnlineResponse ---")
    }
     
    //显示消息
    func showAlert(title:String, message:String){
        let alertController = UIAlertController(title: title,
            message: message, preferredStyle: .alert)
        let cancelAction = UIAlertAction(title: "确定", style: .cancel, handler: nil)
        alertController.addAction(cancelAction)
        self.window?.rootViewController!.present(alertController, animated: true,
                                                 completion: nil)
    }
     
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions:
        [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        return true
    }
 
    func applicationWillResignActive(_ application: UIApplication) {
    }
 
    func applicationDidEnterBackground(_ application: UIApplication) {
    }
 
    func applicationWillEnterForeground(_ application: UIApplication) {
    }
 
    func applicationDidBecomeActive(_ application: UIApplication) {
    }
 
    func applicationWillTerminate(_ application: UIApplication) {
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1070.zip](https://www.hangge.com/blog_uploads/201609/2016091721162630285.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1364.html