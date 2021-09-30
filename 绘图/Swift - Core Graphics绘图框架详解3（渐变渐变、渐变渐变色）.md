# Swift - Core Graphics绘图框架详解3（渐变渐变、渐变渐变色）

2016-11-21发布：hangge阅读：4535

**Quartz 2D**的渐变方式分为以下两种：

- 直线**渐变**：渐变以直线方式从开始位置逐渐向终点位置渐变
- **泥渐变**：以中心点为圆心从渐变色向圆形，直到渐变终止

使用中我们可以直接绘制一个渐变，可以将填充到现有的图形路径上。下面通过样例分别进行演示。



## 一、渐变的绘制 

### 1，直线渐变

[![原文:Swift - Core Graphics绘图框架详解3（绘制渐变、填充渐变色）](https://www.hangge.com/blog_uploads/201611/2016111417125432330.png)](https://www.hangge.com/blog/cache/detail_1439.html#)

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
         
        //使用rgb颜色空间
        let colorSpace = CGColorSpaceCreateDeviceRGB()
        //颜色数组（这里使用三组颜色作为渐变）fc6820
        let compoents:[CGFloat] = [0xfc/255, 0x68/255, 0x20/255, 1,
                                   0xfe/255, 0xd3/255, 0x2f/255, 1,
                                   0xb1/255, 0xfc/255, 0x33/255, 1]
        //没组颜色所在位置（范围0~1)
        let locations:[CGFloat] = [0,0.5,1]
        //生成渐变色（count参数表示渐变个数）
        let gradient = CGGradient(colorSpace: colorSpace, colorComponents: compoents,
                                  locations: locations, count: locations.count)!
         
        //渐变开始位置
        let start = CGPoint(x: self.bounds.minX, y: self.bounds.minY)
        //渐变结束位置
        let end = CGPoint(x: self.bounds.maxX, y: self.bounds.minY)
        //绘制渐变
        context.drawLinearGradient(gradient, start: start, end: end,
                                   options: .drawsBeforeStartLocation)
    }
}
```



### 2，拖曳倾斜

[![原文:Swift - Core Graphics绘图框架详解3（绘制渐变、填充渐变色）](https://www.hangge.com/blog_uploads/201611/2016111417211220121.png)](https://www.hangge.com/blog/cache/detail_1439.html#)

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
         
        //使用rgb颜色空间
        let colorSpace = CGColorSpaceCreateDeviceRGB()
        //颜色数组（这里使用三组颜色作为渐变）fc6820
        let compoents:[CGFloat] = [0xfc/255, 0x68/255, 0x20/255, 1,
                                   0xfe/255, 0xd3/255, 0x2f/255, 1,
                                   0xb1/255, 0xfc/255, 0x33/255, 1]
        //没组颜色所在位置（范围0~1)
        let locations:[CGFloat] = [0,0.5,1]
        //生成渐变色（count参数表示渐变个数）
        let gradient = CGGradient(colorSpace: colorSpace, colorComponents: compoents,
                                  locations: locations, count: locations.count)!
         
        //渐变圆心位置（这里外圆内圆都用同一个圆心）
        let center = CGPoint(x: self.bounds.midX, y: self.bounds.midY)
        //外圆半径
        let endRadius = min(self.bounds.width, self.bounds.height) / 2
        //内圆半径
        let startRadius = endRadius / 3
        //绘制渐变
        context.drawRadialGradient(gradient,
                                   startCenter: center, startRadius: startRadius,
                                   endCenter: center, endRadius: endRadius,
                                   options: .drawsBeforeStartLocation)
    }
}

```



## 二、渐变的填充

不是直线渐变、还是摇摇，除了像上面那样直接画到背景上外，还可以填充成任何的形状。

首先我们自定义一个要填充的**路径**，不管是什么形状都可以。接下来将其加入到**上下文**中，制作**Clip**。最后在绘制压缩。

### 1，填充椭圆

通过**context.clip(to: CGRect)**和**context.clip(to: [CGRect])**，我们可以很方便的设置椭圆的**Clip**。下面样例使用后一个方法，在视图上同时添加一些椭圆**Clip**。

