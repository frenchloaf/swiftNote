# Swift - 实现自定义的滑块组件（轨道上带着月牙）

2017-07-05发布：hangge阅读：4737

我的一篇关于**UISlider**的文章：[Swift - 这个（UISlider）的最合适](https://www.hangge.com/blog/cache/detail_536.html)，介绍了**UISlider**的使用方法以及修改。但如果我们想在背景轨道上进行一些早期的遍历线，功能 **UISlider自定义**是没有的提供的。

本文演示如何通过继承**UISlider**，实现一个支持显示刻度线的滑块组件。比如我们可以在轨道的**1/4**，**1/2**，**3/4**处做个标记，方便用户定位。又比如使用**滑块**作为播放眉条时，可以在脑海描绘出关键帧的位置。

### 1，效果图

这个自定义的风格组件可以自由设置和模特位置：

- 第1个样例，轨道为深默认轨迹，轨迹为浅尘。同时轨道走为每**10%**的位置，每走一条。
- 第2个样例我们修改了左右位置的颜色和颜色，生日的颜色和宽度。同时修改每个月出现的颜色。
- 第3个样例还修改了轨道上控制器的颜色。
- 第4个样例使用图片来代替轨道上的控制按钮。

[![原文:Swift - 实现自定义的滑块组件（轨道上带着月牙）](https://www.hangge.com/blog_uploads/201707/2017070218400091426.png)](https://www.hangge.com/blog/cache/detail_1597.html#)



### 2，自定义的组件代码（MarkSlider.swift）

其实现原理是通过继承**UISlider**，在**绘制**中生成带周的方法图片（**UIImage**），再通过**setMinimumTrackImage**和**setMaximumTrackImage**设置到**Slider**上。

这里特别要注意左侧的**UIImage**，为了避免路线走时出现的情况，方法我们还可以通过**.resizableImage(withCapInsets: .zero)**将左侧路线设置为不拉长。



```swift
import UIKit
 
//带有刻度的自定义滑块
class MarkSlider: UISlider {
    //刻度位置集合
    var markPositions:[CGFloat] = []
    //刻度颜色
    var markColor: UIColor?
    //刻度宽度
    var markWidth: CGFloat?
    //左侧轨道的颜色
    var leftBarColor: UIColor?
    //右侧轨道的颜色
    var rightBarColor:UIColor?
    //轨道高度
    var barHeight: CGFloat?
     
    //初始化
    override init(frame: CGRect) {
        super.init(frame: frame)
         
        //设置样式的默认值
        self.markColor = UIColor(red: 106/255.0, green: 106/255.0, blue: 124/255.0,
                                 alpha: 0.7)
        self.markPositions = [10,20,30,40,50,60,70,80,90]
        self.markWidth = 1.0
        self.leftBarColor = UIColor(red: 55/255.0, green: 55/255.0, blue: 94/255.0,
                                    alpha: 0.8)
        self.rightBarColor = UIColor(red: 179/255.0, green: 179/255.0, blue: 193/255.0,
                                     alpha: 0.8)
        self.barHeight = 12
    }
     
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
     
     
    override func draw(_ rect: CGRect) {
        super.draw(rect)
 
        //得到左侧带有刻度的轨道图片（注意：图片不拉伸）
        let leftTrackImage = createTrackImage(rect: rect, barColor: self.leftBarColor!)
            .resizableImage(withCapInsets: .zero)
         
        //得到右侧带有刻度的轨道图片
        let rightTrackImage = createTrackImage(rect: rect, barColor: self.rightBarColor!)
         
        //将前面生产的左侧、右侧轨道图片设置到UISlider上
        self.setMinimumTrackImage(leftTrackImage, for: .normal)
        self.setMaximumTrackImage(rightTrackImage, for: .normal)
    }
     
    //生成轨道图片
    func createTrackImage(rect: CGRect, barColor:UIColor) -> UIImage {
        //开始图片处理上下文
        UIGraphicsBeginImageContextWithOptions(rect.size, false, 0)
        let context: CGContext = UIGraphicsGetCurrentContext()!
         
        //绘制轨道背景
        context.setLineCap(.round)
        context.setLineWidth(self.barHeight!)
        context.move(to: CGPoint(x:self.barHeight!/2, y:rect.height/2))
        context.addLine(to: CGPoint(x:rect.width-self.barHeight!/2, y:rect.height/2))
        context.setStrokeColor(barColor.cgColor)
        context.strokePath()
         
        //绘制轨道上的刻度
        for i in 0..<self.markPositions.count {
            context.setLineWidth(self.markWidth!)
            let position: CGFloat = self.markPositions[i]*rect.width/100.0
            context.move(to: CGPoint(x:position, y: rect.height/2-self.barHeight!/2+1))
            context.addLine(to: CGPoint(x:position, y:rect.height/2+self.barHeight!/2-1))
            context.setStrokeColor(self.markColor!.cgColor)
            context.strokePath()
        }
         
        //得到带有刻度的轨道图片
        let trackImage = UIGraphicsGetImageFromCurrentImageContext()!
        //结束上下文
        UIGraphicsEndImageContext()
        return trackImage
    }
}
```



### 3，使用样例

```swift
import UIKit
 
class ViewController: UIViewController {
 
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //滑块1：使用默认样式、默认刻度（每10%一个刻度）
        let firstSlider = MarkSlider(frame:CGRect(x:10,y:50,width:200,height:40))
        view.addSubview(firstSlider)
         
        //滑块2：修改默认样式
        let secondSlider = MarkSlider(frame:CGRect(x:10,y:100,width:200,height:40))
        secondSlider.markColor = UIColor(white: 1, alpha: 0.5) //刻度颜色
        secondSlider.markPositions = [15,30,75] //刻度位置
        secondSlider.markWidth = 3.0 //刻度宽度
        secondSlider.leftBarColor = UIColor.orange //左轨道颜色
        secondSlider.rightBarColor = UIColor(red: 255/255.0, green: 222/255.0,
                                             blue: 0/255.0, alpha: 1.0) //右轨道颜色
        secondSlider.barHeight = 16 //轨道高度
        view.addSubview(secondSlider)
         
        //滑块3：除了修改默认样式，还改变控制器颜色
        let thirdSlider = MarkSlider(frame:CGRect(x:10,y:150,width:200,height:40))
        thirdSlider.markColor = UIColor(white: 0, alpha: 0.1)
        thirdSlider.leftBarColor = UIColor(red: 108/255.0, green: 200/255.0, blue: 0/255.0,
                                           alpha: 1.0)
        thirdSlider.rightBarColor = UIColor(red: 138/255.0, green: 255/255.0, blue: 0/255.0,
                                            alpha: 1.0)
        thirdSlider.thumbTintColor = UIColor.orange  //改变控制器颜色
        view.addSubview(thirdSlider)
         
        //滑块4：除了修改默认样式，还改变控制器图片
        let fourthSlider = MarkSlider(frame:CGRect(x:10,y:200,width:200,height:40))
        fourthSlider.markColor = UIColor(white: 0, alpha: 0.1)
        fourthSlider.leftBarColor = UIColor(red: 108/255.0, green: 200/255.0, blue: 0/255.0,
                                            alpha: 1.0)
        fourthSlider.rightBarColor = UIColor(red: 138/255.0, green: 255/255.0,
                                             blue: 0/255.0, alpha: 1.0)
        fourthSlider.setThumbImage(UIImage(named: "sliderHandle"),
                                   for: .normal) //改变控制器图片
        fourthSlider.barHeight = 4
        view.addSubview(fourthSlider)
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1597.zip](https://www.hangge.com/blog_uploads/201707/201707022052044685.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1597.html