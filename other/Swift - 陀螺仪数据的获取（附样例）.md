# Swift - 陀螺仪数据的获取（附样例）

2017-02-22发布：hangge阅读：5222

使用 **CoreMotion** 框架，我们可以很方便地获取设备各种传感器的数据（比如：加速器、陀螺仪、磁力计）。关于加速器之前已经写过相关文章了（[Swift - 加速传感器（CoreMotion）的用法](https://www.hangge.com/blog/cache/detail_545.html)），本文讲讲如何获取陀螺仪（螺旋仪）相关数据。

## 一、陀螺仪介绍

通过陀螺仪我们能够感知设备的握持方式，这个在一些赛车类的游戏中常常会用到。

陀螺仪与加速计的区别在于：在静止时，陀螺仪的每个轴的数据都为 **0**。但是加速计则根据手机的摆放位置，**xyz** 的每个轴的数也会不一样。

### 1，x、y、z轴

加速器和陀螺仪数据以 **iOS** 设备上 **3** 维坐标来展示。

（1）对于 **iphone** 手机来说，画面上下为 **y** 轴，左右为 **x** 轴，贯穿屏幕为 **z** 轴。

（2）向上，向右，手机的前面分别是各轴的正方向。

[![原文:Swift - 陀螺仪数据的获取（附样例）](https://www.hangge.com/blog_uploads/201702/2017020819193293224.png)](https://www.hangge.com/blog/cache/detail_1548.html#)

### 2，CMMotionManager类

（1）通过 **CMMotionManager** 类我们可以获取到加速器，陀螺仪，磁力仪，传感器这**4**类数据，同时它们的接口是一致的。

（2）**CMMotionManager** 同时提供了"**拉**"和"**推**"两种方式的获取方式。二者区别如下：

- 通过获取当前任何传感器的状态或者 **CoreMotionManager** 的只读属性组合数据来“**拉**”动态数据。
- 通过 **block** 闭包在特定的时间间隔来获取我们想要的更新数据集合来捕获“**推**”数据。

## 二、使用样例

### 1，需要的时候才去采集数据

（1）效果图

这个便是前面讲的拉数据（**pull**）。我们每次点击“**获取数据**”按钮后，便去取当前陀螺仪的数据并显示出来。

[![原文:Swift - 陀螺仪数据的获取（附样例）](https://www.hangge.com/blog_uploads/201702/2017020911031479951.png)](https://www.hangge.com/blog/cache/detail_1548.html#)

（2）样例代码

```swift
import UIKit
import CoreMotion
 
class ViewController: UIViewController {
    //用于显示数据信息
    @IBOutlet weak var textView: UITextView!
     
    //运动管理器
    let motionManager = CMMotionManager()
     
    override func viewDidLoad() {
        super.viewDidLoad()
 
    }
     
    //点击获取当前陀螺仪数据
    @IBAction func getGyroData(_ sender: Any) {
        //判断设备支持情况
        guard motionManager.isGyroAvailable else {
            self.textView.text = "\n当前设备不支持陀螺仪\n"
            return
        }
         
        //获取数据
        self.motionManager.startGyroUpdates()
        if let gyroData = self.motionManager.gyroData {
            let rotationRate = gyroData.rotationRate
            var text = "---当前陀螺仪数据---\n"
            text += "x: \(rotationRate.x)\n"
            text += "y: \(rotationRate.y)\n"
            text += "z: \(rotationRate.z)\n"
            self.textView.text = text
        }
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

### 2，实时采集所有数据

（1）效果图
这个便是前面讲的推数据（**push**）。根据我们设置的采集频率（通知频率）来不断地获取最新数据。

[![原文:Swift - 陀螺仪数据的获取（附样例）](https://www.hangge.com/blog_uploads/201702/2017020911035084946.png)](https://www.hangge.com/blog/cache/detail_1548.html#)

（2）样例代码

```swift
import CoreMotion
 
class ViewController: UIViewController {
    //用于显示实时信息
    @IBOutlet weak var textView: UITextView!
     
    //运动管理器
    let motionManager = CMMotionManager()
     
    //刷新时间间隔
    let timeInterval: TimeInterval = 0.2
     
    override func viewDidLoad() {
        super.viewDidLoad()
 
        //开始陀螺仪更新
        startGyroUpdates()
    }
     
    // 开始获取陀螺仪数据
    func startGyroUpdates() {
        //判断设备支持情况
        guard motionManager.isGyroAvailable else {
            self.textView.text = "\n当前设备不支持陀螺仪\n"
            return
        }
         
        //设置刷新时间间隔
        self.motionManager.gyroUpdateInterval = self.timeInterval
         
        //开始实时获取数据
        let queue = OperationQueue.current
        self.motionManager.startGyroUpdates(to: queue!, withHandler: { (gyroData, error) in
            guard error == nil else {
                print(error!)
                return
            }
            // 有更新
            if self.motionManager.isGyroActive {
                if let rotationRate = gyroData?.rotationRate {
                    var text = "---当前陀螺仪数据---\n"
                    text += "x: \(rotationRate.x)\n"
                    text += "y: \(rotationRate.y)\n"
                    text += "z: \(rotationRate.z)\n"
                    self.textView.text = text
                }
            }
        })
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1548.html