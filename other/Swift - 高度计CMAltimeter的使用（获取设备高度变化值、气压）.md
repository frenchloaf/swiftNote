# Swift - 高度计CMAltimeter的使用（获取设备高度变化值、气压）

2017-02-17发布：hangge阅读：1184

高度计 **CMAltimeter** 是 **CoreMotion** 框架中的一个功能类，主要用于记录设备的高度变化。本文演示它时如何使用的。 

###  1，效果图

（1）程序启动后先判断设备是否支持高度计。

（2）如果支持的话则实时获取高度变化的差值（相对程序启动时设备最开始的位置），以及当前的大气压强值。

[![原文:Swift - 高度计CMAltimeter的使用（获取设备高度变化值、气压）](https://www.hangge.com/blog_uploads/201702/2017020915072091435.png)](https://www.hangge.com/blog/cache/detail_1551.html#)

###  2，样例实现

（1）由于安全限制，首先我们要在 **info.plist** 文件中加入访问用户健康和运动信息的相关描述。

[![原文:Swift - 高度计CMAltimeter的使用（获取设备高度变化值、气压）](https://www.hangge.com/blog_uploads/201702/201702081616076710.png)](https://www.hangge.com/blog/cache/detail_1551.html#)

（2）样例代码

```swift
import UIKit
import CoreMotion
 
class ViewController: UIViewController {
    //用于显示实时信息
    @IBOutlet weak var textView: UITextView!
     
    //高度计对象
    let altimeter = CMAltimeter()
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //开始高度计更新
        startRelativeAltitudeUpdates()
    }
     
    // 开始获取高度计数据
    func startRelativeAltitudeUpdates() {
        //判断设备支持情况
        guard CMAltimeter.isRelativeAltitudeAvailable() else {
            self.textView.text = "\n当前设备不支持获取高度\n"
            return
        }
         
        //初始化并开始实时获取数据
        let queue = OperationQueue.current
        self.altimeter.startRelativeAltitudeUpdates(to: queue!, withHandler: {
            (altitudeData, error) in
            //错误处理
            guard error == nil else {
                print(error!)
                return
            }
             
            //获取各个数据
            var text = "---高度计数据---\n"
            text += "高度变化: \(altitudeData!.relativeAltitude) 米\n"
            text += "压力: \(altitudeData!.pressure) kPa\n"
            self.textView.text = text
 
        })
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1551.html