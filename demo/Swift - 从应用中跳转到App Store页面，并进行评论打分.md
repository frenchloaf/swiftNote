# Swift - 从应用中跳转到App Store页面，并进行评论打分

2016-09-05发布：hangge阅读：3305

（本文代码已升级至Swift3）

我们常常看到市面上有很多 **App**，每用个一段时间就会弹出个框，问你是否要去 **AppStore** 上给它打个分。如果你选择是的话，就会自动打开 **AppStore**，并显示这个应用的首页，用户便可以在这里撰写评论并打分了。

这个其实很简单，只需要通过 **UIApplication.shared.open()** 方法打开相应应用的 **App Store** 链接即可。除了跳转到 **App Store**，**UIApplication.shared.open()**还要其它许多用法，具体参考我原来写的这篇文章：[Swift - 打开第三方应用，并传递参数（附常用App的URL Scheme）](https://www.hangge.com/blog/cache/detail_1141.html)


**1，样例效果图**
当程序启动的时候会弹出消息框询问是否去评价。点击“好的”即跳转到这个应用的 **AppStore** 页面。
[![原文:Swift - 从应用中跳转到App Store页面，并进行评论打分](https://www.hangge.com/blog_uploads/201609/2016090411450165658.png)](https://www.hangge.com/blog/cache/detail_1265.html#)[![原文:Swift - 从应用中跳转到App Store页面，并进行评论打分](https://www.hangge.com/blog_uploads/201609/2016090411451410021.png)](https://www.hangge.com/blog/cache/detail_1265.html#)



**2，样例代码**
链接地址的末尾是你要跳转到的应用的 **appID**，这个是你提交 **app** 时候自动生成的，也是 **AppStore** 中的唯一的 **ID**（作为演示，这里我就使用 **QQ** 的 **appID**）。

```swift
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    override func viewDidAppear(_ animated: Bool) {
        //弹出消息框
        let alertController = UIAlertController(title: "觉得好用的话，给我个评价吧！",
                                                message: nil, preferredStyle: .alert)
        let cancelAction = UIAlertAction(title: "暂不评价", style: .cancel, handler: nil)
        let okAction = UIAlertAction(title: "好的", style: .default,
                                     handler: {
                                        action in
                                        self.gotoAppStore()
        })
        alertController.addAction(cancelAction)
        alertController.addAction(okAction)
        self.present(alertController, animated: true, completion: nil)
    }
     
    //跳转到应用的AppStore页页面
    func gotoAppStore() {
        let urlString = "itms-apps://itunes.apple.com/app/id444934666"
        if let url = URL(string: urlString) {
            //根据iOS系统版本，分别处理
            if #available(iOS 10, *) {
                UIApplication.shared.open(url, options: [:],
                                          completionHandler: {
                                            (success) in
                })
            } else {
                UIApplication.shared.openURL(url)
            }
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1265.html