# Swift - 去处图片的白色、黑色背景（使UIImage背景透明）

2016-12-26发布：hangge阅读：2891

我之前写过一篇关于抠图的文章：[Swift - 抠图，及图片合成功能的实现（适用于纯色背景）](https://www.hangge.com/blog/cache/detail_904.html)。其适用范围是纯色背景，但不能用与背景色是纯白或纯黑的情况。本文演示对于图片背景是白色或黑色的情况下，如何进行抠图。

**1，效果图**

（1）点击“**白底原图**”“**黑底原图**”按钮可以切换显示白底或黑底的图片。

（2）点击“**抠图**”按钮后，将当前选择的图片的背景变成透明，并显示在 **imageView** 上。这里为了看出处理后的图片与原图的区别，我将 **imageView** 的背景色设为淡蓝色。

（3）点击“**抠图并合成**”按钮后，除了将原图的白色或黑色背景去处外，还会将处理后的图片与另一张图片进行合并显示。

  [![原文:Swift - 去处图片的白色、黑色背景（使UIImage背景透明）](https://www.hangge.com/blog_uploads/201612/201612222109148387.png)](https://www.hangge.com/blog/cache/detail_1496.html#)  [![原文:Swift - 去处图片的白色、黑色背景（使UIImage背景透明）](https://www.hangge.com/blog_uploads/201612/2016122221115385063.png)](https://www.hangge.com/blog/cache/detail_1496.html#)  [![原文:Swift - 去处图片的白色、黑色背景（使UIImage背景透明）](https://www.hangge.com/blog_uploads/201612/2016122221120096517.png)](https://www.hangge.com/blog/cache/detail_1496.html#)

  [![原文:Swift - 去处图片的白色、黑色背景（使UIImage背景透明）](https://www.hangge.com/blog_uploads/201612/2016122221121625520.png)](https://www.hangge.com/blog/cache/detail_1496.html#)  [![原文:Swift - 去处图片的白色、黑色背景（使UIImage背景透明）](https://www.hangge.com/blog_uploads/201612/2016122221122972021.png)](https://www.hangge.com/blog/cache/detail_1496.html#)  [![原文:Swift - 去处图片的白色、黑色背景（使UIImage背景透明）](https://www.hangge.com/blog_uploads/201612/2016122221123696475.png)](https://www.hangge.com/blog/cache/detail_1496.html#)

**2，扩展UIImage**

我们这里对 **UIImage** 进行扩展，增加了处理白色背景与黑色背景的方法，调用后自动返回个背景透明的 **UIImage**。

```swift
extension UIImage {
    //返回一个将白色背景变透明的UIImage
    func imageByRemoveWhiteBg() -> UIImage? {
        let colorMasking: [CGFloat] = [222, 255, 222, 255, 222, 255]
        return transparentColor(colorMasking: colorMasking)
    }
     
    //返回一个将黑色背景变透明的UIImage
    func imageByRemoveBlackBg() -> UIImage? {
        let colorMasking: [CGFloat] = [0, 32, 0, 32, 0, 32]
        return transparentColor(colorMasking: colorMasking)
    }
     
    func transparentColor(colorMasking:[CGFloat]) -> UIImage? {
        if let rawImageRef = self.cgImage {
            UIGraphicsBeginImageContext(self.size)
            if let maskedImageRef = rawImageRef.copy(maskingColorComponents: colorMasking) {
                let context: CGContext = UIGraphicsGetCurrentContext()!
                context.translateBy(x: 0.0, y: self.size.height)
                context.scaleBy(x: 1.0, y: -1.0)
                context.draw(maskedImageRef, in: CGRect(x:0, y:0, width:self.size.width,
                                                        height:self.size.height))
                let result = UIGraphicsGetImageFromCurrentImageContext()
                UIGraphicsEndImageContext()
                return result
            }
        }
        return nil
    }
}
```


**3，样例代码**

```swift
import UIKit
 
class ViewController: UIViewController {
     
    @IBOutlet weak var imageView: UIImageView!
     
    //当前选择的图片图片
    var originalImage:UIImage!
    //白底图片
    var image1 = UIImage(named: "flower.png")!
    //黑底图片
    var image2 = UIImage(named: "flower2.png")!
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        originalImage = image1
    }
     
    //原图1
    @IBAction func resetImage1(_ sender: AnyObject) {
        originalImage = image1
        self.imageView.image = originalImage
    }
     
    //原图2
    @IBAction func resetImage2(_ sender: AnyObject) {
        originalImage = image2
        self.imageView.image = originalImage
    }
     
    //抠图
    @IBAction func cutOut(_ sender: AnyObject) {
        if originalImage == image1 {
            imageView.image = originalImage.imageByRemoveWhiteBg()
        }else{
            imageView.image = originalImage.imageByRemoveBlackBg()
        }
    }
     
    //抠图并合成
    @IBAction func cutOutAndCompose(_ sender: AnyObject) {
        //获取背景图
        let bgImage = UIImage(named: "bg.png")!
        //获取处理后的前景图
        var flowerImage:UIImage
        if originalImage == image1 {
            flowerImage = originalImage.imageByRemoveWhiteBg()!
        }else{
            flowerImage = originalImage.imageByRemoveBlackBg()!
        }
         
        //开始图片处理上下文（由于输出的图不会进行缩放，所以缩放因子等于屏幕的scale即可
        UIGraphicsBeginImageContextWithOptions(bgImage.size, false, 0)
        //绘制背景
        bgImage.draw(at: CGPoint.zero)
        //绘制前景
        flowerImage.draw(at: CGPoint.zero)
        let newImage = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
         
        //显示最终图片
        imageView.image = newImage
    }
     
    //还原图片
    @IBAction func resetImage(_ sender: AnyObject) {
        self.imageView.image = originalImage
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

源码下载：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1496.zip](https://www.hangge.com/blog_uploads/201612/2016122221161350240.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1496.html