# Swift - Core Graphics绘图框架详解5（绘制文字、文字复杂排版）

2016-11-23发布：hangge阅读：2819

### 1，简单的文字绘制样例

下面样例中我们设置了文字的字体、颜色以及文字对齐方式。

[![原文:Swift - Core Graphics绘图框架详解5（绘制文字、文字复杂排版）](https://www.hangge.com/blog_uploads/201611/2016111514334937826.png)](https://www.hangge.com/blog/cache/detail_1442.html#)

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
        //为方便看到效果，背景使用黑色
        //self.backgroundColor = UIColor.clear
    }
     
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
     
    override func draw(_ rect: CGRect) {
        super.draw(rect)
         
        //要绘制的文字
        let str = "欢迎访问hangge.com"
        //文字样式属性
        let style = NSMutableParagraphStyle()
        style.alignment = .center
        let attributes = [NSFontAttributeName: UIFont.systemFont(ofSize: 20),
                          NSForegroundColorAttributeName: UIColor.orange,
                          NSParagraphStyleAttributeName: style]
         
        //绘制在指定区域
        (str as NSString).draw(in: self.bounds, withAttributes: attributes)
        //从指定点开始绘制
        //(str as NSString).draw(at: CGPoint(x:0,y:0), withAttributes: attributes)
    }
}
```



#### 2，所有的文字属性

除了前面使用的三个属性外，文字绘制支持的所有属性如下：

- **NSFontAttributeName** 设置字体属性，默认值：12号的系统字体
- **NSForegroundColorAttributeName** 设置字体颜色，取值为 UIColor对象，默认为黑色
- **NSBackgroundColorAttributeName** 设置字体所在区域背景颜色，取值为 UIColor对象，默认值为nil, 透明
- **NSLigatureAttributeName** 设置连体属性，取值为整数，0 表示没有连体字符，1 表示使用默认的连体字符
- **NSKernAttributeName** 设定字符间距，取值为整数，正值间距加宽，负值间距变窄
- **NSStrikethroughStyleAttributeName** 设置删除线，取值为整数
- **NSStrikethroughColorAttributeName** 设置删除线颜色，取值为 UIColor 对象，默认值为黑色
- **NSUnderlineStyleAttributeName** 设置下划线，取值为整数，枚举常量 NSUnderlineStyle中的值，与删除线类似
- **NSUnderlineColorAttributeName** 设置下划线颜色，取值为 UIColor 对象，默认值为黑色
- **NSStrokeWidthAttributeName** 设置笔画宽度，取值为整数，负值填充效果，正值中空效果
- **NSStrokeColorAttributeName** 填充部分颜色，不是字体颜色，取值为 UIColor 对象
- **NSShadowAttributeName** 设置阴影属性，取值为 NSShadow 对象
- **NSTextEffectAttributeName** 设置文本特殊效果，取值为 NSString 对象，目前只有图版印刷效果可用
- **NSBaselineOffsetAttributeName** 设置基线偏移值，取值为 float,正值上偏，负值下偏
- **NSObliquenessAttributeName** 设置字形倾斜度，取值为 float,正值右倾，负值左倾
- **NSExpansionAttributeName** 设置文本横向拉伸属性，取值为 float,正值横向拉伸文本，负值横向压缩文本
- **NSWritingDirectionAttributeName** 设置文字书写方向，从左向右书写或者从右向左书写
- **NSVerticalGlyphFormAttributeName** 设置文字排版方向，取值为整数，0 表示横排文本，1 表示竖排文本
- **NSLinkAttributeName** 设置链接属性，点击后调用浏览器打开指定URL地址
- **NSAttachmentAttributeName** 设置文本附件,取值为NSTextAttachment对象,常用于文字图片混排
- **NSParagraphStyleAttributeName** 设置文本段落排版格式，取值为 NSParagraphStyle 对象



### 3，在指定的路径区域中绘制文字

**CoreText** 是用于处理文字和字体的底层技术。使用它我们可以让一段文字在指定路径中进行绘制，它会自动实现排版。比如下面样例，我们把文字绘制在椭圆型的区域中。

[![原文:Swift - Core Graphics绘图框架详解5（绘制文字、文字复杂排版）](https://www.hangge.com/blog_uploads/201611/201611151559025725.png)](https://www.hangge.com/blog/cache/detail_1442.html#)



由于 **UIView** 的坐标原点是左上角，而 **CoreText** 的坐标原点是右下角。所以为了方便操作，代码中我们先将坐标系上下翻转下，这样让两个坐标系原点重合。

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
        //为方便看到效果，背景使用黑色
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
         
        //将坐标系系上下翻转
        context.textMatrix = CGAffineTransform.identity
        context.translateBy(x: 0, y: self.bounds.height)
        context.scaleBy(x: 1, y: -1)
         
        //创建并设置路径(排版区域)
        let path = CGMutablePath()
        //绘制椭圆
        path.addEllipse(in: self.bounds.insetBy(dx: 1, dy: 1))
        //绘制边框
        context.addPath(path)
        context.strokePath()
         
        //根据framesetter和绘图区域创建CTFrame
        let str = "欢迎访问hangge.com!欢迎访问hangge.com!欢迎访问hangge.com!欢迎访问hangge.com!欢迎访问hangge.com!欢迎访问hangge.com!欢迎访问hangge.com!"
        let attrString = NSAttributedString(string:str)
        let framesetter = CTFramesetterCreateWithAttributedString(attrString)
        let frame = CTFramesetterCreateFrame(framesetter, CFRangeMake(0, attrString.length),
                                             path, nil)
         
        //使用CTFrameDraw进行绘制
        CTFrameDraw(frame, context)
         
        //恢复成初始状态
        context.restoreGState()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1442.html