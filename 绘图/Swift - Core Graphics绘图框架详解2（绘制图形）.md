# Swift - Core Graphics绘图框架详解2（绘制图形）

2016-11-18发布：hangge阅读：3494

### 1，绘制矩形

[![原文:Swift - Core Graphics绘图框架详解2（绘制图形）](https://www.hangge.com/blog_uploads/201611/2016111410491497674.png)](https://www.hangge.com/blog/cache/detail_1438.html#)

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
         
        //获取绘图上下文
        guard let context = UIGraphicsGetCurrentContext() else {
            return
        }
         
        //创建一个矩形，它的所有边都内缩3点
        let drawingRect = self.bounds.insetBy(dx: 3, dy: 3)
         
        //创建并设置路径
        let path = CGMutablePath()
        //绘制矩形
        path.addRect(drawingRect)
         
        //添加路径到图形上下文
        context.addPath(path)
         
        //设置笔触颜色
        context.setStrokeColor(UIColor.orange.cgColor)
        //设置笔触宽度
        context.setLineWidth(6)
        //设置填充颜色
        context.setFillColor(UIColor.blue.cgColor)
         
        //绘制路径并填充
        context.drawPath(using: .fillStroke)
    }
}
```



### 2，绘制圆角矩形

[![原文:Swift - Core Graphics绘图框架详解2（绘制图形）](https://www.hangge.com/blog_uploads/201611/2016111410525316601.png)](https://www.hangge.com/blog/cache/detail_1438.html#)

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
         
        //创建一个矩形，它的所有边都内缩3点
        let drawingRect = self.bounds.insetBy(dx: 3, dy: 3)
         
        //创建并设置路径
        let path = CGMutablePath()
        //绘制矩形
        path.addRoundedRect(in: drawingRect, cornerWidth: 10, cornerHeight: 10)
         
        //添加路径到图形上下文
        context.addPath(path)
         
        //设置笔触颜色
        context.setStrokeColor(UIColor.orange.cgColor)
        //设置笔触宽度
        context.setLineWidth(6)
        //设置填充颜色
        context.setFillColor(UIColor.blue.cgColor)
         
        //绘制路径并填充
        context.drawPath(using: .fillStroke)
    }
}

```



### 3，绘制圆形

绘制正圆和绘制椭圆都是使用 **addEllipse()** 方法。如果宽高都一样就是正圆形。

[![原文:Swift - Core Graphics绘图框架详解2（绘制图形）](https://www.hangge.com/blog_uploads/201611/2016111411060245053.png)](https://www.hangge.com/blog/cache/detail_1438.html#)

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
         
        //创建并设置路径
        let path = CGMutablePath()
        //绘制正圆
        let minWidth = min(self.bounds.width, self.bounds.height)
        path.addEllipse(in: CGRect(x: 3, y: 3, width: minWidth-6, height: minWidth-6))
         
        //添加路径到图形上下文
        context.addPath(path)
         
        //设置笔触颜色
        context.setStrokeColor(UIColor.orange.cgColor)
        //设置笔触宽度
        context.setLineWidth(6)
        //设置填充颜色
        context.setFillColor(UIColor.blue.cgColor)
         
        //绘制路径并填充
        context.drawPath(using: .fillStroke)
    }
}
```



### 4，绘制椭圆

绘制正圆和绘制椭圆都是使用 **addEllipse()** 方法。如果宽高不一样就是椭圆形。

[![原文:Swift - Core Graphics绘图框架详解2（绘制图形）](https://www.hangge.com/blog_uploads/201611/2016111411081344482.png)](https://www.hangge.com/blog/cache/detail_1438.html#)

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
         
        //创建一个矩形，它的所有边都内缩3点
        let drawingRect = self.bounds.insetBy(dx: 3, dy: 3)
         
        //创建并设置路径
        let path = CGMutablePath()
        //绘制椭圆
        path.addEllipse(in: drawingRect)
         
        //添加路径到图形上下文
        context.addPath(path)
         
        //设置笔触颜色
        context.setStrokeColor(UIColor.orange.cgColor)
        //设置笔触宽度
        context.setLineWidth(6)
        //设置填充颜色
        context.setFillColor(UIColor.blue.cgColor)
         
        //绘制路径并填充
        context.drawPath(using: .fillStroke)
    }
}

```



### 5，复杂图形的绘制

应用中有时会需要绘制一些不规则的图形。我们可以通过多段路径链接，或者多条子路径组合的形式来实现。具体参考我原来写的两篇文章：

[Swift - 在UIView上使用自定义曲线绘制复杂图形（贝塞尔曲线）](https://www.hangge.com/blog/cache/detail_939.html)

[![原文:Swift - Core Graphics绘图框架详解2（绘制图形）](https://www.hangge.com/blog_uploads/201511/2015111620142112085.png)](https://www.hangge.com/blog/cache/detail_1438.html#)

[Swift - 在UIView上使用多条子路径绘制图形](https://www.hangge.com/blog/cache/detail_941.html)

[![原文:Swift - Core Graphics绘图框架详解2（绘制图形）](https://www.hangge.com/blog_uploads/201511/2015111718283289199.png)](https://www.hangge.com/blog/cache/detail_1438.html#)



### 6，设置阴影

给图形设置阴影同给线条添加阴影方法是一样的，这里就不再说明了。具体参考前面文章：[Swift - Core Graphics绘图框架详解1（绘制线条）](https://www.hangge.com/blog/cache/detail_1437.html)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1438.html