# Swift - Core Graphics绘图框架详解1（绘制线条）

2016-11-17发布：hangge阅读：8987

## 一、Core Graphics介绍

### 1，什么是Core Graphics

（1）**Core Graphics Framework** 是一套基于 **C** 的 **API** 框架，使用了 **Quartz** 作为绘图引擎，可用于一切绘图操作。它提供了低级别、轻量级、高保真度的 **2D** 渲染。
（2）**Quartz 2D** 是 **Core Graphics Framework** 的一部分，是一个强大的二维图像绘制引擎。
（3）我们使用的 **UIKit** 库中所有 **UI** 组件其实都是由 **CoreGraphics** 绘制实现的。所以使用 **Core Graphics** 可以实现比 **UIKit** 更底层的功能。
（4）当我们引入 **UIKit** 框架时系统会自动引入 **Core Graphics** 框架，同时在 **UIKit** 内部还对一些常用的绘图 **API** 进行了封装，方便我们使用。 (比如：**CGMutablePath** 是 **Core Graphics** 的底层API，而 **UIBezierPath** 就是对 **CGMutablePath** 的封装。)

### 2，绘图的一般步骤

（1）获取绘图上下文

（2）创建并设置路径

（3）将路径添加到上下文

（4）设置上下文状态（如笔触颜色、宽度、填充色等等）

（5）绘制路径

## 二、线条的绘制

### 1，绘制直线

[![原文:Swift - Core Graphics绘图框架详解1（绘制线条）](https://www.hangge.com/blog_uploads/201611/2016111310512029895.png)](https://www.hangge.com/blog/cache/detail_1437.html#)

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
         
        //创建一个矩形，它的所有边都内缩3
        let drawingRect = self.bounds.insetBy(dx: 3, dy: 3)
         
        //创建并设置路径
        let path = CGMutablePath()
        path.move(to: CGPoint(x:drawingRect.minX, y:drawingRect.minY))
        path.addLine(to:CGPoint(x:drawingRect.maxX, y:drawingRect.minY))
        path.addLine(to:CGPoint(x:drawingRect.maxX, y:drawingRect.maxY))
         
        //添加路径到图形上下文
        context.addPath(path)
         
        //设置笔触颜色
        context.setStrokeColor(UIColor.orange.cgColor)
        //设置笔触宽度
        context.setLineWidth(6)
         
        //绘制路径
        context.strokePath()
    }
}
```



### 2，设置端点的样式 

通过 **setLineCap** 可以设置线条端点（顶点）的样式，使用的是 **CGLineCap** 枚举，内容如下：



- **.butt**：不绘制端点， 线条结尾处直接结束。（默认值）
- **.round**：绘制圆形端点， 线条结尾处绘制一个直径为线条宽度的半圆
- **.square**：线条结尾处绘制半个边长为线条宽度的正方形。这种形状的端点与“**butt**”形状的端点十分相似，只是线条略长一点而已。

下面样例使用圆形端点：

[![原文:Swift - Core Graphics绘图框架详解1（绘制线条）](https://www.hangge.com/blog_uploads/201611/2016111311032757440.png)](https://www.hangge.com/blog/cache/detail_1437.html#)

```swift
//设置顶点样式（圆角）
context.setLineCap(.round)
```



### 3，设置连接点的样式 

通过 **setLineJoin** 可以设置线条拐点的样式，使用的是 **CGLineJoin** 枚举，内容如下：

- **.mitre**：锐角斜切（默认值）
- **.round**：圆头
- **.bevel**：平头斜切

下面样例使用圆头连接点：

[![原文:Swift - Core Graphics绘图框架详解1（绘制线条）](https://www.hangge.com/blog_uploads/201611/2016111311124989308.png)](https://www.hangge.com/blog/cache/detail_1437.html#)

```swift
//设置连接点样式（圆角）
context.setLineJoin(CGLineJoin.round)
```

### 4，设置阴影

我们可以设置阴影的偏移量、模糊度和颜色。 

[![原文:Swift - Core Graphics绘图框架详解1（绘制线条）](https://www.hangge.com/blog_uploads/201611/2016111311404525303.png)](https://www.hangge.com/blog/cache/detail_1437.html#)

```
//设置阴影
context.setShadow(offset: CGSize(width:3, height:3), blur: 0.6,
                  color: UIColor.lightGray.cgColor)
```



### 5，绘制虚线

[![原文:Swift - Core Graphics绘图框架详解1（绘制线条）](https://www.hangge.com/blog_uploads/201611/2016111311173497209.png)](https://www.hangge.com/blog/cache/detail_1437.html#)

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
         
        //创建一个矩形，它的所有边都内缩3
        let drawingRect = self.bounds.insetBy(dx: 3, dy: 3)
         
        //创建并设置路径
        let path = CGMutablePath()
        path.move(to: CGPoint(x:drawingRect.minX, y:drawingRect.minY))
        path.addLine(to:CGPoint(x:drawingRect.maxX, y:drawingRect.minY))
        path.addLine(to:CGPoint(x:drawingRect.maxX, y:drawingRect.maxY))
         
        //添加路径到图形上下文
        context.addPath(path)
         
        //设置笔触颜色
        context.setStrokeColor(UIColor.orange.cgColor)
        //设置笔触宽度
        context.setLineWidth(6)
         
        //虚线每个线段的长度与间隔
        let lengths: [CGFloat] = [15,4]
        //设置虚线样式
        context.setLineDash(phase: 0, lengths: lengths)
         
        //绘制路径
        context.strokePath()
    }
}
```

