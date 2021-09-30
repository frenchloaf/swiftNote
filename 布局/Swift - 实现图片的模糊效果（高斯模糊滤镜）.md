# Swift - 实现图片的模糊效果（高斯模糊滤镜）

2016-11-07发布：hangge阅读：4249

我之前写过一篇文章演示如何实现毛玻璃效果：[Swift - 实现毛玻璃效果（Blur、模糊、虚化背景元素）](https://www.hangge.com/blog/cache/detail_1135.html)。原理是通过添加一个模糊视图，使得其覆盖下的所有视图都会有模糊效果。

本文介绍另一种实现图片模糊的方法，即使用高斯模糊滤镜。

### 1，效果图

通过滑动滑块，设置不同的模糊半径来实现不同的模糊程度。

[![原文:Swift - 实现图片的模糊效果（高斯模糊滤镜）](https://www.hangge.com/blog_uploads/201611/2016110510345749213.png)](https://www.hangge.com/blog/cache/detail_1424.html#)[![原文:Swift - 实现图片的模糊效果（高斯模糊滤镜）](https://www.hangge.com/blog_uploads/201611/2016110510350380392.png)](https://www.hangge.com/blog/cache/detail_1424.html#)[![原文:Swift - 实现图片的模糊效果（高斯模糊滤镜）](https://www.hangge.com/blog_uploads/201611/2016110510350955458.png)](https://www.hangge.com/blog/cache/detail_1424.html#)

### 2，样例代码

```swift
import UIKit
 
class ViewController: UIViewController {
     
    @IBOutlet weak var imageView: UIImageView!
     
    @IBOutlet weak var slider: UISlider!
     
    //原图
    lazy var originalImage: UIImage = {
        return UIImage(named: "image1.jpg")
        }()!
     
    lazy var context: CIContext = {
        return CIContext(options: nil)
    }()
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    //滑块拖动后
    @IBAction func sliderValueChanged(_ sender: AnyObject) {
        //获取原始图片
        let inputImage =  CIImage(image: originalImage)
        //使用高斯模糊滤镜
        let filter = CIFilter(name: "CIGaussianBlur")!
        filter.setValue(inputImage, forKey:kCIInputImageKey)
        //设置模糊半径值（越大越模糊）
        filter.setValue(slider.value, forKey: kCIInputRadiusKey)
        let outputCIImage = filter.outputImage!
        let rect = CGRect(origin: CGPoint.zero, size: originalImage.size)
        let cgImage = context.createCGImage(outputCIImage, from: rect)
        //显示生成的模糊图片
        imageView.image = UIImage(cgImage: cgImage!)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1424.zip](https://www.hangge.com/blog_uploads/201611/2016110510280228170.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1424.html