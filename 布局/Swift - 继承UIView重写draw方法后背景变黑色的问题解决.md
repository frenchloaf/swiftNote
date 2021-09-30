# Swift - 继承UIView重写draw方法后背景变黑色的问题解决

2016-11-15发布：hangge阅读：2742

通常我们会继承 **UIView** 来实现一些自定义的组建。如果需要在上面进行绘图操作的话，可以通过重写 **draw** 方法来实现。比如下面样例，在组件内部绘制一个空心矩形。

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
     
    override func draw(_ rect: CGRect) {
        super.draw(rect)
         
        //获取绘图上下文
        guard let context = UIGraphicsGetCurrentContext() else {
            return
        }
         
        //创建一个矩形，它的所有边都内缩10点
        let drawingRect = self.bounds.insetBy(dx: 10, dy: 10)
         
        //创建并设置路径
        let path = CGMutablePath()
        path.addRect(drawingRect)
        context.addPath(path)
         
        //设置笔触颜色
        context.setStrokeColor(UIColor.orange.cgColor)
        //设置笔触宽度
        context.setLineWidth(6)
 
        //绘制路径并填充
        context.strokePath()
    }
}
```

但我们将其添加到主视图中会发现，背景色（即除了绘制区域外所有地方）变成黑色的。

[![原文:Swift - 继承UIView重写draw方法后背景变黑色的问题解决](https://www.hangge.com/blog_uploads/201611/2016111416255544714.png)](https://www.hangge.com/blog/cache/detail_1436.html#)

有两种方式解决这个问题。

###  1，使用这个自定义视图的时候将背景色设为透明

```swift
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let frame = CGRect(x: 30, y: 30, width: 250, height: 100)
        let cgView = CGView(frame: frame)
        cgView.backgroundColor = UIColor.clear
        self.view.addSubview(cgView)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

可以发现背景变成透明的了：

[![原文:Swift - 继承UIView重写draw方法后背景变黑色的问题解决](https://www.hangge.com/blog_uploads/201611/2016111416273698173.png)](https://www.hangge.com/blog/cache/detail_1436.html#)



### 2，在自定义视图的init()方法中将背景色设为透明

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
         
        //创建一个矩形，它的所有边都内缩10点
        let drawingRect = self.bounds.insetBy(dx: 10, dy: 10)
         
        //创建并设置路径
        let path = CGMutablePath()
        path.addRect(drawingRect)
        context.addPath(path)
         
        //设置笔触颜色
        context.setStrokeColor(UIColor.orange.cgColor)
        //设置笔触宽度
        context.setLineWidth(6)
 
        //绘制路径并填充
        context.strokePath()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1436.html