[![原文:Swift - Core Graphics绘图框架详解3（绘制渐变、填充渐变色）](https://www.hangge.com/blog_uploads/201611/2016111510164624608.png)](https://www.hangge.com/blog/cache/detail_1439.html#)

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
         
        //创建clip矩形
        let rect1 = CGRect(x: 0, y: 0,
                           width: self.bounds.width/4, height: self.bounds.height/2)
        let rect2 = CGRect(x: self.bounds.maxX/4, y: self.bounds.maxY/2,
                           width: self.bounds.width/4, height: self.bounds.height/2)
        let rect3 = CGRect(x: self.bounds.maxX/2, y: 0,
                           width: self.bounds.width/4, height: self.bounds.height/2)
        let rect4 = CGRect(x: self.bounds.maxX/4*3, y: self.bounds.maxY/2,
                           width: self.bounds.width/4, height: self.bounds.height/2)
        context.clip(to: [rect1, rect2, rect3, rect4])
         
        //使用rgb颜色空间
        let colorSpace = CGColorSpaceCreateDeviceRGB()
        //颜色数组（这里使用三组颜色作为渐变）fc6820
        let compoents:[CGFloat] = [0xfc/255, 0x68/255, 0x20/255, 1,
                                   0xfe/255, 0xd3/255, 0x2f/255, 1,
                                   0xb1/255, 0xfc/255, 0x33/255, 1]
        //没组颜色所在位置（范围0~1)
        let locations:[CGFloat] = [0,0.5,1]
        //生成渐变色（count参数表示渐变个数）
        let gradient = CGGradient(colorSpace: colorSpace, colorComponents: compoents,
                                  locations: locations, count: locations.count)!
         
        //渐变开始位置
        let start = CGPoint(x: self.bounds.minX, y: self.bounds.minY)
        //渐变结束位置
        let end = CGPoint(x: self.bounds.maxX, y: self.bounds.minY)
        //绘制渐变
        context.drawLinearGradient(gradient, start: start, end: end,
                                   options: .drawsBeforeStartLocation)
    }
}
```



### 2，填充不规则的图形

要实现不规则形状的填充，可以先创建对应的**路径**并添加到上下文中，通过**context.clip()**设置**Clip**微生物。下面以实现一个直线形状的方法为例。

[![原文:Swift - Core Graphics绘图框架详解3（绘制渐变、填充渐变色）](https://www.hangge.com/blog_uploads/201611/2016111510101776592.png)](https://www.hangge.com/blog/cache/detail_1439.html#)

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
        path.move(to: CGPoint(x: self.bounds.midX, y: self.bounds.minY))
        path.addLine(to: CGPoint(x: self.bounds.maxX, y: self.bounds.midY))
        path.addLine(to: CGPoint(x: self.bounds.midX, y: self.bounds.maxY))
        path.addLine(to: CGPoint(x: self.bounds.minX, y: self.bounds.midY))
        path.closeSubpath()
        //添加路径到图形上下文
        context.addPath(path)
        context.clip()
         
        //使用rgb颜色空间
        let colorSpace = CGColorSpaceCreateDeviceRGB()
        //颜色数组（这里使用三组颜色作为渐变）fc6820
        let compoents:[CGFloat] = [0xfc/255, 0x68/255, 0x20/255, 1,
                                   0xfe/255, 0xd3/255, 0x2f/255, 1,
                                   0xb1/255, 0xfc/255, 0x33/255, 1]
        //没组颜色所在位置（范围0~1)
        let locations:[CGFloat] = [0,0.5,1]
        //生成渐变色（count参数表示渐变个数）
        let gradient = CGGradient(colorSpace: colorSpace, colorComponents: compoents,
                                  locations: locations, count: locations.count)!
         
        //渐变开始位置
        let start = CGPoint(x: self.bounds.minX, y: self.bounds.minY)
        //渐变结束位置
        let end = CGPoint(x: self.bounds.maxX, y: self.bounds.minY)
        //绘制渐变
        context.drawLinearGradient(gradient, start: start, end: end,
                                   options: .drawsBeforeStartLocation)
    }
}

```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1439.html