# Swift - 自定义UIAlertController的样式2（添加自定义视图）

2017-04-24发布：hangge阅读：8620

在上一篇文章中，我介绍了如何修改 **UIAlertController** 里标题和按钮的文字样式（[点击查看](https://www.hangge.com/blog/cache/detail_1658.html)）。但有时只显示文字内容还不能满足我们的需求，下面介绍如何在 **UIAlertController** 中显示自定义 **View**。

###  1，实现原理

（1）通过 **alertController.view.addSubview** 方法可以将我们要显示的内容控件插入进来。

（2）但 **alertController** 的显示区域大小并不会根据我们插入的自定义 **View** 的尺寸来自适应。因此需要我们将 **title** 设置成多个换行符，人为地将空间给预留出来。

### 2，效果图

（1）消息内容区域改成使用 **UIImageView** 组件显示一张图片。

（2）下面分别展示 **UIAlertController** 在弹出框（**alert**）以及操作表（**actionSheet**）这两种样式下的显示情况。这两种样式下，**imageView** 的宽度设置有些区别：

- **.alert**：**imageView** 不管在哪种手机屏幕下，宽度都是一样的。我这里设置成 **250**。
- **.actionSheet**：不同屏幕宽度不一样，我这里将 **imageView** 宽度设置成 “**alertController.view.bounds.size.width - 40**”

   [![原文:Swift - 自定义UIAlertController的样式2（添加自定义视图）](https://www.hangge.com/blog_uploads/201704/2017042014235011429.png)](https://www.hangge.com/blog/cache/detail_1657.html#)    [![原文:Swift - 自定义UIAlertController的样式2（添加自定义视图）](https://www.hangge.com/blog_uploads/201704/2017042014241027215.png)](https://www.hangge.com/blog/cache/detail_1657.html#)

### 3，样例代码

```swift
import UIKit
 
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    override func viewDidAppear(_ animated: Bool){
        super.viewDidAppear(animated)
         
        let alertController = UIAlertController(title: "\n\n\n\n\n\n\n",
                                                message: nil,
                                                preferredStyle: .actionSheet)
         
        //添加imageView控件
        let image = UIImage(named: "image1.jpg")
        let imageView = UIImageView(image: image)
        //actionSheet样式尺寸
        imageView.frame = CGRect(x: 10, y: 10,
                                 width: alertController.view.bounds.size.width - 40,
                                 height: 135)
        alertController.view.addSubview(imageView)
         
        let cancelAction = UIAlertAction(title: "取消", style: .cancel, handler: nil)
        let okAction = UIAlertAction(title: "确定", style: .default, handler: {
            action in
            print("点击了确定")
        })
        alertController.addAction(cancelAction)
        alertController.addAction(okAction)
        self.present(alertController, animated: true, completion: nil)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1657.html