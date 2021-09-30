# Swift - 刮刮卡效果的实现（附样例）

2017-07-13发布：hangge阅读：2341

刮刮卡是指卡上覆盖有涂层，刮开后才能看到下方的数字和字母密码等文字的卡片。常用于电话卡、游戏卡、抽奖卡、刮刮乐卡等上面。

虽然刮刮卡最早只是实体卡，但现在越来越多的网站或者 **App** 上的抽奖也会模拟刮刮卡效果。用户在屏幕上划动，就能刮开涂层看到下方信息。比起点击直接开奖，这种方式会让用户更有参与感。

下面演示如何实现一个刮刮卡效果，为了方便使用我会将其封装成组件。

### 1，实现原理

（1）刮刮卡由两张叠在一起的图片组成。底图显示的是中奖信息，上面覆盖的是涂层图片。

（2）刮开涂层的效果其实就是一个简单的画图板功能。手指滑动时在涂层的相应位置上绘制透明线条，露出下方的底图，看起来就像是刮开涂层一样。

（3）刮奖的过程中还会实时统计出刮开区域占总面积的百分比，其计算方式是“**透明像素个数 / 总像素个数**”。

### 2，效果图

（1）手指在屏幕划动，即可刮开涂层露出下方的中奖图片。

（2）在刮奖的过程中，下方会实时显示出当前已刮开面积占总面积的百分比。为了让用户体验更好，有时没必要让用户把整张彩票都完全刮开，通常刮了超过一定面积（比如 **75%**），就自动将剩余部分全部刮开，

