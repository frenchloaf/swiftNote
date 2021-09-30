# Swift - 磁力计数据的获取（附样例）

2017-02-24发布：hangge阅读：1865

使用 **CoreMotion** 框架，我们可以很方便地获取设备各种传感器的数据（比如：加速器、陀螺仪、磁力计）。上文中我介绍了陀螺仪（[点击查看](https://www.hangge.com/blog/cache/detail_1548.html)），本文接着讲解如何获取磁力计相关数据。

## 一、相关说明

### 1，磁力计介绍

磁力计（又叫磁力传感器、磁力仪、磁场计、磁强计），它可以用来感应周边的磁场情况。

其获取的数据包含三轴磁力值。

iPad的盒盖锁屏功能就是通过磁力计实现。同时它可以感应地球磁场， 获得方向信息， 使位置服务数据更精准，常用于电子罗盘和导航应用。

### 2，CMMotionManager类

（1）通过 **CMMotionManager** 类我们可以获取到加速器，陀螺仪，磁力仪，传感器这**4**类数据，同时它们的接口是一致的。

（2）**CMMotionManager** 同时提供了"拉"和"推"两种方式的获取方式。二者区别如下：

- 通过获取当前任何传感器的状态或是 **CoreMotionManager** 的只读属性组合数据来“拉”动态数据。
- 通过 **block** 闭包在特定的时间间隔来获取你想要的更新数据集合来捕获“推”数据。



## 二、使用样例

### 1，需要的时候才去采集数据

（1）效果图

这个便是前面讲的拉数据（**pull**）。我们每次点击“**获取数据**”按钮后，便去取当前磁力计的数据并显示出来。

[![原文:Swift - 磁力计数据的获取（附样例）](https://www.hangge.com/blog_uploads/201702/2017020914155663951.png)](https://www.hangge.com/blog/cache/detail_1550.html#)

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
     
    //点击获取当前磁力计数据
    @IBAction func getMagnetometerData(_ sender: Any) {
        //判断设备支持情况
        guard motionManager.isMagnetometerAvailable else {
            self.textView.text = "\n当前设备不支持磁力计\n"
            return
        }
         
        //获取数据
        self.motionManager.startMagnetometerUpdates()
        if let magnetometerData = self.motionManager.magnetometerData {
            let magneticField = magnetometerData.magneticField
            var text = "---当前磁力计数据---\n"
            text += "x: \(magneticField.x)\n"
            text += "y: \(magneticField.y)\n"
            text += "z: \(magneticField.z)\n"
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

[![原文:Swift - 磁力计数据的获取（附样例）](https://www.hangge.com/blog_uploads/201702/2017020914165812378.png)](https://www.hangge.com/blog/cache/detail_1550.html#)

（2）样例代码

```swift
import UIKit
import CoreMotion
 
class ViewController: UIViewController {
    //用于显示数据信息
    @IBOutlet weak var textView: UITextView!
     
    //运动管理器
    let motionManager = CMMotionManager()
     
    //刷新时间间隔
    let timeInterval: TimeInterval = 0.2
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //开始磁力计更新
        startMagnetometerUpdates()
    }
     
    // 开始获取磁力计数据
    func startMagnetometerUpdates() {
        //判断设备支持情况
        guard motionManager.isMagnetometerAvailable else {
            self.textView.text = "\n当前设备不支持磁力计\n"
            return
        }
         
        //设置刷新时间间隔
        self.motionManager.magnetometerUpdateInterval = self.timeInterval
         
        //开始实时获取数据
        let queue = OperationQueue.current
        self.motionManager.startMagnetometerUpdates(to: queue!, withHandler: {
            (magnetometerData, error) in
            guard error == nil else {
                print(error!)
                return
            }
            // 有更新
            if self.motionManager.isMagnetometerActive {
                if let magneticField = magnetometerData?.magneticField {
                    var text = "---当前磁力计数据---\n"
                    text += "x: \(magneticField.x)\n"
                    text += "y: \(magneticField.y)\n"
                    text += "z: \(magneticField.z)\n"
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


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1550.html