# Swift - 启动画面放大淡出效果的实现2（使用LaunchScreen.storyboard）

2016-06-28发布：hangge阅读：3667

在前文中：[Swift - 启动画面放大淡出效果的实现1（使用launch image）](https://www.hangge.com/blog/cache/detail_1246.html)。演示了如何让启动画面消失的时候有动画效果。当时是使用 **launch image** 来实现启动图片。本文演示在使用 **LaunchScreen.storyboard** 作为启动画面的情况下，如何实现同样的过渡效果。

**1，效果图如下：**

启动页面同样会放大淡出直至消失。

  [![原文:Swift - 启动画面放大淡出效果的实现2（使用LaunchScreen.storyboard）](https://www.hangge.com/blog_uploads/201606/2016062314092149509.png)](https://www.hangge.com/blog/cache/detail_1247.html#)  [![原文:Swift - 启动画面放大淡出效果的实现2（使用LaunchScreen.storyboard）](https://www.hangge.com/blog_uploads/201606/2016062314092662606.png)](https://www.hangge.com/blog/cache/detail_1247.html#)  [![原文:Swift - 启动画面放大淡出效果的实现2（使用LaunchScreen.storyboard）](https://www.hangge.com/blog_uploads/201606/2016062314093369241.png)](https://www.hangge.com/blog/cache/detail_1247.html#)

**2，实现原理：**

我们知道启动页通常有两种方式实现。一种是使用 **LaunchImage** 来设置，另一种是使用 **LaunchScreen.storyboard**。

不管哪种方式，我们都不能直接在它上面做动画。我们可以在程序载入后，往页面上再添加一个同启动页一样的视图。由于内容一样，时间又衔接在一起，用户看不出其实启动页面已经被替换掉。

最后，我们在新添加的视图上做动画即可。

**3，本文介绍使用LaunchScreen.storyboard实现启动页的情况：**

（1）App默认情况下，就是以 **LaunchScreen.storyboard** 作为启动界面。作为演示，我们先在上面添加一些组件。

[![原文:Swift - 启动画面放大淡出效果的实现2（使用LaunchScreen.storyboard）](https://www.hangge.com/blog_uploads/201606/2016062314121346258.png)](https://www.hangge.com/blog/cache/detail_1247.html#)


（2）在 **LaunchScreen.storyboard** 中将 **Storyboard ID** 设置为 **launch**

[![原文:Swift - 启动画面放大淡出效果的实现2（使用LaunchScreen.storyboard）](https://www.hangge.com/blog_uploads/201606/2016062314133260757.png)](https://www.hangge.com/blog/cache/detail_1247.html#)

（3）下面是完整代码。关键在于当程序启动后，我们要根据 **Storyboard ID**，将我们应用的启动页获取到并显示。

```swift
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //播放启动画面动画
        launchAnimation()
    }
     
    //播放启动画面动画
    private func launchAnimation() {
        //获取启动视图
        let vc = UIStoryboard(name: "LaunchScreen", bundle: nil)
            .instantiateViewController(withIdentifier: "launch")
        let launchview = vc.view!
        let delegate = UIApplication.shared.delegate
        delegate?.window!!.addSubview(launchview)
        //self.view.addSubview(launchview) //如果没有导航栏，直接添加到当前的view即可
         
        //播放动画效果，完毕后将其移除
        UIView.animate(withDuration: 1, delay: 1.5, options: .beginFromCurrentState,
        animations: {
                launchview.alpha = 0.0
                let transform = CATransform3DScale(CATransform3DIdentity, 1.5, 1.5, 1.0)
                launchview.layer.transform = transform
        }) { (finished) in
            launchview.removeFromSuperview()
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


**4，源码下载**

![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1247.zip](https://www.hangge.com/blog_uploads/201607/2016071514470174821.zip)（2016-07-15 Swift 2）

![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1247.zip](https://www.hangge.com/blog_uploads/201610/2016100419021596130.zip)（2016-10-04 Swift 3 最新版）


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1247.html