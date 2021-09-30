# Swift - 延长启动图片的显示时间（LaunchImage）

2016-06-23发布：hangge阅读：2619

通过添加应用的 **LaunchImage**，可以让App在加载进入主界面之前显示一张启动图片。具体方法可以参考我原来写的这篇文章：[Swift - 设置程序的应用图标和启动界面](https://www.hangge.com/blog/cache/detail_672.html)

[![原文:Swift - 延长启动图片的显示时间（LaunchImage）](https://www.hangge.com/blog_uploads/201606/2016062117043859041.png)](https://www.hangge.com/blog/cache/detail_1238.html#)

**延长启动图片的显示时间：**

启动界面在程序加载完毕后就会立刻消失，所以有时我们会还没怎么看清启动画面，就直接跳过了。
如果想要延长 **LaunchImage** 的显示时间，可以在第一个加载页面中添加如下代码来做个延时。

```swift
import UIKit
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
        
        NSThread.sleepForTimeInterval(3.0) //延长3秒
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1238.html