# Swift - 生成随机颜色（扩展UIColor）

2016-10-30发布：hangge阅读：3539

在开发中，我们有时需要生成一些随机的颜色。但 **UIColor** 没有提供方法或属性来直接获取随机颜色，这里对其进行扩展，方便使用。



### 1，扩展UIColor，增加随机颜色属性

```swift
extension UIColor {
    //返回随机颜色
    open class var randomColor:UIColor{
        get
        {
            let red = CGFloat(arc4random()%256)/255.0
            let green = CGFloat(arc4random()%256)/255.0
            let blue = CGFloat(arc4random()%256)/255.0
            return UIColor(red: red, green: green, blue: blue, alpha: 1.0)
        }
    }
}
```



### 2，使用样例

（1）这里我们使用随机颜色来创建一个马赛克墙

[![原文:Swift - 生成随机颜色（扩展UIColor）](https://www.hangge.com/blog_uploads/201610/2016102614341370443.png)](https://www.hangge.com/blog/cache/detail_1413.html#)

（2）完整代码如下

```swift
import UIKit
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //马赛克宽度
        let itemWidth = 5
         
        //行数
        let rowNums = Int(self.view.bounds.height)/itemWidth
         
        //列数
        let colNums = Int(self.view.bounds.width)/itemWidth
         
        for i in 0...rowNums {
            for j in 0...colNums{
                let item = UIView(frame: CGRect(x: j*itemWidth, y: i*itemWidth,
                                                width: itemWidth, height: itemWidth))
                //使用随机颜色
                item.backgroundColor = UIColor.randomColor
                self.view.addSubview(item)
            }
        }
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
extension UIColor {
    //返回随机颜色
    open class var randomColor:UIColor{
        get
        {
            let red = CGFloat(arc4random()%256)/255.0
            let green = CGFloat(arc4random()%256)/255.0
            let blue = CGFloat(arc4random()%256)/255.0
            return UIColor(red: red, green: green, blue: blue, alpha: 1.0)
        }
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1413.html