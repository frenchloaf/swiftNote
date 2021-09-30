# Swift - 改变UIImage图片的大小尺寸，或按比例缩放

2016-08-22发布：hangge阅读：12648

在开发中，我们有时候需要对原始的 **UIImage** 进行处理，比如修改大小或者进行缩放操作。

[![原文:Swift - 改变UIImage图片的大小尺寸，或按比例缩放](https://www.hangge.com/blog_uploads/201608/2016082214160230980.png)](https://www.hangge.com/blog/cache/detail_1344.html#)[![原文:Swift - 改变UIImage图片的大小尺寸，或按比例缩放](https://www.hangge.com/blog_uploads/201608/2016082214161420605.png)](https://www.hangge.com/blog/cache/detail_1344.html#)[![原文:Swift - 改变UIImage图片的大小尺寸，或按比例缩放](https://www.hangge.com/blog_uploads/201608/2016082214162049146.png)](https://www.hangge.com/blog/cache/detail_1344.html#)

**1，扩展UIImage**

这里先对 **UIImage** 进行扩展，增加两个方法，分别用于尺寸的重置和大小缩放。

```swift
import UIKit
 
extension UIImage {
    /**
     *  重设图片大小
     */
    func reSizeImage(reSize:CGSize)->UIImage {
        //UIGraphicsBeginImageContext(reSize);
        UIGraphicsBeginImageContextWithOptions(reSize,false,UIScreen.mainScreen().scale);
        self.drawInRect(CGRectMake(0, 0, reSize.width, reSize.height));
        let reSizeImage:UIImage = UIGraphicsGetImageFromCurrentImageContext();
        UIGraphicsEndImageContext();
        return reSizeImage;
    }
     
    /**
     *  等比率缩放
     */
    func scaleImage(scaleSize:CGFloat)->UIImage {
        let reSize = CGSizeMake(self.size.width * scaleSize, self.size.height * scaleSize)
        return reSizeImage(reSize)
    }
}
```



**2，使用样例**

```swift
import UIKit
 
class ViewController: UIViewController {
     
    let image = UIImage(named:"img.jpg")
 
    @IBOutlet weak var imageView: UIImageView!
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
 
    //显示原始图片
    @IBAction func btn1Click(sender: AnyObject) {
        imageView.image = image
    }
     
    //将图片修改成指定尺寸（160*100）
    @IBAction func btn2Click(sender: AnyObject) {
        let reSize = CGSize(width: 240, height: 150)
        imageView.image = image?.reSizeImage(reSize)
    }
     
    //将图片缩小成原来的一半
    @IBAction func btn3Click(sender: AnyObject) {
        imageView.image = image?.scaleImage(0.5)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1344.zip](https://www.hangge.com/blog_uploads/201608/2016082214191217474.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1344.html