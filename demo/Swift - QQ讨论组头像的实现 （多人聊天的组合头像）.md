# Swift - QQ讨论组头像的实现 （多人聊天的组合头像）

2016-12-07发布：hangge阅读：2514

我们知道 **QQ** 里面的联系人头像是圆形的。当我们发起多人聊天时，会自动生成一个讨论组。这个讨论组的头像图标是由组内人员头像自动组合生成的。比如：组内有两个人，就用两个人的头像组合成讨论组的头像图标。有三个就是用三个头像来组成，以此类推。最多5个。

本文演示如何实现这种组合头像的功能。

### 1，讨论组头像组件效果图

（1）根据初始化传入的图片数组中图片数量的不同（超过 **5** 张图片的话也只显示前 **5** 个），组件会自动设置内部图片的尺寸和位置。

（2）为了让显示效果更好，多张图片组合时不是简单地将每张图片裁剪成圆形。而会根据位置，向内凹陷一部分。

（3）组件默认背景是透明的，这里为了更好的演示将背景设置成灰色。

[![原文:Swift - QQ讨论组头像的实现 （多人聊天的组合头像）](https://www.hangge.com/blog_uploads/201612/2016120215523897130.png)](https://www.hangge.com/blog/cache/detail_1462.html#)

### 2，组件代码

（1）**GroupIconCell.swift**（组件内部的圆形图标）

- 初始化参数中 **isClip** 表示图标是否要裁剪。
- 如果不裁剪，则生成圆形的图标。
- 如果裁剪，则根据 **degrees** 角度参数，在对应的位置掏一个弧形的缺口。（其实代码中的剪切路径是由两段圆弧组成。外面的大圆弧和内陷的圆弧）

**外面的大圆弧和内陷的圆弧分别使用了两种绘制圆弧的方法：**
**addArc(center: radius: startAngle: endAngle: clockwise:)**：根据圆心坐标、半径、起始角度、结束角度、旋转方向来指定一段圆弧。
**addArc(tangent1End: tangent2End: radius:)**：根据半径和两条切线来绘制指定一段圆弧。

```swift
import UIKit
 
//组合图标内部小图标
class GroupIconCell:CALayer {
    //使用的图片
    var image: UIImage!
    //裁剪缺口位于圆上的位置（0～360度，0为y轴向下位置，顺时针旋转）
    var degrees: Double!
    //是否裁剪
    var isClip: Bool?
     
    //裁剪角度的一半（30即表示裁剪角度为60度，即圆弧上裁剪部分是1/6（60/360））
    let clipHalfAngle:Double = 30
     
    //初始化
    init(image: UIImage, degrees: Double, isClip: Bool) {
        super.init()
        //参数初始化
        self.degrees = degrees
        self.image = image
        self.isClip = isClip
        //这个记得设置，否则图片在Retina设备上显示不准确，会模糊
        self.contentsScale = UIScreen.main.scale
    }
     
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
     
    //绘制内容
    override func draw(in ctx: CGContext) {
        super.draw(in: ctx)
 
        let bounds = self.bounds
        //尺寸
        let size = bounds.size
        //半径
        let radius = size.width/2
        //中心点位置
        let center = CGPoint(x:bounds.midX, y:bounds.midY)
 
        //为方便操作，先变换坐标系
        var transform = CGAffineTransform.identity
        transform = transform.translatedBy(x: center.x, y: center.y)
        transform = transform.rotated(by: CGFloat(radians(degrees: self.degrees)))
        transform = transform.translatedBy(x: -center.x, y: -center.y)
         
        let path = CGMutablePath()
        //判断是非裁剪
        if self.isClip! {
            //绘制大圆弧
            let angle1 = radians(degrees: (90.0-clipHalfAngle))
            let angle2 = radians(degrees: (90.0+clipHalfAngle))
            path.addArc(center: center, radius: radius,
                        startAngle: angle1, endAngle: angle2,
                        clockwise: true, transform: transform)
             
            //绘制小圆弧（形成缺口）
            let angle3 = radians(degrees: (clipHalfAngle))
            let tangent1End = CGPoint(x:radius,
                                      y:radius+(radius*sin(angle1) -
                                        radius*sin(angle3)*tan(angle3)))
            let tangent2End = CGPoint(x:radius+radius*sin(angle3),
                                      y:radius+radius*sin(angle1))
            path.addArc(tangent1End: tangent1End, tangent2End: tangent2End,
                        radius: radius, transform: transform)
        } else {
            //不裁剪的话直接画个圆
            path.addEllipse(in: bounds)
        }
         
        //开始图片处理上下文（由于输出的图不会进行缩放，所以缩放因子等于屏幕的scale即可）
        UIGraphicsBeginImageContextWithOptions(self.frame.size, false, 0)
        let bezierPath = UIBezierPath(cgPath: path)
        bezierPath.close()
        //添加路径裁剪
        bezierPath.addClip()
        //图片绘制
        self.image.draw(in: self.bounds)
        //获得处理后的图片
        let maskedImage = UIGraphicsGetImageFromCurrentImageContext()!
        //结束上下文
        UIGraphicsEndImageContext()
         
        //进入新状态
        UIGraphicsPushContext(ctx)
        //绘制裁剪后的图像
        maskedImage.draw(in: self.bounds)
        //回到之前状态
        UIGraphicsPopContext()
    }
}
 
//将角度转为弧度
func radians(degrees: Double)->CGFloat {
    return CGFloat(degrees/Double(180.0) * M_PI)
}
```


（2）**GroupIcon.swift**（组合头像组件）

- 组件初始化的时候会根据图片数组里图片数量，来调用相应的内部图片元素创建方法。
- 每种创建方法里的行为都差不多。即计算内部图标的尺寸，创建图片并放置到对应的位置。

```swift
 
//组合图标
class GroupIcon:UIView {
    //整个组合图标的长宽尺寸（两个相等）
    var wh:CGFloat!
    //组合图标内部使用的图片
    var images:[UIImage]!
     
    //初始化
    init(wh:CGFloat, images:[UIImage]) {
        //初始化
        super.init(frame:CGRect(x:0, y:0, width:wh, height:wh))
        self.wh = wh
        self.images = images
         
        //背景默认透明
        self.backgroundColor = UIColor.clear
         
        //如果传入的图片数组为空，就不继续创建内部元素了
        if (self.images.count <= 0) {
            return
        }
         
        //根据数量的不同，调用不同的创建方法
        switch images.count{
        case 1:
            self.createCells1()
        case 2:
            self.createCells2()
        case 3:
            self.createCells3()
        case 4:
            self.createCells4()
        case 5:
            //如果有5个或5个以上的图片的话，都只使用前5个图片
            fallthrough
        default:
            self.createCells5()
            break
        }
    }
     
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
     
    //创建内部图标元素（只有一个图标的情况）
    func createCells1(){
        //内部小图标的直径
        let cellD = self.wh!
        //内部小图标的半径
        let cellR = cellD / 2
        //内部每个小图标的尺寸
        let cellSize = CGSize(width:cellD, height:cellD)
         
        //第1个小图标
        let layer0 = GroupIconCell(image:images[0], degrees:0, isClip:false)
        let center0 = CGPoint(x:cellR, y:cellR)
        layer0.frame = getRect(center: center0, size: cellSize)
        self.layer.addSublayer(layer0)
        layer0.setNeedsDisplay()
    }
     
    //创建内部图标元素（有2个图标的情况）
    func createCells2(){
        //内部小图标的直径
        let cellD = (self.wh+self.wh-CGFloat(sqrtf(2))*self.wh)
        //内部小图标的半径
        let cellR = cellD / 2
        //内部每个小图标的尺寸
        let cellSize = CGSize(width:cellD, height:cellD)
         
        //第1个小图标
        let layer0 = GroupIconCell(image:images[0], degrees:0, isClip:false)
        let center0 = CGPoint(x:cellR, y:cellR)
        layer0.frame = getRect(center: center0, size: cellSize)
        self.layer.addSublayer(layer0)
        layer0.setNeedsDisplay()
         
        //第2个小图标
        let layer1 = GroupIconCell(image:images[1], degrees:180 - 45, isClip:true)
        let center1 = CGPoint(x:cellR+CGFloat(sqrtf(2))*cellD/2,
                              y:cellR+CGFloat(sqrtf(2))*cellD/2)
        layer1.frame = getRect(center: center1, size: cellSize)
        self.layer.addSublayer(layer1)
        layer1.setNeedsDisplay()
    }
     
    //创建内部图标元素（有3个图标的情况）
    func createCells3(){
        //内部小图标的直径
        let cellD = self.wh/2
        //内部每个小图标的尺寸
        let cellSize = CGSize(width:cellD, height:cellD)
         
        //第1个小图标
        let layer0 = GroupIconCell(image:images[0], degrees:30, isClip:true)
        let center0 = CGPoint(x:cellD, y:cellD/2)
        layer0.frame = getRect(center: center0, size: cellSize)
        self.layer.addSublayer(layer0)
        layer0.setNeedsDisplay()
         
        //第2个小图标
        let layer1 = GroupIconCell(image:images[1], degrees:270, isClip:true)
        let center1 = CGPoint(x:center0.x-cellD*sin(radians(degrees: 30)),
                              y:cellD/2+cellD*cos(radians(degrees: 30)))
        layer1.frame = getRect(center: center1, size: cellSize)
        self.layer.addSublayer(layer1)
        layer1.setNeedsDisplay()
         
        //第2个小图标
        let layer2 = GroupIconCell(image:images[2], degrees:180 - 30, isClip:true)
        let center2 = CGPoint(x:center1.x+cellD, y:center1.y)
        layer2.frame = getRect(center: center2, size: cellSize)
        self.layer.addSublayer(layer2)
        layer2.setNeedsDisplay()
    }
     
    //创建内部图标元素（有4个图标的情况）
    func createCells4(){
        //内部小图标的直径
        let cellD = self.wh/2
        //内部小图标的半径
        let cellR = cellD / 2
        //内部每个小图标的尺寸
        let cellSize = CGSize(width:cellD, height:cellD)
         
        //第1个小图标
        let layer0 = GroupIconCell(image:images[0], degrees:0, isClip:true)
        let center0 = CGPoint(x:cellR, y:cellR)
        layer0.frame = getRect(center: center0, size: cellSize)
        self.layer.addSublayer(layer0)
        layer0.setNeedsDisplay()
         
        //第2个小图标
        let layer1 = GroupIconCell(image:images[1], degrees:270, isClip:true)
        let center1 = CGPoint(x:center0.x, y:center0.y+cellD)
        layer1.frame = getRect(center: center1, size: cellSize)
        self.layer.addSublayer(layer1)
        layer1.setNeedsDisplay()
         
        //第3个小图标
        let layer2 = GroupIconCell(image:images[2], degrees:180, isClip:true)
        let center2 = CGPoint(x:center1.x+cellD, y:center1.y)
        layer2.frame = getRect(center:center2, size: cellSize)
        self.layer.addSublayer(layer2)
        layer2.setNeedsDisplay()
         
        //第4个小图标
        let layer3 = GroupIconCell(image:images[3], degrees:90, isClip:true)
        let center3 = CGPoint(x:center2.x, y:center2.y-cellD)
        layer3.frame = getRect(center:center3, size: cellSize)
        self.layer.addSublayer(layer3)
        layer3.setNeedsDisplay()
    }
     
    //创建内部图标元素（有5个图标的情况）
    func createCells5(){
        //内部小图标的半径
        let cellR = self.wh/2/(2*sin(radians(degrees: 54))+1)
        //内部小图标的直径
        let cellD = cellR*2
        //内部每个小图标的尺寸
        let cellSize = CGSize(width:cellD, height:cellD)
         
        //第1个小图标
        let layer0 = GroupIconCell(image:images[0], degrees:54, isClip:true)
        let center0 = CGPoint(x:self.wh/2, y:cellR)
        layer0.frame = getRect(center:center0, size: cellSize)
        self.layer.addSublayer(layer0)
        layer0.setNeedsDisplay()
         
        //第2个小图标
        let layer1 = GroupIconCell(image:images[1], degrees:270 + 72, isClip:true)
        let center1 = CGPoint(x:center0.x-cellD*sin(radians(degrees: 54)),
                              y:center0.y+cellD*cos(radians(degrees: 54)))
        layer1.frame = getRect(center:center1, size: cellSize)
        self.layer.addSublayer(layer1)
        layer1.setNeedsDisplay()
         
        //第3个小图标
        let layer2 = GroupIconCell(image:images[2], degrees:270, isClip:true)
        let center2 = CGPoint(x:center1.x+cellD*cos(radians(degrees: 72)),
                              y:center1.y+cellD*sin(radians(degrees: 72)))
        layer2.frame = getRect(center:center2, size: cellSize)
        self.layer.addSublayer(layer2)
        layer2.setNeedsDisplay()
         
        //第4个小图标
        let layer3 = GroupIconCell(image:images[3], degrees:180 + 18, isClip:true)
        let center3 = CGPoint(x:center2.x+cellD, y:center2.y)
        layer3.frame = getRect(center:center3, size: cellSize)
        self.layer.addSublayer(layer3)
        layer3.setNeedsDisplay()
         
        //第5个小图标
        let layer4 = GroupIconCell(image:images[4], degrees:90 + 36, isClip:true)
        let center4 = CGPoint(x:center3.x+cellD*cos(radians(degrees: 72)),
                              y:center3.y-cellD*sin(radians(degrees: 72)))
        layer4.frame = getRect(center:center4, size: cellSize)
        self.layer.addSublayer(layer4)
        layer4.setNeedsDisplay()
    }
     
    //通过中心点坐标和Size尺寸，返回对应的CGRect
    func getRect(center:CGPoint, size:CGSize) -> CGRect {
        return CGRect(x:center.x - size.width / 2, y:center.y - size.height / 2,
                      width:size.width, height:size.height)
    }
}
```

### 3，使用样例

```swift
import UIKit
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //讨论组图标尺寸（长宽一样）
        let viewWH:CGFloat = 140
        //讨论组图标背景色（这里使用灰色，不设置的话则是透明的）
        let viewBgColor = UIColor(red: 0, green:0, blue: 0, alpha: 0.1)
         
        //只由1张图片组成的讨论组图标
        let view1 = GroupIcon(wh: viewWH, images: [UIImage(named:"0")!])
        view1.center = CGPoint(x:85, y:90)
        view1.backgroundColor = viewBgColor
        self.view.addSubview(view1)
         
        //由2张图片组成的讨论组图标
        let view2 = GroupIcon(wh: viewWH, images: [UIImage(named:"0")!,UIImage(named:"1")!])
        view2.center = CGPoint(x:235, y:90)
        view2.backgroundColor = viewBgColor
        self.view.addSubview(view2)
         
        //由3张图片组成的讨论组图标
        let view3 = GroupIcon(wh: viewWH, images: [UIImage(named:"0")!,UIImage(named:"1")!,
                                                   UIImage(named:"2")!])
        view3.center = CGPoint(x:85, y:240)
        view3.backgroundColor = viewBgColor
        self.view.addSubview(view3)
         
        //由4张图片组成的讨论组图标
        let view4 = GroupIcon(wh: viewWH, images: [UIImage(named:"0")!,UIImage(named:"1")!,
                                                   UIImage(named:"2")!,UIImage(named:"3")!])
        view4.center = CGPoint(x:235, y:240)
        view4.backgroundColor = viewBgColor
        self.view.addSubview(view4)
 
        //由5张图片组成的讨论组图标
        let view5 = GroupIcon(wh: viewWH, images: [UIImage(named:"0")!,UIImage(named:"1")!,
                                                   UIImage(named:"2")!,UIImage(named:"3")!,
                                                   UIImage(named:"4")!])
        view5.center = CGPoint(x:85, y:390)
        view5.backgroundColor = viewBgColor
        self.view.addSubview(view5)
 
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

源码下载：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1462.zip](https://www.hangge.com/blog_uploads/201612/2016120215581981667.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1462.html