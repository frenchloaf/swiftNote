# Swift - Core Graphics绘图框架详解4（绘制图片、图形变换）

2016-11-22发布：hangge阅读：2408

## 一、绘制图像

**Core Graphics** 也是支持绘制图片图像的，而且 **UIKit** 中对这个做了封装，方便我们使用。



### 1，绘制到指定的矩形中

使用这种方式的话图片会自动进行拉伸。如果矩形比例不对，图片会变形。

[![原文:Swift - Core Graphics绘图框架详解4（绘制图片、图形变换）](https://www.hangge.com/blog_uploads/201611/2016111511124098372.png)](https://www.hangge.com/blog/cache/detail_1441.html#)

```swift
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let frame = CGRect(x: 30, y: 30, width: 250, height: 100)
        let cgView = CGView(frame: frame)
        self.view.addSubview(cgView)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
class CGView:UIView {
     
    override init(frame: CGRect) {
        super.init(frame: frame)
        //设置背景色为透明，否则是黑色背景
        self.backgroundColor = UIColor.clear
    }
     
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
     
    override func draw(_ rect: CGRect) {
        super.draw(rect)
         
        //获取图像
        let image = UIImage(named: "image1.jpg")
        //绘制到指定的矩形中（大小不合适会自动进行拉伸）
        image?.draw(in: self.bounds)
    }
}
```



### 2，从指定点开始绘制

这种方式图片尺寸大小、比例都不变，如果显示区域不够的话就会显示不全。

[![原文:Swift - Core Graphics绘图框架详解4（绘制图片、图形变换）](https://www.hangge.com/blog_uploads/201611/2016111511170473450.png)](https://www.hangge.com/blog/cache/detail_1441.html#)

```swift
class CGView:UIView {
     
    override init(frame: CGRect) {
        super.init(frame: frame)
        //设置背景色为透明，否则是黑色背景
        self.backgroundColor = UIColor.clear
    }
     
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
     
    override func draw(_ rect: CGRect) {
        super.draw(rect)
         
        //获取图像
        let image = UIImage(named: "image1.jpg")
        //从指定点开始绘制
        image?.draw(at: CGPoint(x: 0, y: 0))
    }
}
```



## 二、变换（Transform）的使用

我们除了可以通过对视图 **View** 进行变换，实现平移旋转缩放操作外。对于绘图上下文，也是可以进行变换的。

[![原文:Swift - Core Graphics绘图框架详解4（绘制图片、图形变换）](https://www.hangge.com/blog_uploads/201611/2016111514141185468.png)](https://www.hangge.com/blog/cache/detail_1441.html#)

（1）上面样例我们对绘图上下文做平移、缩放、旋转变换操作后，对接下来的所有绘图操作都会起作用。这里是使用图像作为演示，如果绘制其它图形同理。

（2）在对绘图上下文进行变换前我们可以先把当前默认状态保存下来，这样绘图完毕后可以还原回去。方便后续操作。



```swift
class CGView:UIView {
     
    override init(frame: CGRect) {
        super.init(frame: frame)
        //设置背景色为透明，否则是黑色背景
        self.backgroundColor = UIColor.clear
    }
     
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
     
    override func draw(_ rect: CGRect) {
        super.draw(rect)
         
        //获取绘图上下文
        guard let context = UIGraphicsGetCurrentContext() else {
            return
        }
         
        //保存初始状态
        context.saveGState()
         
        //变换1：向左向下本别平移10点
        context.translateBy(x: 10, y: 10)
        //变换2：缩放成0.1
        context.scaleBy(x: 0.1, y: 0.1)
        //变换3：旋转10度
        context.rotate(by: CGFloat.pi/18)
         
        //获取图像
        let image = UIImage(named: "image1.jpg")
        //从指定点开始绘制
        image?.draw(at: CGPoint(x: 0, y: 0))
         
        //恢复成初始状态
        context.restoreGState()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1441.html