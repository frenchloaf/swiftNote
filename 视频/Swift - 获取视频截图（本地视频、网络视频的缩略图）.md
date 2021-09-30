# Swift - 获取视频截图（本地视频、网络视频的缩略图）

2016-06-08发布：hangge阅读：3021

有时我们需要在界面上显示视频的缩略图，这样用户不用点开也能大概了解到视频的内容。下面分别演示如何获取本地视频，以及网络在线视频的视频截图。

样例的效果图如下，将获取到的截图（视频开始部分）显示在 **imageView** 中。

[![原文:Swift - 获取视频截图（本地视频、网络视频的缩略图）](https://www.hangge.com/blog_uploads/201605/2016051915391396248.png)](https://www.hangge.com/blog/cache/detail_1194.html#)


**1，获取本地视频截图**

```swift
import UIKit
import AVFoundation
import MobileCoreServices
 
class ViewController: UIViewController {
 
    @IBOutlet weak var imageView: UIImageView!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //获取本地视频
        let filePath = NSBundle.mainBundle().pathForResource("hangge", ofType: "m4v")
        let videoURL = NSURL(fileURLWithPath: filePath!)
        let avAsset = AVAsset(URL: videoURL)
         
        //生成视频截图
        let generator = AVAssetImageGenerator(asset: avAsset)
        generator.appliesPreferredTrackTransform = true
        let time = CMTimeMakeWithSeconds(0.0,600)
        var actualTime:CMTime = CMTimeMake(0,0)
        let imageRef:CGImageRef = try! generator.copyCGImageAtTime(time, actualTime: &actualTime)
        let frameImg = UIImage(CGImage: imageRef)
         
        //显示截图
        self.imageView.image = frameImg
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


**2，获取网络视频截图**

由于网络请求比较耗时，所以我们把获取在线视频的相关代码写在异步线程里。

```swift
import UIKit
import AVFoundation
import MobileCoreServices
 
class ViewController: UIViewController {
     
    @IBOutlet weak var imageView: UIImageView!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
         
        //异步获取网络视频
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0), {
            //获取网络视频
            let url = "http://www.hangge.com/hangge.mp4"
            let videoURL = NSURL(string: url)!
            let avAsset = AVURLAsset(URL: videoURL)
             
            //生成视频截图
            let generator = AVAssetImageGenerator(asset: avAsset)
            generator.appliesPreferredTrackTransform = true
            let time = CMTimeMakeWithSeconds(0.0,600)
            var actualTime:CMTime = CMTimeMake(0,0)
            let imageRef:CGImageRef = try! generator.copyCGImageAtTime(time, actualTime: &actualTime)
            let frameImg = UIImage(CGImage: imageRef)
             
            //在主线程中显示截图
            dispatch_async(dispatch_get_main_queue(), {
                self.imageView.image = frameImg
            })
        })
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1194.html