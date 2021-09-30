# Swift - 设置进度条组件（UIProgressView）的宽度和高度

2016-05-03发布：hangge阅读：4157

**1，设置progressView的宽度（进度条长度）**

通常情况下，我们可以在初始化 **progressView** 的时候通过 **frame** 属性设置其宽度（进度条长度）。

比如下面样例，我在屏幕中放置一个横向宽度是200的进度条，其位置是水平居中。

[![原文:Swift - 设置进度条组件（UIProgressView）的宽度和高度](https://www.hangge.com/blog_uploads/201605/2016050109535536566.png)](https://www.hangge.com/blog/cache/detail_1165.html#)

```swift
import UIKit
 
class ViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //将背景设为黑色
        self.view.backgroundColor = UIColor.blackColor()
         
        //创建一个宽度是200的进度条
        let myProgressView = UIProgressView(frame: CGRectMake(0, 0, 200, 10))
         
        //设置进度条位置（水平居中）
        myProgressView.layer.position = CGPoint(x: self.view.frame.width/2, y: 100)
         
        //进度条条进度
        myProgressView.progress = 0.3
         
        //把进度条添加到view中来
        self.view.addSubview(myProgressView)
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


**2，设置progressView的高度**
但我们会发现无论如何设置 **progressView** 的高度，其最终显示出来的高度都不会变化。所以如果想改变高度，可以换个思路，通过改变 **progressView** 的 **scale**（缩放比例）来实现。
下面样例将进度条高度调整到默认的5倍。

[![原文:Swift - 设置进度条组件（UIProgressView）的宽度和高度](https://www.hangge.com/blog_uploads/201605/2016050109540327073.png)](https://www.hangge.com/blog/cache/detail_1165.html#)

```swift
import UIKit
 
class ViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //将背景设为黑色
        self.view.backgroundColor = UIColor.blackColor()
         
        //创建一个宽度是200的进度条
        let myProgressView = UIProgressView(frame: CGRectMake(0, 0, 200, 10))
         
        //设置进度条位置（水平居中）
        myProgressView.layer.position = CGPoint(x: self.view.frame.width/2, y: 100)
         
        //通过变形改变进度条高度（ 横向宽度不变，纵向高度变成默认的5倍）
        myProgressView.transform = CGAffineTransformMakeScale(1.0, 5.0)
         
        //进度条条进度
        myProgressView.progress = 0.3
         
        //把进度条添加到view中来
        self.view.addSubview(myProgressView)
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1165.html