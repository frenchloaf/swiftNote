# Swift - 启动画面放大淡出效果的实现1（使用launch image）

2016-06-26发布：hangge阅读：2572

（本文代码已升级至Swift3）

在之前的文章中：[Swift - 延长启动图片的显示时间（LaunchImage）](https://www.hangge.com/blog/cache/detail_1238.html)。介绍了通过在 **viewDidLoad** 方法中添加个线程休眠，可以延长启动图片的显示时间。但时间一到，整个启动页面就会直接消失，略显生硬。

本文演示如何让启动画面消失的时候有动画效果。

**1，效果图如下：**

启动页面会放大淡出直至消失，整个过渡渐变更加自然。

 [![原文:Swift - 启动画面放大淡出效果的实现1（使用launch image）](https://www.hangge.com/blog_uploads/201606/2016062310224843442.png)](https://www.hangge.com/blog/cache/detail_1246.html#)  [![原文:Swift - 启动画面放大淡出效果的实现1（使用launch image）](https://www.hangge.com/blog_uploads/201606/2016062310225563087.png)](https://www.hangge.com/blog/cache/detail_1246.html#)  [![原文:Swift - 启动画面放大淡出效果的实现1（使用launch image）](https://www.hangge.com/blog_uploads/201606/2016062310230254298.png)](https://www.hangge.com/blog/cache/detail_1246.html#)

**2，实现原理：**

我们知道启动页通常有两种方式实现。一种是使用 **LaunchImage** 来设置，另一种是使用 **LaunchScreen.storyboard**。

不管哪种方式，我们都不能直接在它上面做动画。我们可以在程序载入后，往页面上再添加一个同启动页一样的视图。由于内容一样，时间又衔接在一起，用户看不出其实启动页面已经被替换掉。

最后，我们在新添加的视图上做动画即可。

**3，本文先介绍使用LaunchImage实现启动图片的情况：**

下图可以看出，我们已经在 **Assets.xcassets** 文件中添加了适应各种设备的启动图片。

[![原文:Swift - 启动画面放大淡出效果的实现1（使用launch image）](https://www.hangge.com/blog_uploads/201606/2016062310255359954.png)](https://www.hangge.com/blog/cache/detail_1246.html#)

下面是完整代码。关键在于当程序启动后，我们要根据设备的尺寸和方向，自动获取相应的启动图片来显示。

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
        let statusBarOrientation = UIApplication.shared.statusBarOrientation
        if let img = splashImageForOrientation(statusBarOrientation,
                                               size: self.view.bounds.size) {
            //获取启动图片
            let launchImage = UIImage(named: img)
            let launchview = UIImageView(frame: UIScreen.main.bounds)
            launchview.image = launchImage
            //将图片添加到视图上（分两种情况）
            //情况1:没有导航栏
            //self.view.addSubview(launchview)
            //情况2:有导航栏
            let delegate = UIApplication.shared.delegate
            let mainWindow = delegate?.window
            mainWindow!!.addSubview(launchview)
             
            //播放动画效果，完毕后将其移除
            UIView.animate(withDuration: 1, delay: 1.5, options: .beginFromCurrentState,
                           animations: {
                            launchview.alpha = 0.0
                            launchview.layer.transform = CATransform3DScale(
                                CATransform3DIdentity, 1.5, 1.5, 1.0)
            }) { (finished) in
                launchview.removeFromSuperview()
            }
        }
    }
     
    //获取启动图片名（根据设备方向和尺寸）
    func splashImageForOrientation(_ orientation: UIInterfaceOrientation, size: CGSize)
        -> String?{
        //获取设备尺寸和方向
        var viewSize = size
        var viewOrientation = "Portrait"
         
        if UIInterfaceOrientationIsLandscape(orientation) {
            viewSize = CGSize(width: size.height, height: size.width)
            viewOrientation = "Landscape"
        }
         
        //遍历资源库中的所有启动图片，找出符合条件的
        if let imagesDict = Bundle.main.infoDictionary  {
            if let imagesArray = imagesDict["UILaunchImages"] as? [[String: String]] {
                for dict in imagesArray {
                    if let sizeString = dict["UILaunchImageSize"],
                        let imageOrientation = dict["UILaunchImageOrientation"] {
                        let imageSize = CGSizeFromString(sizeString)
                        if imageSize.equalTo(viewSize)
                            && viewOrientation == imageOrientation {
                            if let imageName = dict["UILaunchImageName"] {
                                return imageName
                            }
                        }
                    }
                }
            }
        }
         
        return nil
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}

```

源码下载：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1246.zip](https://www.hangge.com/blog_uploads/201612/2016122510350023466.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1246.html