当然我们也可以设置虚线的顶点和连接点的样式（这里我都使用圆形）：

[![原文:Swift - Core Graphics绘图框架详解1（绘制线条）](https://www.hangge.com/blog_uploads/201611/2016111311281356609.png)](https://www.hangge.com/blog/cache/detail_1437.html#)



### 6，绘制圆弧

圆弧就是圆上的一部分。要绘制圆弧必须确定：圆形中点的坐标、圆的半径、圆弧的起点角度和终点角度。

其中起点角度和终点角度都要用弧度表示，即常量 **pi** 的倍数（**1pi** 是半圆，**2pi** 是整个圆形）。

[![原文:Swift - Core Graphics绘图框架详解1（绘制线条）](https://www.hangge.com/blog_uploads/201601/2016011411044873105.png)](https://www.hangge.com/blog/cache/detail_1437.html#)

下面样例中，我们在自定义的视图中心位置绘制一段 **3/4** 的圆弧，半径根据这个视图的尺寸来定。  

[![原文:Swift - Core Graphics绘图框架详解1（绘制线条）](https://www.hangge.com/blog_uploads/201611/2016111312163219456.png)](https://www.hangge.com/blog/cache/detail_1437.html#)

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
         
        //创建一个矩形，它的所有边都内缩3
        let drawingRect = self.bounds.insetBy(dx: 3, dy: 3)
         
        //创建并设置路径
        let path = CGMutablePath()
         
        //圆弧半径
        let radius = min(drawingRect.width, drawingRect.height)/2
        //圆弧中点
        let center = CGPoint(x:drawingRect.midX, y:drawingRect.midY)
        //绘制圆弧
        path.addArc(center: center, radius: radius, startAngle: 0,
                    endAngle: CGFloat.pi * 1.5, clockwise: false)
         
        //添加路径到图形上下文
        context.addPath(path)
         
        //设置笔触颜色
        context.setStrokeColor(UIColor.orange.cgColor)
        //设置笔触宽度
        context.setLineWidth(6)
         
        //绘制路径
        context.strokePath()
    }
}
```



### 7，贝塞尔曲线绘制

在 **Quartz 2D** 中曲线绘制分为：二次贝塞尔曲线和三次贝塞尔曲线。二次曲线只有一个控制点，而三次曲线有两个控制点。

[![原文:Swift - Core Graphics绘图框架详解1（绘制线条）](https://www.hangge.com/blog_uploads/201611/2016111409414095901.png)](https://www.hangge.com/blog/cache/detail_1437.html#)

（1）二次贝塞尔曲线绘制

[![原文:Swift - Core Graphics绘图框架详解1（绘制线条）](https://www.hangge.com/blog_uploads/201611/2016111409494042793.png)](https://www.hangge.com/blog/cache/detail_1437.html#)

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
        //移动起点
        path.move(to: CGPoint(x: drawingRect.minX, y: drawingRect.maxY))
        //二次贝塞尔曲线终点
        let toPoint = CGPoint(x: drawingRect.maxX, y: drawingRect.maxY)
        //二次贝塞尔曲线控制点
        let controlPoint = CGPoint(x: drawingRect.midX, y: drawingRect.minY)
        //绘制二次贝塞尔曲线
        path.addQuadCurve(to: toPoint, control: controlPoint)
         
        //添加路径到图形上下文
        context.addPath(path)
         
        //设置笔触颜色
        context.setStrokeColor(UIColor.orange.cgColor)
        //设置笔触宽度
        context.setLineWidth(6)
         
        //绘制路径
        context.strokePath()
    }
}
```


（2）三次贝塞尔曲线绘制

[![原文:Swift - Core Graphics绘图框架详解1（绘制线条）](https://www.hangge.com/blog_uploads/201611/2016111410345970585.png)](https://www.hangge.com/blog/cache/detail_1437.html#)

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
         
        //创建一个矩形，它的所有边都内缩3
        let drawingRect = self.bounds.insetBy(dx: 3, dy: 3)
         
        //创建并设置路径
        let path = CGMutablePath()
        //移动起点
        path.move(to: CGPoint(x: drawingRect.minX, y: drawingRect.maxY))
        //三次贝塞尔曲线终点
        let toPoint = CGPoint(x: drawingRect.maxX, y: drawingRect.minY)
        //三次贝塞尔曲线第1个控制点
        let controlPoint1 = CGPoint(x: (drawingRect.minX+drawingRect.midX)/2, y: drawingRect.minY)
        //三次贝塞尔曲线第2个控制点
        let controlPoint2 = CGPoint(x: (drawingRect.midX+drawingRect.maxX)/2, y: drawingRect.maxY)
        //绘制三次贝塞尔曲线
        path.addCurve(to: toPoint, control1: controlPoint1, control2: controlPoint2)
         
        //添加路径到图形上下文
        context.addPath(path)
         
        //设置笔触颜色
        context.setStrokeColor(UIColor.orange.cgColor)
        //设置笔触宽度
        context.setLineWidth(6)
         
        //绘制路径
        context.strokePath()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1437.html