（3）默认使用的笔触是矩形的，这样刮奖痕迹会呈锯齿状，显得更真实些。当然我们也可以使用圆形的笔触。下图显示二者的区别。

   [![原文:Swift - 刮刮卡效果的实现（附样例）](https://www.hangge.com/blog_uploads/201707/2017071221201318378.png)](https://www.hangge.com/blog/cache/detail_1660.html#)    [![原文:Swift - 刮刮卡效果的实现（附样例）](https://www.hangge.com/blog_uploads/201707/2017071221202377818.png)](https://www.hangge.com/blog/cache/detail_1660.html#)

### 3，组件代码

（1）刮刮卡涂层（**ScratchMask.swift**）

```swift
import UIKit
 
//刮刮卡涂层
class ScratchMask: UIImageView {
     
    //代理对象
    weak var delegate: ScratchCardDelegate?
     
    //线条形状
    var lineType:CGLineCap!
    //线条粗细
    var lineWidth: CGFloat!
    //保存上一次停留的位置
    var lastPoint: CGPoint?
     
 
    override init(frame: CGRect) {
        super.init(frame: frame)
         
        //当前视图可交互
        isUserInteractionEnabled = true
    }
     
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
     
    //触摸开始
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        //多点触摸只考虑第一点
        guard  let touch = touches.first else {
            return
        }
        //保存当前点坐标
        lastPoint = touch.location(in: self)
         
        //调用相应的代理方法
        delegate?.scratchBegan?(point: lastPoint!)
    }
     
    //滑动
    override func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent?) {
        //多点触摸只考虑第一点
        guard  let touch = touches.first, let point = lastPoint, let img = image else {
            return
        }
         
        //获取最新触摸点坐标
        let newPoint = touch.location(in: self)
        //清除两点间的涂层
        eraseMask(fromPoint: point, toPoint: newPoint)
        //保存最新触摸点坐标，供下次使用
        lastPoint = newPoint
         
        //计算刮开面积的百分比
        let progress = getAlphaPixelPercent(img: img)
        //调用相应的代理方法
        delegate?.scratchMoved?(progress: progress)
    }
     
    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
        //多点触摸只考虑第一点
        guard  touches.first != nil else {
            return
        }
         
        //调用相应的代理方法
        delegate?.scratchEnded?(point: lastPoint!)
    }
     
    //清除两点间的涂层
    func eraseMask(fromPoint: CGPoint, toPoint: CGPoint) {
        //根据size大小创建一个基于位图的图形上下文
        UIGraphicsBeginImageContextWithOptions(self.frame.size, false, UIScreen.main.scale)
         
        //先将图片绘制到上下文中
        image?.draw(in: self.bounds)
         
        //再绘制线条
        let path = CGMutablePath()
        path.move(to: fromPoint)
        path.addLine(to: toPoint)
         
        let context = UIGraphicsGetCurrentContext()!
        context.setShouldAntialias(true)
        context.setLineCap(lineType)
        context.setLineWidth(lineWidth)
        context.setBlendMode(.clear) //混合模式设为清除
        context.addPath(path)
        context.strokePath()
         
        //将二者混合后的图片显示出来
        image = UIGraphicsGetImageFromCurrentImageContext()
        //结束图形上下文
        UIGraphicsEndImageContext()
    }
     
    //获取透明像素占总像素的百分比
    private func getAlphaPixelPercent(img: UIImage) -> Float {
        //计算像素总个数
        let width = Int(img.size.width)
        let height = Int(img.size.height)
        let bitmapByteCount = width * height
         
        //得到所有像素数据
        let pixelData = UnsafeMutablePointer<UInt8>.allocate(capacity: bitmapByteCount)
        let colorSpace = CGColorSpaceCreateDeviceGray()
        let context = CGContext(data: pixelData,
                                width: width,
                                height: height,
                                bitsPerComponent: 8,
                                bytesPerRow: width,
                                space: colorSpace,
                                bitmapInfo: CGBitmapInfo(rawValue:
                                    CGImageAlphaInfo.alphaOnly.rawValue).rawValue)!
        let rect = CGRect(x: 0, y: 0, width: width, height: height)
        context.clear(rect)
        context.draw(img.cgImage!, in: rect)
         
        //计算透明像素个数
        var alphaPixelCount = 0
        for x in 0...Int(width) {
            for y in 0...Int(height) {
                if pixelData[y * width + x] == 0 {
                    alphaPixelCount += 1
                }
            }
        }
         
        free(pixelData)
         
        return Float(alphaPixelCount) / Float(bitmapByteCount)
    }
}
```


（2）刮刮卡组件、及相关代理协议（**ScratchCard.swift**）

```swift
import UIKit
 
//刮刮卡代理协议
@objc protocol ScratchCardDelegate {
    @objc optional func scratchBegan(point: CGPoint)
    @objc optional func scratchMoved(progress: Float)
    @objc optional func scratchEnded(point: CGPoint)
}
 
//刮刮卡
class ScratchCard: UIView {
    //涂层
    var scratchMask: ScratchMask!
    //底层券面图片
    var couponImageView: UIImageView!
 
    //刮刮卡代理对象
    weak var delegate: ScratchCardDelegate?
    {
        didSet
        {
            scratchMask.delegate = delegate
        }
    }
     
    public init(frame: CGRect, couponImage: UIImage, maskImage: UIImage,
                scratchWidth: CGFloat = 15, scratchType: CGLineCap = .square) {
        super.init(frame: frame)
         
        let childFrame = CGRect(x: 0, y: 0, width: self.frame.width,
                                height: self.frame.height)
         
        //添加底层券面
        couponImageView = UIImageView(frame: childFrame)
        couponImageView.image = couponImage
        self.addSubview(couponImageView)
         
        //添加涂层
        scratchMask = ScratchMask(frame: childFrame)
        scratchMask.image = maskImage
        scratchMask.lineWidth = scratchWidth
        scratchMask.lineType = scratchType
        self.addSubview(scratchMask)
    }
     
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

### 4，使用样例

```swift
import UIKit
 
class ViewController: UIViewController, ScratchCardDelegate {
 
    @IBOutlet weak var label: UILabel!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //创建刮刮卡组件
        let scratchCard = ScratchCard(frame: CGRect(x:20, y:80, width:241, height:106),
                                       couponImage: UIImage(named: "coupon.png")!,
                                       maskImage: UIImage(named: "mask.png")!)
        //设置代理
        scratchCard.delegate = self
        self.view.addSubview(scratchCard)
    }
     
    //滑动开始
    func scratchBegan(point: CGPoint) {
        print("开始刮奖：\(point)")
    }
     
    //滑动过程
    func scratchMoved(progress: Float) {
        print("当前进度：\(progress)")
         
        //显示百分比
        let percent = String(format: "%.1f", progress * 100)
        label.text = "刮开了：\(percent)%"
    }
     
    //滑动结束
    func scratchEnded(point: CGPoint) {
        print("停止刮奖：\(point)")
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


画笔的粗细和样式也是可以设置的：

```swift
let scratchCard = ScratchCard(frame: CGRect(x:20, y:80, width:241, height:106),
                               couponImage: UIImage(named: "coupon.png")!,
                               maskImage: UIImage(named: "mask.png")!,
                               scratchWidth: 20, scratchType: .round)
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1660.zip](https://www.hangge.com/blog_uploads/201707/2017071221253261029.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1660.html