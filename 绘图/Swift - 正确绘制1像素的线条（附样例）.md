# Swift - 正确绘制1像素的线条（附样例）

2017-01-06发布：hangge阅读：2996

当我们进行 **iOS** 开发时，**UIKit**、**CoreGraphics**、**CoreAnimation** 这些框架坐标系统都是采用点（**Point**）来衡量的。系统在实际渲染时会自动实现点到屏幕像素（**Pixel**）的转换。
系统设备的 **DPI** 不同，点所对应的像素个数也不一样。通常来说我们并不需要管这些，但在特定情况下我们还是需要考虑点与像素的转化。比如本文通过样例，演示如何在不同的设备上都绘制能最细的线条（**1** 像素宽的直线）。



### 1，效果图

这里以自定义一个 **UIView** 为例，我们要实现在这个 **view** 上绘制一个**1**像素的边框，同时内部中央再绘制横纵两根**1**像素的分隔线。整个视图呈现一个田字形样式，效果图如下：

[![原文:Swift - 正确绘制1像素的线条（附样例）](https://www.hangge.com/blog_uploads/201611/2016111614471175024.png)](https://www.hangge.com/blog/cache/detail_1443.html#)



### 2，计算线宽

由于不同的设备下，**1**个点对应的像素个数不一样。

- 非 **Retina** 屏幕：**1**个像素就是**1**个点。
- **Retina** 屏幕中：**iphone5**/**6**/**7**是**4**个像素组成一个点。**plus**是**9**个像素组成一个点。

这样我们就需要动态地计算出当前设备下，一个像素对应多少点。即**1**像素的线宽是对应多少点。
这个我们可以根据当前屏幕的缩放因子来计算：

```swift
//线宽
let lineWidth = 1 / UIScreen.main.scale
```



### 3，计算偏移量

我们使用上面计算出来线宽直接绘制，可能会发现渲染结果并不是我们想象中的那样，线条模糊且宽度不止**1**像素（而边框线条由于裁剪其实只显示一半）。

[![原文:Swift - 正确绘制1像素的线条（附样例）](https://www.hangge.com/blog_uploads/201611/2016111614530764679.png)](https://www.hangge.com/blog/cache/detail_1443.html#)


这是由于系统的反锯齿（**antialiasing**）技术造成的。

**反锯齿技术：**
奇数像素宽度的线在渲染的时候将会表现为柔和的宽度扩展到向上的整数宽度的线，除非你手动的调整线的位置，使线刚好落在一行或列的显示单元内。

也就是说，如果要画一条黑线，条线刚好落在了一列或者一行显示显示单元之内，将会渲染出标准的一个像素的黑线。 但如果线落在了两个行或列的中间时，那么会得到一条“失真”的线，其实是两个像素宽的灰线。

[![原文:Swift - 正确绘制1像素的线条（附样例）](https://www.hangge.com/blog_uploads/201611/2016111614324666244.png)](https://www.hangge.com/blog/cache/detail_1443.html#)

所以对于偶数像素宽度的线我们就不需要偏移了（否则会失真）。只需要对宽度为奇数像素的线条做个位置偏移，使其刚好落在显示单元中。具体偏移量如下：

- **非Retina屏**：0.5Point（1/2）
- **Retina屏非Plus设备**：0.25Point（1/2/2）
- **Retina屏Plus设备**：0.1.666.. Point（1/3/2）

偏移量我们同样可以根据当前屏幕的缩放因子来计算：

```swift
//线偏移量
let lineAdjustOffset = 1 / UIScreen.main.scale / 2
```



### 4，完整代码

```swift
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let frame = CGRect(x: 30, y: 30, width: 180, height: 100)
        let cgView = CGView(frame: frame)
        self.view.addSubview(cgView)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
class CGView:UIView {
     
    //线宽
    let lineWidth = 1 / UIScreen.main.scale
     
    //线偏移量
    let lineAdjustOffset = 1 / UIScreen.main.scale / 2
     
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
         
        //创建一个矩形，它的所有边都内缩固定的偏移量
        let drawingRect = self.bounds.insetBy(dx: lineAdjustOffset, dy: lineAdjustOffset)
         
        //创建并设置路径
        let path = CGMutablePath()
        //外边框
        path.addRect(drawingRect)
        //横向分隔线(中点同样要计算偏移量)
        let midY = CGFloat(Int(self.bounds.midY)) + lineAdjustOffset
        path.move(to: CGPoint(x: drawingRect.minX, y: midY))
        path.addLine(to: CGPoint(x: drawingRect.maxX, y: midY))
        //纵向分隔线(中点同样要计算偏移量)
        let midX = CGFloat(Int(self.bounds.midX)) + lineAdjustOffset
        path.move(to: CGPoint(x: midX, y: drawingRect.minY))
        path.addLine(to: CGPoint(x: midX, y: drawingRect.maxY))
         
        //添加路径到图形上下文
        context.addPath(path)
         
        //设置笔触颜色
        context.setStrokeColor(UIColor.black.cgColor)
        //设置笔触宽度
        context.setLineWidth(lineWidth)
 
        //绘制路径
        context.strokePath()
    }
}
```