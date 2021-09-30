# Swift - 隐藏状态栏（statusBar）的两种方法

2017-01-16发布：hangge阅读：9349

默认情况下，程序启动后页面顶部会有一个状态栏（**statusBar**），如下图：

[![原文:Swift - 隐藏状态栏（statusBar）的两种方法](https://www.hangge.com/blog_uploads/201701/201701091614394583.png)](https://www.hangge.com/blog/cache/detail_1518.html#)



如果我们想要去掉状态栏，有两种办法实现。

[![原文:Swift - 隐藏状态栏（statusBar）的两种方法](https://www.hangge.com/blog_uploads/201701/2017010916161612921.png)](https://www.hangge.com/blog/cache/detail_1518.html#)





### 1，全局设置

这种方法修改后，整个应用的所有视图都不显示状态栏。

（1）在 **Info.plist** 中添加如下配置

[![原文:Swift - 隐藏状态栏（statusBar）的两种方法](https://www.hangge.com/blog_uploads/201605/2016050117161517571.png)](https://www.hangge.com/blog/cache/detail_1518.html#)

```xml
<key>UIViewControllerBasedStatusBarAppearance</key>
<false/>
```


（2）在 **General** -> **Deployment Info** 中，将 **Hide status bar** 勾选。

[![原文:Swift - 隐藏状态栏（statusBar）的两种方法](https://www.hangge.com/blog_uploads/201701/2017010916212691058.png)](https://www.hangge.com/blog/cache/detail_1518.html#)

###   2，在视图控制器中单独设置

这用方法适合于只隐藏部分页面的状态栏。我们在需要隐藏 **statusbar** 的 **ViewController** 中添加如下代码即可。

```swift
import UIKit
 
class ViewController: UIViewController {
     
    //隐藏状态栏
    override var prefersStatusBarHidden: Bool {
        return true
    }
  
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1518.html