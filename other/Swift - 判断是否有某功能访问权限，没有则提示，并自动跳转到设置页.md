# Swift - 判断是否有某功能访问权限，没有则提示，并自动跳转到设置页

2017-03-03发布：hangge阅读：4136

由于 **iOS** 系统的安全限制，**App** 如果需要访问设备的通讯录、麦克风、 相册、 相机、地理位置等时，需要请求用户是否允许访问。

[![原文:Swift - 判断是否有某功能访问权限，没有则提示，并自动跳转到设置页](https://www.hangge.com/blog_uploads/201701/2017010916565956896.png)](https://www.hangge.com/blog/cache/detail_1517.html#)

有时用户不小心点了“**不允许**”，后面可能就不知道要去哪里再开启这个权限了。这就要求我们应用在每次调用相关功能的时候先获取相关的授权状态，如果还没授权则弹出授权申请的提示框。如果之前被拒绝了，则弹出相关提示框让用户很方便地自动跳转到设置页面去修改权限。

### 1，样例效果图

（1）这里以照片的访问权限为例。为方便演示，我在页面初始化完毕后就请求权限。

（2）第一次请求的时候我们选择“**不允许**”。再次启动程序时，会提示用户照片访问受限，需要授权。

（3）点击“设置”按钮后便自动跳转到系统设置页面。
[![原文:Swift - 判断是否有某功能访问权限，没有则提示，并自动跳转到设置页](https://www.hangge.com/blog_uploads/201701/2017010917082754509.png)](https://www.hangge.com/blog/cache/detail_1517.html#)[![原文:Swift - 判断是否有某功能访问权限，没有则提示，并自动跳转到设置页](https://www.hangge.com/blog_uploads/201701/2017010917083243282.png)](https://www.hangge.com/blog/cache/detail_1517.html#)



### 2，样例代码

```swift
import UIKit
import Photos
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        _ = authorize()
    }
     
    func authorize()->Bool{
        let status = PHPhotoLibrary.authorizationStatus()
         
        switch status {
        case .authorized:
            return true
             
        case .notDetermined:
            // 请求授权
            PHPhotoLibrary.requestAuthorization({ (status) -> Void in
                DispatchQueue.main.async(execute: { () -> Void in
                    _ = self.authorize()
                })
            })
             
        default: ()
        DispatchQueue.main.async(execute: { () -> Void in
            let alertController = UIAlertController(title: "照片访问受限",
                                                    message: "点击“设置”，允许访问您的照片",
                                                    preferredStyle: .alert)
             
            let cancelAction = UIAlertAction(title:"取消", style: .cancel, handler:nil)
             
            let settingsAction = UIAlertAction(title:"设置", style: .default, handler: {
                (action) -> Void in
                let url = URL(string: UIApplicationOpenSettingsURLString)
                if let url = url, UIApplication.shared.canOpenURL(url) {
                    if #available(iOS 10, *) {
                        UIApplication.shared.open(url, options: [:],
                                                  completionHandler: {
                                                    (success) in
                        })
                    } else {
                       UIApplication.shared.openURL(url)
                    }
                }
            })
             
            alertController.addAction(cancelAction)
            alertController.addAction(settingsAction)
             
            self.present(alertController, animated: true, completion: nil)
        })
        }
        return false
    }
  
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



## 附：麦克风权限申请

[![原文:Swift - 判断是否有某功能访问权限，没有则提示，并自动跳转到设置页](https://www.hangge.com/blog_uploads/201802/201802021618271057.png)](https://www.hangge.com/blog/cache/detail_1517.html#)

```swift
import UIKit
import Photos
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        _ = authorize()
    }
     
    func authorize()->Bool{
        let status = AVCaptureDevice.authorizationStatus(forMediaType: AVMediaTypeAudio)
         
        switch status {
        case .authorized:
            return true
             
        case .notDetermined:
            // 请求授权
            AVCaptureDevice.requestAccess(forMediaType: AVMediaTypeAudio, completionHandler: {
                (status) in
                DispatchQueue.main.async(execute: { () -> Void in
                    _ = self.authorize()
                })
            })           
        default: ()
        DispatchQueue.main.async(execute: { () -> Void in
            let alertController = UIAlertController(title: "麦克风访问受限",
                                                    message: "点击“设置”，允许访问您的麦克风",
                                                    preferredStyle: .alert)
             
            let cancelAction = UIAlertAction(title:"取消", style: .cancel, handler:nil)
             
            let settingsAction = UIAlertAction(title:"设置", style: .default, handler: {
                (action) -> Void in
                let url = URL(string: UIApplicationOpenSettingsURLString)
                if let url = url, UIApplication.shared.canOpenURL(url) {
                    if #available(iOS 10, *) {
                        UIApplication.shared.open(url, options: [:],
                                                  completionHandler: {
                                                    (success) in
                        })
                    } else {
                        UIApplication.shared.openURL(url)
                    }
                }
            })
             
            alertController.addAction(cancelAction)
            alertController.addAction(settingsAction)
             
            self.present(alertController, animated: true, completion: nil)
        })
        }
        return false
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1517.html